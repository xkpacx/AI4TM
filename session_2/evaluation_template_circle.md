# Evaluating Model Outputs: Accuracy, Precision, Recall, F1 & the Confusion Matrix

This is the text companion to the evaluation template notebook. Point the template at any set of **true labels** and **predicted labels**, and it tells you not just *whether the model ran*, but *whether it was right* — and, critically, *how* it was wrong.

**Time**: ~20 minutes
**Cost**: A few cents at most — see Session 1's [token cost guide](../session_1/token_cost_guide_circle.md) for how to check before you run anything

> 📝 **For Juliana**: The code and light framing are in place below. Sections marked **📝 Juliana: expand this** are where the deeper logic explanations and business-decision framing should go.

---

## The Model Ran vs. The Model Was Right

An API call can succeed — no error, a clean response, a confident-sounding answer — and still be **wrong**. "It ran" tells you the plumbing worked. It tells you nothing about whether you can trust the output for a business decision.

Evaluation is how you close that gap. Instead of eyeballing a handful of outputs and calling it good, you compare the model's predictions against a set of **known correct answers** (called *ground truth* or *true labels*) and measure exactly how often, and in what way, it gets things wrong.

> 📝 **Juliana: expand this** — a concrete example of "ran but wrong" that would land with agency/consultant folks (e.g. a classifier confidently mislabeling a churn-risk complaint as "general feedback"), and why that distinction matters before a model's output touches a real business decision. This also sets up the Human-in-the-Loop discussion later in the session.

---

## The Evaluation Dataset

To keep the template self-contained and privacy-safe, it runs on a small **synthetic** dataset — 45 invented customer feedback snippets, hand-labeled across five topics an agency or CX team would recognize:

- Shipping & Delivery
- Product Quality
- Pricing & Billing
- Customer Service
- Returns & Refunds

No real customer data, nothing to anonymize. **Swap this for your own labeled data** later — anything with a text column and a true-label column works with the rest of the template.

---

## Accuracy

**Accuracy** answers one question: *of everything the model labeled, what percentage did it get exactly right?*

It's the simplest metric — and the easiest to misread. A model can score 90% accuracy and still be useless if the 10% it gets wrong are the cases that matter most to your business (e.g. always missing the angriest, highest-churn-risk complaints).

> 📝 **Juliana: expand this** — why accuracy alone can be misleading, ideally with a marketing example (e.g. an imbalanced dataset where 90% of feedback is "Shipping & Delivery" — a model that *always* guesses that category looks 90% accurate while being useless).

---

## The Confusion Matrix

Accuracy gives you one number. The **confusion matrix** shows you *where the model gets confused* — which categories it mixes up with which. Rows are the true label, columns are what the model predicted; the diagonal is everything it got right, and everything off the diagonal is a specific kind of mistake.

> 📝 **Juliana: expand this** — how to read a confusion matrix in plain language, and why "which mistakes" often matters more to a business than "how many mistakes" (e.g. confusing Pricing & Billing with Customer Service is a much smaller problem than confusing Returns & Refunds with general feedback, if refunds are what trigger a churn-prevention workflow).

---

## Precision, Recall & F1 — Per Category

Accuracy treats every category the same. These three metrics look at each category individually, and answer three different questions:

- **Precision** — *of everything the model labeled as this category, how much actually was?* Low precision means the model cries wolf — it over-labels this category.
- **Recall** — *of everything that actually was this category, how much did the model catch?* Low recall means the model misses real cases — it under-labels this category.
- **F1 score** — a single number that balances precision and recall, useful when you want one metric per category instead of two.

> 📝 **Juliana: expand this** — the "which mistake would you rather make" framing per metric. E.g. for Returns & Refunds, missing a real return request (low recall) is usually worse than occasionally flagging something as a return that wasn't (low precision), because the cost of a false negative and a false positive aren't symmetric. This is a great lead-in to the Human-in-the-Loop discussion: **high-stakes categories need a human check regardless of the model's overall accuracy.**

---

## The Reusable Template

The notebook wraps all of the above into one function, `evaluate(y_true, y_pred, labels)`, that anyone in the course can copy into their own notebook — swap in your own `true_label` / `predicted_label` columns and run it. It prints the accuracy, the per-category precision/recall/F1 breakdown, and plots the confusion matrix.

### Using it on your own data

1. Get your data into a table with (at minimum) a `true_label` column and a `predicted_label` column — same row order, same category names in both.
2. If you're starting from a CSV: `df = pd.read_csv('your_file.csv')`
3. Run: `evaluate(df['true_label'], df['predicted_label'], labels=[...your categories...])`

That's it — the function doesn't care what the categories are or where the predictions came from (Gemini, another model, or even a human).

---

## ✅ Wrap-Up

You now have a working, reusable way to answer "was the model actually right?" instead of just "did it run?" — accuracy for the overall picture, the confusion matrix for *where* it gets confused, and precision/recall/F1 for *how* it's wrong on each category.

**Next in Session 2**: Human-in-the-loop validation — at what point does a wrong prediction actually reach a business decision, and where does a human need to check it first? The per-category breakdown above is exactly the tool you'd use to decide that.

**Questions?** Post in the Circle community.
