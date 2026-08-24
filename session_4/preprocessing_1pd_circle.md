# Prepare a synthetic dataset

*Preparing a real dataset, and setting the baseline*

A synthetic model only ever learns what the real data actually shows it. Get the real table wrong, and CTGAN or TVAE won't correct that mistake, they'll reproduce it at scale, just harder to notice because the result will still look plausible. This piece is where that gets caught, before either model ever sees the table: why real data has to come first, why a table needs specific checks before it's ready, and how good a model trained on the real data alone actually is, the number everything synthetic gets measured against later.

**Time**: about 20 minutes
**Cost**: $0. Everything in the companion notebook runs locally or on Colab's own CPU/GPU, no API key involved

> **Disclaimer.** This is educational guidance, not a production recipe. `auto_synthetic_data_platform` is not an official Google product and has had a single release since February 2024, so its own dependencies are aging — the companion notebook's setup cell handles that automatically, but expect the underlying packages to need revisiting again in the future.

---

## Why real data comes before anything synthetic

It can sound backwards: this session is about generating synthetic data, and the first step is spending time on a real table. The reason is that a synthetic data model doesn't invent patterns from nothing — it learns whatever pattern the real data actually contains, and reproduces it. If the real table is wrong in some way a person didn't catch, the synthetic table built from it will be wrong in exactly the same way.

This week's exercise is a **propensity model**: a model that predicts how likely a customer is to do something specific, here, buy again within 90 days. That prediction only means something if it's learned from a real repeat-purchase pattern, so this piece works from the [Olist Brazilian e-commerce dataset](https://huggingface.co/datasets/miminmoons/olist-ecommerce-for-delivery-and-review-prediction) — about 100,000 real, anonymised orders from a Brazilian online marketplace between September 2016 and September 2018 — rather than a table invented to look convenient.

Two columns from this week's illustrative schema don't exist in Olist, and naming the substitution matters more than pretending it isn't there: **age band** becomes **top product category** (Olist has no age data), and **channel of first visit** becomes **primary payment type** (Olist has no marketing-channel data). Everything else — region, number of orders, average basket value, days since the last order, and the target itself — comes straight from real order history.

One design choice made that target column trustworthy: a fixed cutoff date, set 90 days before the dataset's last order, with every feature computed only from orders *before* that cutoff and the label checking only for orders *after* it. That's worth explaining, because the tempting shortcut fails in a way that's easy to miss: counting each customer's total orders and asking whether their last one was followed by another almost answers itself, since knowing a customer has, say, 4 orders total already gives away most of what's being predicted. A real prediction only ever has today's information, not tomorrow's, and the fixed cutoff is what keeps this table honest about that.

---

## Why a rare outcome needs special handling

Before any model gets involved, it's worth asking how often the thing being predicted actually happens. When the answer is rare, a model can look accurate while having learned almost nothing, simply by guessing the common answer every time, and for a generative model specifically, an outcome it barely sees during training is one it will barely produce in the synthetic data later.

On this table, the answer is genuinely rare: **0.54%** of customers bought again within 90 days of the cutoff. As a percentage that doesn't sound dramatic, but turned into people it's clearer — out of every 1,000 customers, roughly 5 came back. This is what's called **class imbalance**, one outcome vastly outnumbering the other, and by the thresholds `auto_synthetic_data_platform` itself uses to judge severity (mild between 20% and 40%, moderate between 1% and 20%, extreme below 1%), 0.54% counts as **extreme**. This reflects a real repeat-purchase rate on a real marketplace, not something particular to how this table was built.

An imbalance this extreme is also impractical to train against directly in a teaching setting: with well under 1 in 100 rows positive, every training run would take far longer than this notebook should ask for. The workaround is a **stratified sample**: keep every customer who did come back, and add just enough of the rest to bring the positive rate up to roughly 6%, a more workable imbalance a laptop can train against in reasonable time. A production run would skip this and work from the full table.

---

## Why the table needs to be checked before a model sees it

A model has no way to know, on its own, what kind of thing a column actually represents. Handed a column of numbers, it can't tell whether those numbers are a genuine quantity, where two nearby values are meaningfully close to each other, or a label that just happens to be written in digits. Get that wrong, and a model starts treating something like a postcode as a number it can average, producing a synthetic postcode that's the mathematical midpoint of two real ones, describing no real place at all. So the first thing `Preprocessor`, this week's checking tool, needs from a person is an explicit answer for every column: **numerical**, where distance between values carries real meaning (average basket value, days since the last order), or **categorical**, a label rather than a quantity (region, payment type, the target itself).

The second thing preprocessing tools typically do is protect a model from extreme, rare values that could otherwise pull its attention away from the pattern that actually matters. The usual way to do this automatically is to look at where most of a column's values sit and trim away whatever falls far outside that range, an idea that rests on one assumption: that there's a meaningful "far outside" to trim in the first place. On a column where almost every value is identical, that assumption has nothing to work with.

That's exactly what happened on this table's `num_orders` column: over 90% of customers placed exactly one order before the cutoff, so the automatic trimming range collapsed onto that single number at both ends, and the rule behind it, keep only values strictly between the two ends, had nothing left that qualified. Every row was dropped, and the entire preprocessing step failed with an empty table rather than a merely worse one.

> **Good to know.** This is a habit worth carrying into any tool that runs automatic checks: read what a check actually does before trusting its name. A less extreme column would have produced the same underlying problem without an error to flag it, rows dropped, with nobody choosing to drop them.

The fix is to turn that automatic trimming off, and apply it only where a column genuinely has extreme values worth removing, here `avg_basket_value`, whose real maximum is well over ten times its median. The general lesson outlasts this one dataset: an automatic check is only as good as the assumption sitting underneath it, and it's worth knowing what that assumption is before trusting what the check is named.

---

## Why the check keeps a log, and why that log needs care

Every run of `Preprocessor` keeps a log of what it found and fixed, built so that someone without access to the raw customer table, a colleague, a client, a reviewer, can still see that the data was actually checked rather than assumed clean. On this table, that log flagged a missing-value note on `top_category` (a small number of orders have no matched product, resolved by dropping those rows), a low-variety note on `region` and `primary_payment_type`, and a rarity warning on the target column itself, at the same level the real 0.54% rate would suggest. That last point matters beyond this one dataset: the same check that flags a rare product category is the check that catches a rare outcome too, wherever one shows up.

A log like this is safer to share than the raw table, but it still names real column names and real category values, so it's worth reading before it's sent, the same handoff habit Session 3's data-protection lesson covers in full.

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

Column types declared for a reason, outlier removal applied only where it actually belongs, a real baseline to measure against: none of it involves a single synthetic row yet, and all of it decides what the model downstream actually learns, and what "good enough" should mean once synthetic data enters the picture. The next piece picks up from here: training and auto-tuning CTGAN and TVAE against the table this step produced.

**Also in Session 4**: when to use synthetic data at all, and what a synthetic dataset does and doesn't protect once it exists.

Questions belong in the Circle community.
