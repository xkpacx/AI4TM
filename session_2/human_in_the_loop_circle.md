# Human-in-the-loop validation: deciding where a person needs to check

This is the text companion to the human-in-the-loop notebook. It builds directly on the evaluation template. Once you know how right a model is, per category, the next question is where a person still needs to look at its work before it reaches a real decision. Reviewing everything doesn't scale; reviewing nothing lets a quiet failure run for months before anyone notices. This piece is about landing somewhere in between, deliberately, using the numbers evaluation gives you instead of a gut feeling.

**Time**: about 25 minutes
**Cost**: a few cents at most for the LLM-as-judge section — estimate first, per the token cost guide; the BLEU/ROUGE section runs entirely locally and costs nothing

---

## Recap: what the evaluation template already gave you

The evaluation template scored a classifier's predictions per category: precision (does it cry wolf), recall (does it miss real cases), and a confusion matrix showing which categories the model mixes up with which. That's the right tool for one specific kind of task, classification, where every item has one fixed, correct label to compare against.

Two things that template doesn't cover, and this piece does. What do you do with a task that has no fixed right answer? A tagline, a subject line, a support reply draft — there's no single correct output sitting in a spreadsheet to check against, so accuracy-style metrics don't apply. And once a model's precision and recall are known, what do you actually do with that number? This is where human-in-the-loop validation comes in: deciding, category by category, whether a person needs to check the output before it reaches someone, or whether the model's track record is good enough to let it through unchecked.

---

## Two kinds of tasks, two kinds of metrics

Classification tasks, the five-category feedback classifier from the evaluation template, have a fixed answer key: every item has one correct label, so accuracy, precision, and recall all work by comparing a prediction against that fixed answer.

Generative tasks don't have that. Ask a model to write three subject lines for a promotion, and there's no single "correct" subject line to compare against; several very different outputs could all be equally good. Comparing generated text this way needs a different toolkit: BLEU and ROUGE when there's one reference text worth comparing against, and LLM-as-judge when there isn't, or when what actually matters, tone, brand voice, whether a claim is accurate, isn't something word-overlap can measure at all.

---

## BLEU and ROUGE: what they actually measure

Both compare a generated piece of text against one reference text already trusted, and both work by counting overlapping words and short phrases, n-grams, meaning sequences of *n* consecutive words, rather than understanding meaning.

**BLEU** (bilingual evaluation understudy) started in machine translation. It's precision-focused: of the words and phrases in the generated text, how many also appear in the reference? A generated sentence that repeats the reference almost word-for-word scores high; a fluent, accurate paraphrase that uses different words scores low, because BLEU has no way to know the two mean the same thing.

**ROUGE** (recall-oriented understudy for gisting evaluation) started in summarization. It's recall-focused: of the words and phrases in the reference, how many made it into the generated text? Same blind spot as BLEU, just measured from the other direction.

The course notebook demonstrates both on a small refund-confirmation example: an exact match scores near 1.0 on BLEU, while a fluent paraphrase using entirely different words scores around 0.18, despite meaning exactly the same thing. ROUGE shows the same pattern on a summarization-style example, dropping noticeably between a close paraphrase and a same-meaning, differently-worded version.

---

## Why these are a weak fit for most marketing generative work

Both demos show the same failure: text that means the same thing, phrased differently, scores noticeably lower than a near-identical copy, even though a person reading both would call the paraphrase just as good, maybe better.

That's fine for tasks with one genuinely correct answer, like translating a sentence or summarizing a document against a reference summary someone wrote. It breaks down for marketing generation, where the whole point is usually that there isn't one right answer: three different taglines for the same brief can all be strong, share almost no words, and still all deserve a high score.

Know these two names, they show up constantly in papers and vendor benchmarks, but don't reach for them as a default metric on a generative marketing task. Reach for them when there's a genuine one-right-answer reference to compare against (translation, summarization against a source document), and reach for LLM-as-judge or human review everywhere else.

---

## LLM-as-judge: scoring generated text with a second model

