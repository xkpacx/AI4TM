# Generating and evaluating a landing page brief

This is the text companion to the landing page brief notebook. **Time**: about 35-45 minutes, most of it spent waiting on model calls. **Cost**: a few cents at most on Gemini's cheapest models -- the notebook makes on the order of 50-60 small calls, most of them in the evaluation section.

The previous notebook this week built an agent that reads a content gap table through three tools and writes a short recommendation. This piece goes one level deeper on a single output: a full **landing page brief**, generated for the single highest-demand gap, and then -- this is where most of the notebook's time actually goes -- checked, not just read.

---

## Writing the taxonomy down before labeling anything, applied to a brief

The earlier lesson this week made a specific methodological point about intent labeling: the taxonomy gets written down before anything gets labeled, because the categories a reviewer can check against have to exist before the checking starts. The same discipline applies here, one level removed. The brief's ten required fields -- the target question, the real queries it targets, its stated intent, why the current best-matching page falls short, a title, a meta description, an H1, a section outline, the existing pages it should link to, and a note on length and format -- are fixed before a single brief gets generated. A brief with no fixed shape is unreviewable, because there's nothing consistent to check across more than one of them.

---

## A guardrail, not a hope

The brief-generation function refuses to run on a cluster that isn't flagged as a gap, checked in plain code before any model call happens. That's a deliberate echo of a pattern Session 5 introduced under a different name: a model-written Cypher query only got executed after a check for destructive keywords, because trusting a generation step to police itself is a real, avoidable risk. Here the equivalent risk is a brief written for a page that's already adequately served -- work a content team would waste time on, and confidently, since nothing about a well-written brief signals that the underlying gap it's answering isn't real. Catching that in code, before the API call that would produce a plausible-looking brief for a non-problem, costs nothing and closes an entire failure mode.

---

## Reusing the Week 3 template on a brief instead of a classifier

Session 2's evaluation template compares predictions against known correct answers. Reusing it here means reframing "is this brief good" as a set of pass/fail checks, each one a small classification task with a **true label** and a **predicted label** for the same yes/no question.

The predicted label comes from an **LLM judge**: the model itself, shown the brief alongside the same real data it was supposed to be grounded in, asked to answer PASS or FAIL. For four of the five criteria, the true label comes from something different -- code that recomputes the underlying fact directly. Does this URL actually appear in the page list. Does this query actually appear in the cluster. Does the reasoning actually name the real best-matching page. None of those require a second round of model opinion standing in for ground truth; they're checkable mechanically, the same way a returns policy either does or doesn't cover a specific defect.

> That split exists because of a finding this week's overview named directly: the SIGIR 2025 comparison between weak supervision and LLMs found the language model ahead on recall and behind on precision across every configuration tested, attributed to a generative model favoring inclusiveness over strict relevance. A model judging its own kind of output is exposed to the same bias. Comparing its PASS against a fact recomputed independently is what actually tests whether that bias shows up here, rather than assuming it doesn't.

One criterion, whether the section outline is genuinely specific rather than generic filler, has no objective check available. Judging that requires reading the actual headings, so it's scored by the judge alone and named explicitly as the one check this method can't automate -- printed out at the end of the notebook for exactly that reading to happen.

---

## Reading the evaluation, not just running it

`evaluate()` on the whole sample answers one question: overall, does the judge's PASS mean what it says. Breaking the same table down by criterion answers a sharper one, and it's worth working through the two different kinds of failure that can show up there rather than treating every low number the same way.

A low **objective pass rate** on a criterion -- regardless of what the judge says about it -- means the brief-writing step itself isn't carrying forward what it was given. It had the real queries, the real intent, the real best-matching page sitting directly in its prompt, and produced something that doesn't reflect them anyway. That's a failure in generation, not in retrieval, and it's only cleanly attributable to generation because this notebook controls the gap table and can assert it's correct going in. A real pipeline doesn't get that assertion for free; ruling out an upstream clustering or mapping error first is a genuine, separate step.

A gap between the **objective true label and the judge's predicted label**, on a criterion where the objective pass rate is actually high, points somewhere else: not at the brief, but at the judge. That's the more concerning pattern of the two, because it means the automated check itself can't be trusted at face value on that criterion -- exactly the asymmetry the SIGIR comparison predicts, and exactly why this notebook didn't settle for one round of model judgment as its only measurement.

---

## What gaps to carry forward

The evaluation in this exercise rests on one gap, regenerated five times. That's a demonstration of the method, not a benchmark of the pipeline's typical reliability -- a real rollout would want this same table run across every flagged gap, not only the highest-demand one, before treating any pass rate here as a property of the system rather than of this one question. The gap table itself, in this notebook, uses a placeholder median-score threshold rather than the previous notebook's actual three-step human-labeling procedure, which means the set of clusters eligible to be briefed at all hasn't had a real reviewer's judgment applied to it yet. And `concrete_section_outline` stays a human-only check by design, not by oversight -- a reminder that not every part of a quality judgment reduces to a fact a script can recompute, and that the honest response to that is naming the gap directly rather than automating past it without saying so.

**This closes the course.** Five weeks built the pieces this pipeline actually draws on -- evaluation metrics and human-in-the-loop review, the compliance instincts that show up here as a guardrail refusing to brief a non-gap, comfort with synthetic data building the dataset this week runs on honestly, and the groundedness question Session 5 first asked of a graph answer, asked again here of a landing page brief.
