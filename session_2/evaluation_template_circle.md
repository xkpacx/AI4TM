# Evaluating model outputs: accuracy, precision, recall, F1, and the confusion matrix

This is the text companion to the evaluation template notebook. Point the template at a set of **true labels** and **predicted labels**, and it reports not just whether the model ran, but whether it was right, and, critically, how it was wrong.

**Time**: about 20 minutes
**Cost**: a few cents at most — see Session 1's [token cost guide](../session_1/token_cost_guide_circle.md) for how to check before running anything

> **Note for Juliana**: the code and light framing are in place below. Sections marked "Juliana: expand this" are where the deeper logic explanations and business-decision framing should go.

---

## The model ran versus the model was right

An API call can succeed — no error, a clean response, an answer delivered with apparent confidence — and still be wrong. That the model ran tells you the plumbing worked; it tells you nothing about whether the output can be trusted for a business decision.

Evaluation is how that gap gets closed. Rather than eyeballing a handful of outputs and calling it good, the model's predictions are compared against a set of known correct answers, called *ground truth* or *true labels*, and the comparison measures exactly how often, and in what way, the model gets things wrong.

> **Juliana: expand this** — a concrete example of "ran but wrong" that would land with agency and consultant learners (for instance, a classifier confidently mislabeling a churn-risk complaint as general feedback), and why that distinction matters before a model's output touches a real business decision. This also sets up the human-in-the-loop discussion later in the session.

---

## The evaluation dataset

To keep the template self-contained and privacy-safe, it runs on a small synthetic dataset: 45 invented customer feedback snippets, hand-labeled across five topics an agency or CX team would recognize — shipping and delivery, product quality, pricing and billing, customer service, and returns and refunds. None of it is real customer data, so there's nothing to anonymize.

This dataset is a placeholder. Any table with a text column and a true-label column works with the rest of the template, so it can be swapped for real labeled data later.

---

## Accuracy

**Accuracy** answers a single question: of everything the model labeled, what percentage did it get exactly right? It's the simplest metric to compute, and correspondingly the easiest to misread. A model can score 90% accuracy and still be close to useless if the 10% it gets wrong are the cases that matter most to the business, such as consistently missing the angriest, highest-churn-risk complaints.

> **Juliana: expand this** — why accuracy alone can mislead, ideally with a marketing example. An imbalanced dataset where 90% of feedback falls under shipping and delivery makes a model that always guesses that category look 90% accurate, while being useless in practice.

---

## The confusion matrix

Accuracy gives a single number. The **confusion matrix** shows where the model gets confused, meaning which categories it mixes up with which others. Rows represent the true label, columns represent what the model predicted; the diagonal holds everything it got right, and everything off the diagonal is a specific kind of mistake.

> **Juliana: expand this** — how to read a confusion matrix in plain language, and why which mistakes a model makes often matters more to a business than how many mistakes it makes. Confusing pricing and billing with customer service is a much smaller problem than confusing returns and refunds with general feedback, if refunds are what trigger a churn-prevention workflow.

---

## Precision, recall, and F1, by category

Accuracy treats every category the same. These three metrics look at each category individually, and each answers a different question. **Precision** asks: of everything the model labeled as this category, how much actually was? Low precision means the model cries wolf, over-labeling the category. **Recall** asks: of everything that actually was this category, how much did the model catch? Low recall means the model misses real cases, under-labeling the category. **F1 score** balances the two into a single number per category, useful when one figure is preferable to two.

> **Juliana: expand this** — the "which mistake would you rather make" framing, applied per metric. For returns and refunds, missing a real return request (low recall) is usually worse than occasionally flagging something as a return that wasn't (low precision), because the cost of a false negative and a false positive aren't symmetric. This is a natural lead-in to the human-in-the-loop discussion: high-stakes categories need a human check regardless of the model's overall accuracy.

---

## The reusable template

The notebook wraps all of the above into a single function, `evaluate(y_true, y_pred, labels)`, built so that anyone in the course can copy it into their own notebook. Swap in your own `true_label` and `predicted_label` columns and run it; it prints the accuracy, the per-category precision, recall, and F1 breakdown, and plots the confusion matrix.

To use it on other data: get the data into a table with, at minimum, a `true_label` column and a `predicted_label` column, in matching row order and using the same category names in both. From a CSV, that's `df = pd.read_csv('your_file.csv')`. Then run `evaluate(df['true_label'], df['predicted_label'], labels=[...your categories...])`. The function doesn't care what the categories are or where the predictions came from, whether that's Gemini, another model, or a human.

---

## Wrap-up

There is now a working, reusable way to answer whether a model was actually right, rather than only whether it ran: accuracy for the overall picture, the confusion matrix for where it gets confused, and precision, recall, and F1 for how it's wrong on each category.

**Next in Session 2**: human-in-the-loop validation, and specifically at what point a wrong prediction actually reaches a business decision, and where a human needs to check it first. The per-category breakdown above is exactly the tool used to decide that.

Questions belong in the Circle community.
