# Human-in-the-loop validation: deciding where a person needs to check

**Time**: about 30 minutes
**Cost**: a few cents at most for the LLM-as-judge section, estimate first per the token cost guide; the BLEU/ROUGE section runs entirely locally and costs nothing

---

## What "in the loop" actually means

**Human-in-the-loop validation** is a person checking a sample of what a model produced, comparing each item against the original input and their own judgment of what a correct answer looks like, and recording whether they agree. That record does two things. It catches a wrong output before it reaches whoever depends on it, whether that's a customer, a report, or another automated step further down the pipeline. And, accumulated across many checks, it builds a track record that says whether a model can be trusted on a given task without a person checking every item.

Take a company that routes incoming customer messages into five categories, shipping and delivery, product quality, pricing and billing, customer service, and returns and refunds, using a classifier scored the way the evaluation template lesson describes. Applied to that system, human-in-the-loop validation means a person opens some of those categorized messages, reads the original text, checks whether the category assigned actually matches what the message is about, and records one of three outcomes: the assignment was right, it was wrong (and what the correct category actually is), or the message is genuinely ambiguous even to a person reading it carefully.

Everything below, from the two text-generation metrics that open this piece to the review process that closes it, answers one question about a system like that: which messages need a person to check them, and what does checking actually involve, end to end?

---

## Recap: what the evaluation template already gave you

The evaluation template scored a classifier's predictions per category: **precision** (does it cry wolf), **recall** (does it miss real cases), and a confusion matrix showing which categories the model mixes up with which. That's the right tool for one specific kind of task, classification, where every item has one fixed, correct label to compare against.

Two things that template doesn't cover, and this piece does. First, what happens with a task that has no fixed right answer? A tagline, a subject line, a support reply draft, there's no single correct output sitting in a spreadsheet to check against, so accuracy-style metrics don't apply. Second, once a model's precision and recall on a classification task are known, what do you actually do with that number? A score on its own doesn't tell a team where to put a reviewer's time. That decision, and what a reviewer actually does once a checkpoint exists, is the subject of the second half of this piece.

---

## Two kinds of tasks, two kinds of metrics

Classification tasks, the five-category feedback classifier above, have a fixed answer key: every item has one correct label, so accuracy, precision, and recall all work by comparing a prediction against that fixed answer.

Generative tasks don't have that. Ask a model to write three subject lines for a promotion, and there's no single "correct" subject line to compare against; several very different outputs could all be equally good. Comparing generated text needs a different toolkit: BLEU and ROUGE when there's one reference text worth comparing against, and LLM-as-judge when there isn't, or when what actually matters, tone, brand voice, whether a claim is accurate, isn't something word-overlap can measure at all.

---

## BLEU and ROUGE: what they actually measure

Both **BLEU** and **ROUGE** compare a generated piece of text against a **reference text**: a human-written or human-approved example of what a correct output looks like for that specific input. A reference text only exists for tasks with a genuinely correct answer, the actual professional translation of a specific sentence, the actual summary a person already wrote for a specific document, since the whole method depends on having one trusted answer to measure against.

Both metrics work the same underlying way: counting overlapping **n-grams**, meaning sequences of *n* consecutive words, between the generated text and the reference. A **1-gram** is a single word. A **2-gram** is two consecutive words, "was refunded" rather than those two words appearing anywhere in the sentence. Both scores run from **0.0** (no shared words or phrases at all) to **1.0** (the generated text matches the reference exactly).

