# Prepare a synthetic dataset

*Preparing a real dataset, and setting the baseline*

A synthetic model only ever learns what the real data actually shows it. Get the real table wrong — the wrong column types, outliers removed from the wrong place, an imbalance nobody noticed — and CTGAN or TVAE will faithfully reproduce that mistake at scale rather than correct it. This piece is where that gets caught, before either model ever sees the table: what `auto_synthetic_data_platform`'s `Preprocessor` class checks, what happens when its defaults meet a real column, and how good a model trained directly on the real data is, the number everything synthetic gets measured against later.

**Time**: about 20 minutes
**Cost**: $0. Everything in the companion notebook runs locally or on Colab's own CPU/GPU, no API key involved

> **Disclaimer.** This is educational guidance, not a production recipe. `auto_synthetic_data_platform` is not an official Google product and has had a single release since February 2024. The install steps referenced here are verified end to end as of this writing, but expect to revisit them if the underlying packages move again.

---

## Setting up, once, before any of this

The companion notebook opens with a setup cell. Run it before touching anything else this week, in Colab or locally, and watch for the line "Setup complete" once it finishes.

This week's toolkit, `auto_synthetic_data_platform`, is the kind of package that doesn't install cleanly with the single command a person would normally reach for — its one and only release is from February 2024, and it locks in old versions of a few libraries that no longer exist for a current Python. The setup cell already works around this, installing everything from a version-controlled list sitting next to the notebook, the same command whether it's running in Colab or locally. The reasoning behind that install step lives in [Session 1's setup guide](../session_1/setup_guide.ipynb) rather than here, since it's the same general pattern this whole course uses; what matters this week is that it's a one-time, automatic step, not something to debug by hand.

