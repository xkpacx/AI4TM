# Prepare a synthetic dataset

*Preparing a real dataset, and setting the baseline*

A synthetic model only ever learns what the real data actually shows it. Get the real table wrong — the wrong column types, outliers removed from the wrong place, an imbalance nobody noticed — and CTGAN or TVAE will faithfully reproduce that mistake at scale rather than correct it. This piece is where that gets caught, before either model ever sees the table.

**Time**: about 20 minutes
**Cost**: $0. Everything in the companion notebook runs locally or on Colab's own CPU/GPU, no API key involved

> **Disclaimer.** This is educational guidance, not a production recipe. `auto_synthetic_data_platform` is not an official Google product and has had a single release since February 2024, so its own dependencies are aging — the companion notebook's setup cell handles that automatically, but expect the underlying packages to need revisiting again in the future.

---

## Why real data comes before anything synthetic

It can sound backwards: this session is about generating synthetic data, and the first step is spending time on a real table. The reason is that a synthetic data model doesn't invent patterns from nothing — it learns whatever pattern the real data actually contains, and reproduces it. If the real table is wrong in some way a person didn't catch, the synthetic table built from it will be wrong in exactly the same way, just harder to notice, because it will still look plausible.

This week's exercise is a **propensity model**: a model that predicts how likely a customer is to do something specific, here, buy again within 90 days. That prediction only means something if it's learned from a real repeat-purchase pattern, so this piece works from the [Olist Brazilian e-commerce dataset](https://huggingface.co/datasets/miminmoons/olist-ecommerce-for-delivery-and-review-prediction) — about 100,000 real, anonymised orders from a Brazilian online marketplace between September 2016 and September 2018 — rather than a table invented to look convenient.

Two columns from this week's illustrative schema don't exist in Olist, and naming the substitution matters more than pretending it isn't there: **age band** becomes **top product category** (Olist has no age data), and **channel of first visit** becomes **primary payment type** (Olist has no marketing-channel data). Everything else — region, number of orders, average basket value, days since the last order, and the target itself — comes straight from real order history.

One design choice made that target column trustworthy: a fixed cutoff date, set 90 days before the dataset's last order, with every feature computed only from orders *before* that cutoff and the label checking only for orders *after* it. A tempting shortcut, counting each customer's total orders and asking whether their last one was followed by another, almost answers itself: knowing a customer has, say, 4 orders total already gives away most of what you're trying to predict. A real prediction only ever has today's information, not tomorrow's, and the fixed cutoff is what keeps this table honest about that.

---

## The real number worth sitting with

Before any model gets involved, it's worth asking a plain question: how often does the thing being predicted actually happen?

On the full customer table, **0.54%** of customers bought again within 90 days of the cutoff. As a percentage that doesn't sound dramatic, but turned into people it's clearer: out of every 1,000 customers, roughly 5 came back. This is what's called **class imbalance** — one outcome (didn't come back) vastly outnumbering the other (did) — and by the platform's own thresholds for how severe that gets (mild between 20% and 40%, moderate between 1% and 20%, extreme below 1%), 0.54% counts as **extreme**. This reflects a real 90-day repeat-purchase rate on a real marketplace, not an artifact of how this particular table was built.

It also makes the model itself harder to train well: with well under 1 in 100 rows positive, a model can report a very low error rate while barely learning to recognise the rare case at all, simply by leaning on "predict no" almost every time. The notebook works around this honestly, with a **stratified sample**: every customer who did come back, plus enough of the rest to bring the positive rate up to roughly 6%, a real, more workable imbalance a laptop can train against in reasonable time. A production run would skip this step and work from the full table directly.

---

## What preprocessing actually catches

**Preprocessing** means checking a raw table for the assumptions a model needs to hold, and fixing what doesn't, before that table reaches the model at all. None of this is specific to synthetic data — any model trained on messy assumptions learns those assumptions — but it decides everything a synthetic model can learn afterward. This week that job belongs to `Preprocessor`, a class from `auto_synthetic_data_platform`.

The first thing it needs is one explicit decision from a person: which columns are **numerical**, where the distance between two values carries real meaning (average basket value, days since the last order), and which are **categorical**, labels rather than quantities (region, payment type, the target itself). The distinction can be less obvious than it sounds: a postcode looks like a number, but averaging two postcodes together doesn't produce a real place in between them, so it has to be declared a category, not a quantity.