**BLEU** ([Papineni et al., 2002](https://aclanthology.org/P02-1040/)) started in machine translation. It's precision-focused: of the words and phrases in the generated text, how many also appear in the reference? A generated sentence that repeats the reference almost word-for-word scores high; a fluent, accurate paraphrase that uses different words scores low, because BLEU has no way to know the two mean the same thing.

**ROUGE** ([Lin, 2004](https://aclanthology.org/W04-1013/)) started in summarization. It's recall-focused: of the words and phrases in the reference, how many made it into the generated text? Same blind spot as BLEU, measured from the other direction.

Here's what that looks like on two short sentences. Reference: "the customer was refunded within three business days after the return was received." A fluent paraphrase of it: "the refund was processed within three business days of the return arriving." At the word level, 8 of the 12 words in the paraphrase also appear in the reference, a raw overlap of 0.67; words like "refund," "processed," and "arriving" don't match anything, since the reference uses "customer," "refunded," and "received" instead. Read as consecutive word pairs instead of single words, the overlap drops further, to 4 of 11 pairs (0.36), because a single substituted word breaks both the pair before it and the pair after it, even though every pair sits in exactly the position it does in the reference. Extended out the way BLEU actually scores it, this pair lands around **0.25**, against **1.0** for a sentence that repeats the reference exactly.

> **Why "different words" costs more than it looks like it should.** The drop from 0.67 to 0.36 above isn't because word order changed, every shared word sits in the same slot it occupies in the reference. It's because a single synonym swap breaks every word-pair it touches. This is the concrete version of BLEU and ROUGE's blind spot: they penalize different wording far more heavily than they reward preserved meaning.

The same pattern shows up on ROUGE with a slightly different pair of sentences: a close paraphrase scores 0.72 on one-word overlap and 0.64 on longest-shared-sequence overlap, while a version that keeps the same meaning but rewords more freely, "refunds typically take a few days once we get the returned item," drops to 0.32 on both. The meaning survives the rewording; the score doesn't.

---

## Why these are a weak fit for most marketing generative work

Both examples above show the same failure: text that means the same thing, phrased differently, scores noticeably lower than a near-identical copy, even though a person reading both would call the paraphrase just as good, maybe better.

That's a reasonable trade-off for tasks with one genuinely correct answer, translating a sentence, summarizing a document against a reference summary someone wrote. It breaks down for marketing generation, where the whole point is usually that there isn't one right answer: three different taglines for the same brief can all be strong, share almost no words, and still all deserve a high score.

Know these two names, they show up constantly in papers and vendor benchmarks, but don't reach for them as a default metric on a generative marketing task. Reach for them when a genuine one-right-answer reference exists to compare against, and reach for LLM-as-judge or human review everywhere else.

---

## LLM-as-judge: scoring generated text with a second model

The pattern: give a model the generated output, the brief or instructions it was written from, and a **rubric**, a short list of what "good" means for this task, and ask it to score against that rubric. It scales further than a person reviewing every item, and it can judge things BLEU and ROUGE can't touch, like tone or whether a claim in the copy is actually supported by the brief. In practice this looks like a short scoring prompt: tone, clarity, and accuracy, each rated on a simple scale, with the model returning its ratings in a structured, consistent shape rather than a paragraph of prose, so they can be logged and compared across many pieces of copy without someone reading through free text every time.

This approach comes with its own failure modes, documented directly in the paper that established LLM-as-judge as a standard technique ([Zheng et al., 2023](https://arxiv.org/abs/2306.05685)), worth knowing before trusting a judge score:

- **Verbosity bias**: judges tend to rate longer answers higher, independent of quality.
- **Position bias**: when comparing two outputs side by side, judges tend to favor whichever one comes first in the prompt.
- **Self-preference** (the paper calls this self-enhancement bias): a model can rate output from its own model family more favorably than a genuinely neutral judge would.

None of these make LLM-as-judge useless, but they mean a judge needs validating against real human ratings on a sample before its scores get trusted at scale, the same way a new hire's judgment gets sanity-checked before they take the whole queue.

---

## Deciding where a human checkpoint actually belongs

Metrics, whichever kind, only matter once they change what happens next. For this week's five-category classifier, the decision runs in two steps, asked in order, not at once.

**Step one: is this a high-stakes category?** A category is high-stakes when a false negative, a miss, costs real money or a real customer. Returns & Refunds is high-stakes in this system because a missed return request can feed straight into a churn-prevention workflow; missing it means a customer who should have been flagged as at-risk never gets flagged at all. If the answer is yes, the category goes to mandatory review and the decision stops there. Recall never gets checked for a high-stakes category, because "usually right" isn't the same guarantee as "always right" when the cost of being wrong is high.

**Step two, only asked if the answer to step one was no: is this category's recall good enough to trust without a second look?** For this week's system, a category with recall below 0.85 goes to mandatory review regardless of stakes; one between 0.85 and 0.95 goes to a spot-check sample; one at 0.95 or above is trusted to auto-approve. Both lines are a business decision, not a universal rule to copy as-is; a different company might draw them somewhere else entirely.

Applied to this week's five categories:

| Category | High-stakes? | Recall | Ends up as |
|---|---|---|---|
| Returns & Refunds | yes | 0.93 | mandatory review |
| Pricing & Billing | no | 0.80 | mandatory review |
| Product Quality | no | 0.91 | spot-check sample |
| Customer Service | no | 0.96 | auto-approve |
| Shipping & Delivery | no | 0.97 | auto-approve |

Returns & Refunds is worth sitting with specifically, because it's the one row where the two steps could have disagreed. Its recall, 0.93, sits in the same range as Product Quality's, the range that would normally mean a spot-check sample rather than full review. The stakes flag overrides that entirely: a high-stakes category is reviewed in full at any recall, which is exactly why step one has to run before step two, not alongside it.

---

## What a mandatory review actually involves

A routing decision produces a queue, not a finished review. Every mandatory-review item goes into that queue in full; spot-check items go in at some smaller, deliberately chosen rate, since a category that's earned a spot-check doesn't need every single item checked to catch drift early. Either way, a queue is a to-do list. Someone still has to open each item and decide.

That means a reviewer reads the original customer message, checks the category the model assigned it against, and lands on one of three outcomes:

- **it was right**, and the item moves on as-is.
- **it was wrong**, and the reviewer notes what the category should actually have been.
- **it's genuinely ambiguous**, which is informative on its own; it usually means the category boundary itself needs work, not that the model made a mistake.

Recording those outcomes over time produces something a single evaluation score can't: how a category is actually doing right now, not just when it was last measured formally. In a small trial run of this process against a handful of flagged messages, one category came back with a mismatch on half the messages reviewed, another on a quarter, and a third had no mismatches at all. With only a few messages reviewed, none of that says a category is actually drifting on its own. It's the same number that, tracked over weeks instead of one afternoon, tells a team when a category needs a second look before its formal recall score is ever remeasured.

---

## Sampling for everything that isn't mandatory, and what drift actually looks like

Spot-check and auto-approve aren't the same as never look. A category with good recall today can drift, and nothing in a single evaluation run warns anyone that it's happening, because that run measured one moment in time and doesn't update itself.

Drift, concretely, tends to look ordinary rather than dramatic. A new product line launches, and customers start describing defects in phrasing the model never saw in training. A common complaint picks up a new slang term. A payment provider gets added, and billing messages start naming something the model has never encountered. None of these show up in the recall number a category was originally routed on; they show up later, as a rising disagreement rate in the ongoing spot-check.

A simple version of the fix: review every mandatory-review item, a fixed share of every spot-check item, and log every disagreement the way the review process above does. If a spot-checked category's disagreement rate climbs across successive samples, say from a handful of percent to something noticeably higher over a few weeks, that's the signal to re-run the evaluation and reconsider the routing, not a signal to just widen the sample and hope the number settles back down.

---

## Wrap-up

Two separate skills, both built on top of what the evaluation template started. For generative tasks with no fixed right answer, BLEU and ROUGE work only when a genuine reference text exists to compare against, and LLM-as-judge fills the gap everywhere else, once validated against real human ratings. For any task with metrics, a two-step decision, stakes first, then recall, decides where a human checkpoint is mandatory, where a sample is enough, and where a model's track record earns it a pass. And a checkpoint only means something once someone actually works the queue: a person's verdict, logged and tracked, turns "watch for drift" from advice into a number a team can act on.

Neither replaces judgment. The recall thresholds and the high-stakes list used above are one week's business decision, not a rule to import as-is; the actual numbers belong to whoever evaluation itself is for.

Questions belong in the Circle community.
