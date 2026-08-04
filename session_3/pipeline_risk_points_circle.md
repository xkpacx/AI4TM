# Finding risk points in an AI pipeline

This is the text companion to the pipeline risk-points notebook. Session 3's first two guides covered two single habits: don't hardcode an API key, don't paste real PII into a prompt. A real pipeline isn't one habit applied once, it's several systems handing data to each other, and each handoff is a place something can go wrong. This piece applies the same instincts across a full pipeline, using a framework security teams call **DLP: data loss prevention**, the general discipline of stopping sensitive data from leaving a system it shouldn't leave.

**Time**: about 20 minutes
**Cost**: $0 — every API call in the course notebook is stubbed out with a fake function, not a real one, so nothing here touches a quota or a real key

> **Disclaimer.** This is educational guidance, not a security audit. It teaches a way of looking at a pipeline, not a checklist that guarantees one is safe. A real production pipeline handling real customer data needs a real review by someone with security expertise — this material builds the habit of asking the right questions, not a substitute for that review. The course and its creators are not liable for a data exposure in a pipeline built or reviewed using only this material.

---

## Five questions to ask at every handoff

A handoff is any point where data moves from one system to another: a file loading into a notebook, a prompt going out to an API, a result getting written to disk, a message going to a dashboard or a chat channel. At each one, ask five questions.

**What data is actually here?** Not what's assumed to be here, what's actually in the variable, the file, or the row at this exact point. **Does it need to be here at all?** This is minimization: the smallest amount of data that gets the job done is the safest amount. A classification prompt needs the feedback text; it doesn't need the customer's name or email. **Is anything sensitive marked as such, or is it just mixed in with everything else?** If nobody has decided which columns are PII, nobody can decide to drop them later. **Where does this get logged, printed, or cached, and was that on purpose?** Debug prints, saved notebook outputs, and error logs are the classic blind spot: the "real" code path gets reviewed, and the debug line right next to it doesn't. **Where does this leave your control entirely?** An **egress point** is anywhere data crosses out of a system you control: an API call, a webhook, a saved file, a Slack message. Every egress point is where a DLP problem becomes a real one.

---

## The pipeline for this exercise

A small marketing pipeline, the kind this course builds throughout: a support team's customer feedback gets classified by topic so it can be routed and reported on. Six stages, each a handoff:

1. A CRM export lands in a shared folder, a CSV with customer ID, name, email, and feedback text.
2. A notebook loads that CSV.
3. Each feedback message is sent to Gemini for topic classification, the pattern from Session 2.
4. The prompt and response get printed to a cell for debugging while the notebook is being built.
5. Results, predictions plus the original data, are saved to a CSV for the reporting team.
6. If a request fails, the error gets logged so someone can follow up.

---

## What's wrong with the first version

The course notebook builds this pipeline with the problems still in it, as an exercise: run it, then check it against the five questions above before reading on. At least five DLP issues are planted in it.

**The key is hardcoded.** A `GEMINI_API_KEY` sitting directly in the code — exactly the problem Session 3's key guide opened with.

**The prompt sends name and email the classifier never needed.** The function builds its prompt from the customer's name, email, and feedback, but the task is topic classification; feedback text alone answers it. Two PII fields travel to a third party for no reason tied to the task.

**The debug print leaks the same PII a second time.** Printing the full prompt for debugging puts the full name, email, and message into the cell's saved output, the same notebook-output risk Session 3's key guide covered for API keys, here applied to customer data. If that notebook is ever saved to GitHub with the output intact, that's a second, independent leak of the same information.

**The saved CSV keeps every column, including the PII ones.** Writing results to a file for a reporting team, without deciding first which columns are sensitive, means name and email ride along into a file that only needed the topic breakdown.

**There's no handling for a failed request**, which is easy to miss because it isn't in the code at all. An error log is exactly where a full prompt or a full exception, PII and all, tends to end up unredacted, because nobody treats an error path with the same scrutiny as the main one.

---

## The hardened version

Same six stages, each issue fixed at the stage where it happens, reusing `redact_pii()` from Session 3's PII guide rather than inventing a new tool. The point isn't a new technique, it's applying the same one at every handoff instead of just the obvious one:

- No hardcoded key — loaded from Colab Secrets, per Session 1 and Session 3's key guide.
- Only feedback text is passed into the classification function; name and email never enter it, let alone the prompt built inside it.
- The debug print shows the redacted prompt, never the raw one.
- A failed request gets the same redaction as a successful one — an exception message can easily contain the same data that was sent, and it isn't exempt just because something went wrong.
- PII columns are dropped before anything is saved to a file that a reporting team, or a future git commit, might pick up.

---

## A reusable risk register

The habit that scales beyond this one example: write down every handoff in a pipeline, what data is present at it, whether it's an egress point, and what mitigates it, before building the pipeline, not after something goes wrong. This is the same table format a security team would call a risk register. The course notebook builds it as a small table with four columns — stage, data present, egress point (yes/no), mitigation — filled in for all six stages of the example pipeline. Adapt the rows to a real pipeline; the columns don't change.

---

## Wrap-up

The tools here aren't new: `redact_pii()` and dropping PII columns both came from Session 3's earlier guide. What's new is applying them at every handoff in a pipeline instead of just the one that's easiest to spot, and treating logging and error paths as real egress points instead of exceptions to the rule. The five questions and the risk register are the parts meant to travel to a pipeline this course didn't build.

**Also in Session 3**: setting up and protecting API keys, and protecting PII when you use AI — this piece assumes both.

Questions belong in the Circle community.
