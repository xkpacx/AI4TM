# Finding risk points in an AI pipeline

This is the text companion to the pipeline risk-points notebook. Session 3's first two guides each covered a single habit: don't hardcode an API key, don't paste real PII into a prompt. Both habits protect one moment. A real pipeline is never one moment: it's several systems passing data to each other, and the discipline that protects the first handoff says nothing about the fifth. This piece takes the instinct behind those two habits and applies it everywhere a pipeline moves data, using a framework security teams call **DLP**.

> **DLP (data loss prevention):** the discipline of stopping sensitive data from leaving a system it shouldn't leave.

**Time**: about 20 minutes
**Cost**: $0. Every API call in the course notebook is stubbed out with a fake function, not a real one, so nothing here touches a quota or a real key.

> **Disclaimer.** This is educational guidance, not a security audit. It teaches a way of looking at a pipeline, not a checklist that guarantees one is safe. A real production pipeline handling real customer data needs a real review by someone with security expertise. This material builds the habit of asking the right questions; it is not a substitute for that review. The course and its creators are not liable for a data exposure in a pipeline built or reviewed using only this material.

---

## Five questions to ask at every handoff

A handoff is any point where data moves from one system to another: a file loading into a notebook, a prompt going out to an API, a result getting written to disk, a message going to a dashboard or a chat channel. Each of the five questions below applies at every one of those points, whether or not it feels like a moment worth stopping for.

1. **What data is actually here?** Not what's assumed to be there, but what's actually in the variable, the file, or the row at this exact point.
2. **Does it need to be here at all?** This is minimization: the smallest amount of data that gets the job done is the safest amount that could. A classification prompt needs the feedback text; it has no use for the customer's name or email.
3. **Is anything sensitive marked as such, or is it just mixed in with everything else?** If nobody has decided which columns are PII, nobody can decide to drop them later.
4. **Where does this get logged, printed, or cached, and was that on purpose?** Debug prints, saved notebook outputs, and error logs are where this is most often overlooked, because the main code path gets reviewed and the debug line sitting beside it does not.
5. **Where does this leave your control entirely?** An **egress point** is anywhere data crosses out of a system you control: an API call, a webhook, a saved file, a Slack message. Every egress point is where a DLP problem becomes a real one.