The pattern: give a model the generated output, the brief or instructions it was written from, and a rubric, a short list of what "good" means for this task, and ask it to score against that rubric. It scales further than a person reviewing every item, and it can judge things BLEU and ROUGE can't touch, like tone or whether a claim in the copy is actually supported by the brief.

It comes with its own failure modes, worth knowing before trusting a judge score. **Verbosity bias**: judges tend to rate longer answers higher, independent of quality. **Position bias**: when comparing two outputs side by side, judges tend to favor whichever one comes first in the prompt. **Self-preference**: a model can rate output from its own model family more favorably than a genuinely neutral judge would.

None of these make LLM-as-judge useless, but they mean a judge needs validating against real human ratings on a sample before its scores get trusted at scale, the same way a new hire's judgment gets sanity-checked before they take the whole queue.

The course notebook builds a small `judge_copy()` function around Gemini: a rubric scoring tone, clarity, and accuracy from 1 to 5, with the model returning structured JSON so the scores can be logged and aggregated rather than read as prose each time.

---

## Deciding where a human checkpoint actually belongs

Metrics, whichever kind, only matter once they change what happens next. Applied to the five-category classifier from the evaluation template: for each category, ask two questions. Is this a high-stakes category, one where a false negative (a miss) costs real money or a real customer, the way Returns & Refunds does if it feeds a churn-prevention workflow? And separately, is the model's recall on this category good enough to trust without a second look?

A category can fail either question on its own. A high-stakes category with excellent recall still deserves a checkpoint, because "usually right" isn't the same guarantee as "always right" when the cost of being wrong is high. A low-stakes category with poor recall doesn't need a person watching every single item, but it does need someone to notice before the miss rate quietly gets worse.

The course notebook builds this as a `route_for_review()` function: high-stakes categories always route to mandatory review regardless of recall; categories below a set recall threshold route to mandatory review too; everything in between gets a spot-check sample; the rest auto-approves. Applied to five example categories, it correctly flags Returns & Refunds for mandatory review purely because it's high-stakes (its recall of 0.93 sits in the middle band, so recall alone would have only routed it to a spot-check sample), and flags Pricing & Billing for mandatory review purely because its recall is too low — two different reasons landing on the same outcome.

---

## Sampling for everything that isn't mandatory

Spot-check and auto-approve aren't the same as never look. A category with good recall today can drift, a model update, a new slang term, a seasonal shift in what customers write about, and nothing in a single evaluation run warns anyone that's happening. The fix isn't reviewing everything; it's a standing sample that keeps a person's eye on the categories not being checked exhaustively.

A simple version: review 100% of anything routed to mandatory review, a fixed percentage of anything routed to spot-check, and log every disagreement between the model and the reviewer. If the disagreement rate on a spot-checked category starts climbing, that's the signal to re-run the evaluation template and reconsider the routing, not a signal to just increase the sample size and hope.

The course notebook implements this as a `build_review_queue()` function: every mandatory-review row goes in, half of every spot-check row goes in (a rate that's a deliberate choice, not a fixed rule), and nothing from auto-approve does.

A queue is only a to-do list, not the review itself — someone still has to open each row and decide. The notebook closes that loop with a `record_review()` function: it takes one queued row and a verdict of agree, disagree (with a corrected label), or ambiguous, logs it, and groups the log by category to produce the disagreement rate the section above says to watch. That's the number that turns "log every disagreement" from advice into something a reviewer can actually act on.

---

## Wrap-up

Two separate skills, both extending what the evaluation template started. For generative tasks with no fixed right answer, BLEU and ROUGE work only when there's a genuine reference text to compare against, and LLM-as-judge fills the gap everywhere else, once validated against real human ratings. For any task with metrics, per-category precision and recall, plus a plain judgment call about what's high-stakes, decide where a human checkpoint is mandatory, where a sample is enough, and where the model's track record earns it a pass. Once that queue exists, `record_review()` is what actually closes the loop.

Neither replaces judgment. The recall thresholds and the high-stakes list in the course notebook are placeholders; the actual numbers are a business decision, made by the same people evaluation itself is for.

Questions belong in the Circle community.
