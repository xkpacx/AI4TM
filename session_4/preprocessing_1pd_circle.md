# Preprocessing real first-party data before it meets a model

This is the text companion to the first half of the synthetic data pipeline notebook. The previous lessons this week covered when synthetic data is the right tool and how a model like TVAE or CTGAN actually learns a table. Neither of those questions matters if the table handed to the model is full of the wrong assumptions. This piece covers what `auto_synthetic_data_platform`'s `Preprocessor` class actually checks before training starts, and what happens when its defaults meet a column they weren't built for.

**Time**: about 20 minutes
**Cost**: $0. Everything in the companion notebook runs locally or on Colab's own CPU/GPU, no API key involved

> **Disclaimer.** This is educational guidance, not a production recipe. `auto_synthetic_data_platform` is not an official Google product and has had a single release since February 2024. The install steps referenced here are verified end to end as of this writing, but expect to revisit them if the underlying packages move again.

---

## Setting up, once, before any of this

The companion notebook opens with a setup cell, and it's worth running that cell and reading this section before touching anything else this week, in Colab or locally.

`auto_synthetic_data_platform` wraps a library called [synthcity](https://github.com/vanderschaarlab/synthcity) for its actual CTGAN and TVAE models. Neither installs cleanly with the single command a person would normally reach for, `pip install auto_synthetic_data_platform`, because the platform's one and only release, from February 2024, locks in exact versions of `numpy`, `torch`, and a few other libraries that no longer exist for a current Python. Installing `synthcity` on its own also has an independent problem: it resolves a `torch` version that satisfies its own pin, but pulls in a version of `opacus`, an internal dependency, that needs a *newer* `torch` than the one just installed, which crashes on import before any of this week's code runs.

The notebook's setup cell works around both problems in three lines:

```
pip install -q synthcity
pip install -q --no-deps auto_synthetic_data_platform
pip install -q "opacus<1.5" ipython kaleido fastapi uvicorn python-multipart plotly
```

The middle line is the trick that makes this simple: `--no-deps` tells pip to install the platform's own code without trying to enforce its frozen version requirements, so it never touches the broken pins in the first place. This has been verified, end to end, on both Python 3.11 and Python 3.12, so there's no particular Python version to chase.

**In Colab**, that's the whole story: run the setup cell, run the check cell right after it (it just confirms everything imported correctly and tells you in plain language if it didn't), and move on.

**Running this locally instead**, create a virtual environment first, in a terminal, before opening the notebook:

```bash
python3 -m venv venv
source venv/bin/activate
pip install notebook
jupyter notebook
```

Then open the notebook inside that environment and run the setup cell as normal.

> **About the warnings.** The setup cell prints several lines like `auto-synthetic-data-platform 0.0.1 requires numpy==1.23.5, but you have numpy 1.26.4 which is incompatible`. That's expected. Those exact old versions are what's broken; the whole point of `--no-deps` is skipping them in favour of modern, already-installed, mutually compatible ones. The line to actually watch for is the one after: "Setup complete."

---

## The dataset, and two honest substitutions