Left on its default settings, `Preprocessor` also tries to remove numerical outliers automatically — it lines up every value in a numerical column from lowest to highest and trims away the extreme half a percent at each end, on the assumption that whatever sits furthest out is noise rather than a real pattern. Run against this table as it comes, that default crashes the entire preprocessing step with an empty table — a hard failure, not a minor drop in quality. The cause: over 90% of customers placed exactly one order before the cutoff, so the "trim the extreme half a percent" rule landed on the exact same number at both ends. Its own logic, keep only values strictly between the two ends, then had nothing left to keep, and every single row got dropped.

> **Good to know.** This is a habit worth carrying into any tool that runs automatic checks: read what a check actually does before trusting its name. "Remove outliers" sounds like a safe default on any numerical column, but a column where almost every customer has the exact same value breaks that assumption entirely, and here the failure was loud, an error on an empty table. A less extreme column would have produced a quieter version of the same problem: rows deleted without anyone choosing to drop them, and no error to notice.

The fix: turn automatic outlier removal off, and apply it by hand only where it actually belongs, `avg_basket_value`, which does carry genuine extreme values (its real maximum is well over ten times its median). Numbers, dates, and category labels each fail their own particular way, and this is what that lesson looks like on a real table rather than a hypothetical one.

---

## What the log showed, and who else might see it

`Preprocessor` keeps a running log of everything it noticed and fixed as it worked, meant to be handed to someone, a colleague, a client, a reviewer, who doesn't have access to the raw customer table itself. Run correctly, this table's log flagged a missing-value warning on `top_category` (a small number of orders have no matched product, resolved by dropping those rows), a note that `region` and `primary_payment_type` have relatively few distinct values, and class-imbalance warnings on those same columns plus, at the same "moderate" level the real rate would suggest, on the target column itself. That last one matters beyond this dataset: the same check that flags a rare product category is the check that would have caught the target's own rarity automatically, had it been run against the full table instead of the stratified sample.

The log is built to be safer to share than the raw table, but it still names real column names and some real category values. Read one before sending it, the same handoff habit Session 3's data-protection lesson asked for at every point data changes hands.

---

## Setting the baseline

Every synthetic-trained model in this pipeline gets measured against one number: how good is a propensity model trained directly on the real data, with no synthetic step involved at all? A synthetic table only earns its place in this pipeline if a model trained on it comes reasonably close to matching this.

Training a simple predictive model on this same real, stratified table, and testing it against real customers it never saw during training, produced these scores:

| Metric | Score |
|---|---|
| Accuracy | 0.934 |
| Precision | 0.273 |
| Recall | 0.030 |
| F1 | 0.054 |

Worth a reminder of what these actually ask, since accuracy alone would be misleading here. **Precision** asks: of the customers the model flagged as likely to return, how many actually did? **Recall** asks: of the customers who really did return, how many did the model actually catch? **F1** combines the two into a single number. Accuracy just asks how often the model was right overall, and on a table this imbalanced that number is close to meaningless: predicting "no" for every single customer would already score above 93%, since well under 10% of customers in this table bought again.

Recall is the number that matters most here, and it's genuinely weak: 0.030 means this model catches roughly 3 in every 100 customers who really did come back. This is a real, if faint, signal on a genuinely hard, rare-event problem, not a strong model by any usual standard, and it's the bar a synthetic-trained model has to clear later this session, not some idealised 90%+ target.

> **Where this number comes from.** This result comes from later in this week's hands-on notebook, trained on real data only, before the synthetic table is ever compared against it. It's introduced here because knowing "how good is real-only" matters before spending any time tuning CTGAN or TVAE, not after.

---

## Wrap-up

Column types declared correctly, outlier removal applied only where it belongs, and a real baseline number to measure against: none of it involves a single synthetic row yet, and all of it decides what the model downstream actually learns, and what "good enough" should mean once synthetic data enters the picture. The next piece picks up from here: training and auto-tuning CTGAN and TVAE against the table this step produced.

**Also in Session 4**: when to use synthetic data at all, and what a synthetic dataset does and doesn't protect once it exists.

Questions belong in the Circle community.
