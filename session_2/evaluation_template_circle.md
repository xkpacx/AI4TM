# Evaluating model outputs: accuracy, precision, recall, F1, and the confusion matrix

This is the text companion to the evaluation template notebook. Point the template at a set of **true labels** and **predicted labels**, and it reports not just whether the model ran, but whether it was right, and, critically, how it was wrong.

**Time**: about 20 minutes
**Cost**: a few cents at most — see Session 1's token cost guide for how to check before running anything

---

## The model ran versus the model was right

An API call can succeed — no error, a clean response, an answer delivered with apparent confidence — and still be wrong. That the model ran tells you the plumbing worked; it tells you nothing about whether the output can be trusted for a business decision. Language models write the same way whether or not they are correct, so the smoothness of an answer carries no information about the accuracy of its content.

Evaluation is how that gap gets closed. Rather than eyeballing a handful of outputs and calling it good, the model's predictions are compared against a set of known correct answers, called *ground truth* or *true labels*, and the comparison measures exactly how often, and in what way, the model gets things wrong.

This is not a hypothetical risk. Zillow priced homes it was buying partly on an algorithmic valuation; when the model's errors went unmeasured against a market moving faster than expected, the company wrote down $407.9 million in inventory for 2021 and closed the business, cutting about a quarter of its workforce. Epic's sepsis-prediction model ran in hundreds of hospitals before an [independent validation study](https://jamanetwork.com/journals/jamainternalmedicine/fullarticle/2781307) at Michigan Medicine found it caught only a third of real cases. Neither failure was one wrong number; each was a wrong number reaching a decision with nothing in between to catch it.

The next section in this course covers human-in-the-loop validation, and specifically where a human needs to check a model's output before it reaches a decision like the ones above. Everything below is the tool used to decide where that check belongs.

---

## The evaluation dataset

To keep the template self-contained and privacy-safe, it runs on a small synthetic dataset: 45 invented customer feedback snippets, hand-labeled across five topics an agency or CX team would recognize — shipping and delivery, product quality, pricing and billing, customer service, and returns and refunds. None of it is real customer data, so there's nothing to anonymize.

This dataset is a placeholder. Any table with a text column and a true-label column works with the rest of the template, so it can be swapped for real labeled data later.

---

## Generate predictions

With the dataset in place, the next step is having Gemini classify each snippet without seeing the true label, so there's something to compare against it. This is the same pattern as the course's first AI-assistant use case, classification, just run here on a placeholder dataset instead of a live one.

Before running a batch job, the habit from Session 1's token cost guide is to estimate the cost first. Forty-five short classification calls costs $0 on the free tier, but still counts against the daily request quota, so it's worth checking that habit even when the dollar figure is zero.

Each response is checked against the five known category names; anything that doesn't match exactly, or a call that fails outright after a couple of retries, is recorded as its own `UNKNOWN` label rather than dropped, so a formatting slip or a failed call shows up in the metrics below instead of silently vanishing from them.

---

## Accuracy

**Accuracy** answers a single question: of everything the model labeled, what percentage did it get exactly right? It's the simplest metric to compute, and correspondingly the easiest to misread. A model can score 90% accuracy and still be close to useless if the 10% it gets wrong are the cases that matter most to the business, such as consistently missing the angriest, highest-churn-risk complaints.

Picture a lead-scoring model instead: 10,000 people fill in a contact form each month, and about 500 of them, 5%, are serious buyers. A model that flags nobody at all, answering "not a buyer" every single time, is right 95% of the time without having learned anything. A real model that flags 1,200 submissions and gets 300 of them right, correctly leaving the other 8,600 alone, scores (300 + 8,600) / 10,000 = 89%, lower than the model that does nothing.

Before reporting an accuracy figure, it's worth working out what the model would score by always guessing the most common category. If it isn't clearly ahead of that baseline, nothing has been shown yet. The same trap applies to this template's own dataset, which is deliberately balanced at nine items per category so it can't demonstrate the effect; real customer feedback is almost never evenly split, and that is where this matters.

---

## The confusion matrix

Accuracy gives a single number. The **confusion matrix** shows where the model gets confused, meaning which categories it mixes up with which others. Rows represent the true label, columns represent what the model predicted; the diagonal holds everything it got right, and everything off the diagonal is a specific kind of mistake. A reply that doesn't match one of the five category names, or a call that fails outright, is recorded as its own `UNKNOWN` category rather than being dropped, so it still shows up as a distinct row and column here instead of silently disappearing from the count.

Reading across a row shows where a category's items actually went, which reveals what that category is losing. Reading down a column shows what got pulled into a category, which reveals what it's over-collecting.

Which mistakes a model makes usually costs more or less than how many it makes. An item drifting between pricing and billing and customer service barely matters if the same team triages both queues. An item drifting from returns and refunds into customer service is expensive if a refund mention is what triggers a churn-prevention workflow: each one is a customer who dropped out of a process built to keep them, and nothing in the accuracy figure reveals that it happened. Accuracy says the model was wrong 10 times out of 100; the matrix says which 10, and that is the piece of information that can actually be acted on.

One note on axes: scikit-learn's `confusion_matrix`, used in the notebook, puts true labels on the rows and predictions on the columns, matching the description above. Some other tools flip this, so it's worth checking the axis labels before reading anyone else's matrix.

---

## Precision, recall, and F1, by category

Accuracy treats every category the same. These three metrics look at each category individually, and each answers a different question. **Precision** asks: of everything the model labeled as this category, how much actually was? Precision = true positives ÷ (true positives + false positives). A false positive here is a row the model labeled as this category that actually belongs to a different one. Low precision means the model cries wolf, over-labeling the category. **Recall** asks: of everything that actually was this category, how much did the model catch? Recall = true positives ÷ (true positives + false negatives). A false negative is a row that actually belongs to this category but got labeled as something else. Low recall means the model misses real cases, under-labeling the category. **F1 score** is the harmonic mean of precision and recall, not a plain average, so it drops toward whichever of the two is lower rather than splitting the difference between them — useful when one figure is preferable to two. F1 weights precision and recall equally, which is itself an assumption: it says the two error types cost the same. When they don't, weighted variants exist, F2 leaning toward recall and F0.5 toward precision.

A model that flags every single item scores 100% recall and near-zero precision. A plain average of the two would still look presentable; the harmonic mean collapses toward zero instead, the correct read, since a model that flags everything has told you nothing.

Picking a category to measure means picking a threshold too: the point above which the model's output counts as a positive prediction for that category. Lowering it means the model flags more, so recall rises as fewer real cases slip past, while precision falls as more of what got flagged shouldn't have been; raising it does the reverse. No setting maximises both, so choosing one means choosing which mistake to make, and that is a business decision rather than a statistical one.

For returns and refunds, missing a genuine return request (low recall, a false negative) usually costs more than occasionally flagging something that wasn't one (low precision, a false positive): the miss is an unresolved customer, and the false alarm is a few seconds of an agent's attention. For a category that feeds an automated action rather than a human queue, that calculus can flip: a false positive that reaches a customer unprompted is a different kind of expensive.

Once precision gets low enough on a high-volume category, people stop working the flagged list at all, and a list nobody works has an effective recall of zero regardless of what the model is doing, a failure mode worth naming: alert fatigue. A low-precision model can destroy the human check that was supposed to protect against it, which is exactly why high-stakes categories need a human review step sized to the category's actual error costs, not to the model's overall accuracy.

One vocabulary note for reading outside marketing: precision is what medicine and statistics call positive predictive value, or PPV; recall is sensitivity. There is also specificity, true negatives divided by true negatives plus false positives, which has no standard machine-learning name of its own. Same quantities, different fields.

---

## The reusable template

The notebook wraps all of the above into a single function, `evaluate(y_true, y_pred, labels)`, built so that anyone in the course can copy it into their own notebook. Swap in your own `true_label` and `predicted_label` columns and run it; it prints the accuracy, the per-category precision, recall, and F1 breakdown, and plots the confusion matrix.

To use it on other data: get the data into a table with, at minimum, a `true_label` column and a `predicted_label` column, in matching row order and using the same category names in both. From a CSV, that's `df = pd.read_csv('your_file.csv')`. Then run `evaluate(df['true_label'], df['predicted_label'], labels=[...your categories...])`. The function doesn't care what the categories are or where the predictions came from, whether that's Gemini, another model, or a human.

---

## Wrap-up

There is now a working, reusable way to answer whether a model was actually right, rather than only whether it ran: accuracy for the overall picture, the confusion matrix for where it gets confused, and precision, recall, and F1 for how it's wrong on each category.

**Next in Session 2**: human-in-the-loop validation, and specifically at what point a wrong prediction actually reaches a business decision, and where a human needs to check it first. The per-category breakdown above is exactly the tool used to decide that.

Questions belong in the Circle community.