**Running this locally instead of Colab** follows the same `uv`-based approach as every other session — see the "Optional B: Local Setup" section of [Session 1's setup guide](../session_1/setup_guide.ipynb) for the one-time install, then run this from inside the `session_4` folder instead of a plain `jupyter notebook`:

```bash
uv run --with jupyter jupyter lab
```

That builds a fresh, isolated environment for this session automatically, every time, the same disposable-per-session guarantee Colab gives you for free, without a venv to create by hand or an environment to remember to activate. Open the notebook in the browser tab it launches and run the setup cell as normal.

> **About the warnings.** The setup cell prints a few lines that look like errors, saying an old package version is required but a newer one is already installed. That's expected — the whole point of this step is skipping those old, broken requirements in favour of modern, already-installed, compatible ones. The line to actually watch for is the one after: "Setup complete."

---

## Why a real dataset, not an invented one

This week's exercise is a propensity model: predicting whether a customer buys again within 90 days. That prediction only means something if the repeat-purchase pattern behind it is real, so this piece works from the [Olist Brazilian e-commerce dataset](https://huggingface.co/datasets/miminmoons/olist-ecommerce-for-delivery-and-review-prediction) — about 100,000 real, anonymised orders from a Brazilian online marketplace between September 2016 and September 2018 — rather than a table built to look convenient.

Two columns from this week's illustrative schema don't exist in Olist, and naming the substitution matters more than pretending it isn't there: **age band** becomes **top product category** (Olist has no age data), and **channel of first visit** becomes **primary payment type** (Olist has no marketing-channel data). Everything else — region, number of orders, average basket value, days since the last order, and the target itself — comes straight from real order history.

One design choice made that target column trustworthy: a fixed cutoff date, 90 days before the dataset's last order, with every feature computed only from orders *before* that cutoff and the label checking only for orders *after* it. The tempting shortcut — counting each customer's total orders and asking whether their last one was followed by another — leaks the future into the present, since a customer's own final order count already answers the question being predicted.

---

## The real number worth sitting with

Before any model gets involved, it's worth asking: how rare is the thing being predicted, on this real marketplace?

On the full customer table, **0.54%** of customers bought again within 90 days of the cutoff — an **extreme** imbalance by the platform's own thresholds (mild between 20% and 40%, moderate between 1% and 20%, extreme below 1%). That's not an artifact of how this table was built. It's what a real 90-day repeat-purchase rate looks like, and exactly the rare-event scenario Session 2's evaluation lessons were written for.

It's also too rare to tune a generative model against quickly — a model can report a very low loss while barely representing the minority class at all, and every training run would take far longer than a teaching notebook should ask for. The notebook works around this honestly, with a **stratified sample**: every customer who did buy again, plus enough of the rest to bring the positive rate to roughly 6%, a real, moderate imbalance a laptop can train on in reasonable time. A production run would skip this step and tune against the full table.

---

## What preprocessing actually catches

`Preprocessor` exists to catch exactly this kind of mistake before it reaches a model, and running it against a real table, rather than a clean invented one, is what actually shows what it catches.

It needs one explicit decision first: which columns are **numerical**, where the distance between two values carries meaning (average basket value, days since last order), and which are **categorical**, labels rather than quantities (region, payment type, the target itself). Even a column that looks numeric can be categorical — a postcode or product ID stored as digits, for instance, since averaging those produces a number that describes no real place at all.

Left on its default settings, though, `Preprocessor` doesn't just underperform on this table, it crashes with an empty dataframe. The cause: over 90% of customers placed exactly one order before the cutoff, so the automatic outlier filter's low and high cutoffs on that column landed on the exact same number, and its rule, keep only values strictly between the two, left nothing to keep. Every row got dropped.

> **Good to know.** This is the habit this course keeps returning to: read what a check actually does before trusting its name. A percentile-based outlier filter sounds safe on any numerical column. It isn't, on a column dominated by one repeated value, and here the failure was loud, an error on an empty dataframe. A slightly less skewed column would have produced a quieter version of the same problem: rows deleted without anyone choosing to drop them.

The fix: turn automatic outlier removal off, and apply it by hand only where it actually belongs, `avg_basket_value`, which does carry genuine extreme values (its real maximum is well over ten times its median). Numbers, dates, and IDs each fail in their own particular way, and this is what that lesson looks like on a real table.

---

## What the log showed, and who else might see it

Run correctly, `Preprocessor`'s log flagged a missing-value warning on `top_category` (a small number of orders have no matched product, resolved by dropping those rows), a low-cardinality note on `region` and `primary_payment_type`, and moderate class-imbalance warnings on those same columns plus, at the same "moderate" level the real rate would suggest, on the target column `bought_again_90d` itself. That last one matters beyond this dataset: the same check that flags a rare product category is the check that would have caught the target's own rarity automatically, had it run against the full table instead of the sample.

That log is built to be shareable with someone who can't see the raw table, but it still names real columns and real category values. Read a log before sending it, the same handoff habit Session 3's DLP lesson asked for.

---

## Setting the baseline

Every synthetic-trained model in this pipeline gets measured against one number: how good is a propensity model trained directly on the real data, no synthetic step involved? A synthetic table only earns its place here if a model trained on it comes reasonably close to matching this.

Training a simple predictive model on this same real, stratified table, and testing it against real, held-out customers, scored the way Session 2's rare-class lessons ask for — precision, recall, and F1, not accuracy alone:

| Metric | Score |
|---|---|
| Accuracy | 0.934 |
| Precision | 0.273 |
| Recall | 0.030 |
| F1 | 0.054 |

Read accuracy first, then set it aside: predicting "no" for every customer already scores above 93%, since well under 10% actually bought again. Recall is the number that matters, and it's genuinely weak: 0.030 means this model catches roughly 3 in every 100 customers who really did come back. Not a strong model — a real, if faint, signal on a genuinely hard, rare-event problem, and the bar a synthetic-trained model has to clear later this session, not some idealised 90%+ target.

> **Where this number comes from.** This result comes from later in this week's hands-on notebook, trained on real data only, before the synthetic table is ever compared against it. It's introduced here because knowing "how good is real-only" matters before spending any time tuning CTGAN or TVAE, not after.

---

## Wrap-up

Column types declared correctly, outlier removal applied only where it belongs, and a real baseline number to measure against — none of it involves a single synthetic row yet, and all of it decides what the model downstream actually learns, and what "good enough" should mean once synthetic data enters the picture. The next piece picks up from here: training and auto-tuning CTGAN and TVAE against the table this step produced.

**Also in Session 4**: when to use synthetic data at all, and what a synthetic dataset does and doesn't protect once it exists.

Questions belong in the Circle community.
