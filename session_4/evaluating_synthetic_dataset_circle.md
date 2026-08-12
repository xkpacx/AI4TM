# Reading the evaluation report, and putting it to the actual test

This is the text companion to the final section of the synthetic data pipeline notebook. The previous piece trained and tuned CTGAN and TVAE against a real customer table. This piece reads what the search actually measured, and then goes further than a report: it trains the propensity model this whole exercise was built for, once on the synthetic table and once on the real one, and checks both against real, held-out customers.

**Time**: about 20 minutes
**Cost**: $0

---

## Four families, four different questions

An evaluation report from this platform always answers four separate questions, and it's worth keeping them separate rather than reading a report as one combined score:

- **Sanity** asks whether the output resembles real data at all. This is the check that catches a model that produced something unrecognisable, nothing more.
- **Statistical fidelity** asks how closely the synthetic table's distributions match the real one's, column by column and in the relationships between columns.
- **Suitability**, listed as performance, asks the question this week's use case actually depends on: does a model trained on the synthetic data make accurate predictions on real customers? The example metric trains a gradient-boosted model (XGBoost) on the synthetic set and tests it on the real one, which is exactly this week's propensity question.
- **Privacy** asks how exposed an individual synthetic record is, measured here by k-anonymity: a dataset is k-anonymous when every record shares its combination of identifying attributes with at least k-1 others.

These fight each other by construction, the previous lessons this week already made that case, and a report with a great fidelity score and a mediocre suitability score is not a report to average together. It's a sign that whatever made the distributions match didn't preserve the specific relationship this model needs.

---

## What actually happened when the search was skipped

The tuned search in the previous piece runs multiple full training passes and keeps whichever hyperparameters score best on the suitability metric specifically. It's worth seeing what happens without that step, because the difference is the whole argument for doing it.

A single CTGAN fit, 150 iterations, no search, no tuning, trained on this week's real customer data (a 6.3% positive rate after the stratified sampling from the preprocessing piece) produced a synthetic table with a **1.5%** positive rate. Training the exact same propensity model on each, and testing both against the same real, held-out customers:

| Trained on | Accuracy | Precision | Recall | F1 |
|---|---|---|---|---|
| Real data | 0.934 | 0.273 | 0.030 | 0.054 |
| Synthetic data (quick, untuned fit) | 0.936 | 0.000 | 0.000 | 0.000 |

Read the accuracy column first, and then notice how little it says. A model that predicts "no" for every single customer already scores above 93%, because well under 10% of customers in this table bought again. The real model at least catches a handful of them; the untuned synthetic model catches none, a complete collapse of the exact class the whole exercise cares about, hidden entirely behind a headline number that looks fine.

This isn't a failure of the approach. It's what happens when a general-purpose generative model is asked to learn a rare pattern from a small number of examples in too few training passes, exactly the risk the earlier lesson in this session named directly: a value the model barely sees during training is a value it will barely produce. The tuned search exists specifically to catch this, since it runs multiple trials and keeps whichever one actually scores well on suitability rather than accepting the first fit that finishes.

> **What this comparison licenses, and what it doesn't.** A close result here, once the search has actually run, says this model, trained this way, on this target column, transfers from synthetic to real. It says nothing about a different target, a different model family, or an exploratory analysis run later on the same synthetic file. This is the same boundary the first lesson this session named: general-purpose models like CTGAN and TVAE carry no validity guarantee for an analysis that wasn't actually tested.

---

## What this table does, and does not, protect

Even a well-tuned synthetic table isn't anonymous in some absolute sense, and its privacy protection is a property of this specific dataset and configuration, not of the method in general.

What it protects: individual customers. A well-tuned model makes it harder to point at a synthetic row and recover a specific real person, and k-anonymity is one way of measuring how much harder.

What it does not protect: the organisation's own aggregate patterns. Conversion rate, the shape of the payment-type mix, average basket value, seasonality in order volume, all of it survives into the synthetic table by design, because matching those patterns is the entire point of training the model in the first place. A synthetic dataset protects the people described in the data more than it protects the business the data belongs to, exactly the asymmetry this session's earlier lesson named.

---

## Wrap-up

Four metric families, one real number that matters most for a given use case, and one honest demonstration of what happens when the tuning step gets skipped: that's the whole arc of this session's hands-on work, from a real table with a real 0.54% repeat-purchase rate, through a preprocessing bug worth reading the source code to catch, to a synthetic model that has to actually work on real customers, not just look plausible on a report.

**Also in Session 4**: when to use synthetic data at all, and how a synthetic dataset actually gets made.

Questions belong in the Circle community.