> **This isn't hypothetical.** In March 2023, a bug in redis-py, an open-source caching library, caused OpenAI's own infrastructure to return one ChatGPT user's cached account data to a different, unrelated user. For roughly 1.2% of ChatGPT Plus subscribers, over a nine-hour window on March 20, this exposed another person's first and last name, email address, payment address, and the last four digits and expiration date of their credit card; full card numbers were never exposed. OpenAI traced the cause to canceled requests corrupting shared connections to its Redis cache, made worse by an unrelated server-side change that had, by coincidence, driven up how often requests were being canceled that day. ([OpenAI](https://openai.com/index/march-20-chatgpt-outage/); [BleepingComputer](https://www.bleepingcomputer.com/news/security/openai-chatgpt-payment-data-leak-caused-by-open-source-bug/))

The instinct is to picture a DLP failure as an attacker getting past a defense. Nothing here was breached in that sense. A caching layer, built to make the product faster and never intended to move data anywhere new, carried data across a boundary it was never supposed to cross, because of an ordinary software bug that most engineers wouldn't think to call a security control at all. That's exactly why question 4 names logging, printing, and caching specifically: they're the handoffs least likely to get the same scrutiny as an API call, precisely because they don't look like the pipeline's real job.

---

## The pipeline for this exercise

A small marketing pipeline, the kind this course builds throughout: a support team's customer feedback gets classified by topic so it can be routed and reported on. It runs in six stages, and each one is a handoff.

1. A CRM export lands in a shared folder, a CSV with customer ID, name, email, and feedback text.
2. A notebook loads that CSV.
3. Each feedback message is sent to Gemini for topic classification, the pattern from Session 2.
4. The prompt and response get printed to a cell for debugging while the notebook is being built.
5. Results, predictions plus the original data, are saved to a CSV for the reporting team.
6. If a request fails, the error gets logged so someone can follow up.

---

## What's wrong with the first version

The course notebook builds this pipeline with the problems still in it, as an exercise: run it, check it against the five questions above, and see what turns up before reading on. At least five DLP issues are planted in it.

**The key is hardcoded.** A `GEMINI_API_KEY` sits directly in the code, the exact problem Session 3's key guide opened with.

**The prompt sends name and email the classifier never needed.** The function that builds the prompt pulls in the customer's name and email alongside the feedback text, even though the task is topic classification and the feedback text alone answers it. Two PII fields travel to a third party for no reason tied to the task, a direct failure of question 2.

**The debug print leaks the same PII a second time.** Printing the full prompt for debugging puts the full name, email, and message into the cell's saved output, the same notebook-output risk Session 3's key guide covered for API keys, here applied to customer data instead. If that notebook is ever saved to GitHub with the output intact, that's a second, independent leak of the same information. As the OpenAI case above shows, it doesn't take an attacker for a debug artifact like this one to become the actual leak.

**The saved CSV keeps every column, including the PII ones.** Writing results to a file for a reporting team, without first deciding which columns are sensitive, means name and email ride along into a file that only needed the topic breakdown.

**There's no handling for a failed request**, which is easy to miss because it isn't in the code at all. An error log is exactly where a full prompt or a full exception, PII and all, tends to end up unredacted, because nobody treats an error path with the same scrutiny as the main one: the same blind spot question 4 names, one stage later.

---

## The hardened version

Same six stages, with each issue fixed at the point where it happens, reusing `redact_pii()` from Session 3's PII guide rather than inventing a new tool. Applying one technique at every handoff instead of just the obvious one is the entire lesson:

- No hardcoded key: loaded from Colab Secrets, per Session 1 and Session 3's key guide.
- Only feedback text is passed into the classification function; name and email never enter it, let alone the prompt built inside it.
- The debug print shows the redacted prompt, so the raw one never reaches the saved output.
- A failed request gets the same redaction as a successful one, since an exception message can carry the same data that was sent, and an error path is not exempt just because something went wrong.
- PII columns are dropped before anything is saved to a file that a reporting team, or a future git commit, might pick up.

---

## What a risk register would have caught

Go back to the OpenAI cache for a moment. Its actual fix, added after the leak, was to check that the data coming back from the cache matched the user asking for it: a one-line rule for a real egress point that nobody had written down until an incident forced the question. That's what a risk register is for: writing down, for every handoff in a pipeline, what data is present, whether it's an egress point, and what mitigates it, while the pipeline is still being planned rather than after something breaks. Security teams call this table a risk register, and it isn't a formality. It's the difference between a mitigation designed in from the start and the one OpenAI shipped only after roughly 1.2% of its paying subscribers had already been affected.

The course notebook builds one for the pipeline above: four columns (stage, data present, egress point yes/no, mitigation), one row per stage. The columns don't change from pipeline to pipeline; only the rows do.

---

## Wrap-up

`redact_pii()` and dropping PII columns both came from Session 3's earlier guide; nothing here is a new tool. What's new is applying them at every handoff in a pipeline instead of just the one that's easiest to spot, and treating logging and error paths as real egress points instead of exceptions to the rule. The five questions and the risk register are the parts meant to travel to a pipeline this course didn't build. The OpenAI case is a reminder of why that habit is worth keeping even when nothing about a system looks broken.

**Also in Session 3**: setting up and protecting API keys, and protecting PII when you use AI. This piece assumes both.

Questions belong in the Circle community.

---

## Sources

- [OpenAI, "March 20 ChatGPT outage"](https://openai.com/index/march-20-chatgpt-outage/)
- [BleepingComputer, "OpenAI: ChatGPT payment data leak caused by open-source bug"](https://www.bleepingcomputer.com/news/security/openai-chatgpt-payment-data-leak-caused-by-open-source-bug/)
