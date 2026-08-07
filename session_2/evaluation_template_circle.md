# Evaluating model outputs: accuracy, precision, recall, F1, and the confusion matrix

This is the hands-on companion to the accuracy, precision, recall, F1, and confusion matrix lesson earlier this week. If that lesson hasn't been read yet, start there — this piece assumes those definitions are already familiar, and walks through running them for real against a sample dataset instead of re-explaining what they mean.

**Time**: about 20 minutes
**Cost**: a few cents at most — see Session 1's token cost guide for how to check before running anything

---

## What this template does

An API call succeeding tells you the plumbing worked; it says nothing about whether the answer is right. This template closes that gap directly: it compares a model's predictions against a set of known correct answers, *ground truth* or *true labels*, and reports exactly how often, and in what way, it gets things wrong.

---

## The evaluation dataset

To keep the template self-contained and privacy-safe, it runs on a small synthetic dataset: 45 invented customer feedback snippets, hand-labeled across five topics an agency or CX team would recognize — shipping and delivery, product quality, pricing and billing, customer service, and returns and refunds. None of it is real customer data, so there's nothing to anonymize.

This dataset is a placeholder. Any table with a text column and a true-label column works with the rest of the template, so it can be swapped for real labeled data later.

---

## Generate predictions

With the dataset in place, the next step is having Gemini classify each snippet without seeing the true label, so there's something to compare against it. This is the same pattern as the course's first AI-assistant use case, classification, just run here on a placeholder dataset instead of a live one.

Before running a batch job, the habit from Session 1's token cost guide is to estimate the cost first. Forty-five short classification calls on `gemini-2.5-flash-lite` cost a fraction of a cent, but there's no free quota behind it anymore, so it's worth checking that habit even when the dollar figure is tiny.

Each response is checked against the five known category names; anything that doesn't match exactly, or a call that fails outright after a couple of retries, is recorded as its own `UNKNOWN` label rather than dropped, so a formatting slip or a failed call shows up in the metrics below instead of silently vanishing from them.

---

## Accuracy

As covered in the lesson, accuracy alone can be misleading on imbalanced data — a model that always guesses the majority category can outscore one that's actually learned something. This template's own dataset is deliberately balanced at nine items per category, so running it here won't demonstrate that effect; that's a property of this sample data, not a sign anything's wrong with the code below. Real customer feedback is almost never this evenly split.

---

## The confusion matrix

The matrix below follows the same read as the lesson: rows are the true label, columns are the prediction, the diagonal is everything the model got right. One detail specific to this template: a reply that doesn't match one of the five category names, or a call that fails outright after retrying, is recorded as its own `UNKNOWN` category rather than dropped, so it shows up as a distinct row and column here instead of silently vanishing from the count.

One note on axes: scikit-learn's `confusion_matrix`, used in the notebook, puts true labels on the rows and predictions on the columns, matching the description above. Some other tools flip this, so it's worth checking the axis labels before reading anyone else's matrix.

---

## Precision, recall, and F1, by category

Precision, recall, and F1 are computed per category below, exactly as defined in the lesson. One detail specific to this template: predictions can include the `UNKNOWN` category described above, so it's passed alongside the five real topics into every metric call — otherwise accuracy and the confusion matrix would disagree about how many rows exist.

---

## The reusable template

The notebook wraps all of the above into a single function, `evaluate(y_true, y_pred, labels)`, built so that anyone in the course can copy it into their own notebook. Swap in your own `true_label` and `predicted_label` columns and run it; it prints the accuracy, the per-category precision, recall, and F1 breakdown, and plots the confusion matrix.

To use it on other data: get the data into a table with, at minimum, a `true_label` column and a `predicted_label` column, in matching row order and using the same category names in both. From a CSV, that's `df = pd.read_csv('your_file.csv')`. Then run `evaluate(df['true_label'], df['predicted_label'], labels=[...your categories...])`. The function doesn't care what the categories are or where the predictions came from, whether that's Gemini, another model, or a human.

---

## Wrap-up

The lesson explained what these numbers mean; this template is what actually produces them, against a real (if synthetic) dataset, ready to swap for real data whenever it's needed.

**Next in Session 2**: human-in-the-loop validation, and specifically at what point a wrong prediction actually reaches a business decision, and where a human needs to check it first. The per-category breakdown above is exactly the tool used to decide that.

Questions belong in the Circle community.