The notebook this piece supports works on a real dataset: the [Olist Brazilian e-commerce dataset](https://huggingface.co/datasets/miminmoons/olist-ecommerce-for-delivery-and-review-prediction), about 100,000 real, anonymised orders placed at a Brazilian online marketplace between September 2016 and September 2018. It's used instead of an invented table because the whole point of this week's exercise, a propensity model predicting whether a customer buys again within 90 days, needs real repeat-purchase behaviour to be worth anything.

Two columns from this week's illustrative schema don't exist in Olist, and naming the substitution matters more than pretending it isn't there. Olist has no age data at all, so **age band** is replaced with **top product category**, a real column with genuine marketing value. Olist has no marketing-channel data either, so **channel of first visit** is replaced with **primary payment type**. Everything else, region, number of orders, average basket value, days since the last order, and the target itself, comes straight from real order history.

Building that target column correctly took one deliberate design choice: a fixed cutoff date, set 90 days before the last order in the dataset, with every feature computed only from orders *before* the cutoff and the label checking only for orders *after* it. The naive alternative, counting each customer's total orders and asking whether their last one was followed by another, leaks the future into the present, since a customer's own final order count already answers the question being predicted.

---

## The real number worth sitting with

Before any model gets involved, run the same question this week's earlier lesson asked in the abstract: how rare is the thing being predicted?

On the full customer table, **0.54%** of customers bought again within 90 days of the cutoff. By the thresholds `auto_synthetic_data_platform`'s own imbalance check uses, mild between 20% and 40%, moderate between 1% and 20%, extreme below 1%, that's an **extreme** imbalance. This isn't an artifact of how the table was built. It's what a real 90-day repeat-purchase rate looks like on a real marketplace, and it's exactly the scenario Session 2's rare-class evaluation lessons and this week's own imbalance thresholds were written for.

It's also too rare to tune a generative model against quickly: with well under 1% of rows positive, a model can report a very low loss while barely representing the minority class, and every training run would take far longer than a teaching notebook should ask for. The notebook works around this the honest way, with a **stratified sample**: every customer who did buy again, plus a random sample of the rest, at a ratio that brings the positive rate to roughly 6%, a real, moderate imbalance a laptop can train on in a reasonable time. A production run would skip this step and tune against the full table.

---

## Declaring column types

`Preprocessor` needs an explicit mapping between column names and two categories, `numerical` and `categorical`, and it will not infer this on its own. Region, primary payment type, top category, and the target are categorical labels. Number of orders, average basket value, and days since the last order are numerical, quantities where the distance between two values carries meaning. The distinction sounds obvious until a column looks numeric and isn't: a postcode or a product ID stored as digits is categorical, and declaring it numerical produces synthetic postcodes that are averages of real ones, a number that describes no real place at all.

---

## What happened when the platform's defaults met this table

`Preprocessor`'s default settings remove numerical outliers automatically, trimming every numerical column to its 0.5th-to-99.5th percentile range. Run against this table's columns as-is, that default doesn't just underperform, it crashes the entire preprocessing step with an empty dataframe.

The cause sits in one column: `num_orders`. Over 90% of customers in this table placed exactly one order before the cutoff, which means both the 0.5th and the 99.5th percentile of that column land on the same value, 1. The outlier filter keeps only values strictly greater than the lower cut *and* strictly less than the upper one, and when both cuts are the same number, no value, including that number itself, satisfies both conditions at once. Every row gets dropped, and every step downstream fails on a table with nothing left in it.

> **Good to know.** This is exactly the habit this course keeps returning to: read what a check actually does before trusting its name. A percentile-based outlier filter sounds safe on any numerical column. It isn't, on a column dominated by a single repeated value, and the failure here was loud, an error on an empty dataframe. A slightly less skewed column would have produced a quieter version of the same problem: outlier removal silently deleting rows a person never chose to drop.

The fix is to turn the platform's automatic outlier removal off, and apply it by hand only to the one column where it actually makes sense, `avg_basket_value`, which does carry genuine extreme values (the sample's real maximum is well over ten times its median). Numbers, dates, and IDs each fail in their own particular way, and this is what that lesson looks like on a real table rather than a hypothetical one.

---

## What the real log actually showed

Run correctly, `Preprocessor` produced a log with:

- a missing-value warning on `top_category` (a small number of orders have no matched product), resolved by dropping those rows
- a cardinality note on `region` (27 distinct values) and `primary_payment_type` (4 distinct values), both below the platform's recommended 50-to-100 range
- moderate class-imbalance warnings on `region`, `primary_payment_type`, `top_category`, and, at the same "moderate" level the table's real rate would suggest, on the target column `bought_again_90d` itself

That last line matters beyond this one dataset. The same check that flags a rare product category or an underrepresented region is the check that would have caught the target's own rarity automatically, if it had been run against the full table instead of the stratified sample.

> **A log has its own privacy question.** These warnings name the actual column names and category values that triggered them. A log built to be shared with someone who can't see the raw table is safer than the table itself, but it still carries real column names and some real category labels. Read a log before sending it, the same habit Session 3's DLP lesson asked for at every handoff in a pipeline, and this is one more handoff.

---

## Wrap-up

Declaring column types correctly, handling missing values, and applying outlier removal where it belongs rather than everywhere by default, all of it happens before a single synthetic row gets generated, and all of it changes what the model downstream actually learns. The next piece in this session picks up from here: training and auto-tuning CTGAN and TVAE against the table this preprocessing step produced.

**Also in Session 4**: when to use synthetic data at all, and what a synthetic dataset does and doesn't protect once it exists.

Questions belong in the Circle community.
