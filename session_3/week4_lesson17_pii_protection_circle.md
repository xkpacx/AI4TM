# Protecting PII when you use AI

This is the text companion to the PII protection notebook. Every AI pipeline in this course runs on data, and marketing data is full of **PII**, personally identifiable information: anything that can identify a specific, real person. This piece covers what counts as PII, the specific ways it ends up inside an AI prompt without anyone deciding to put it there, and the habits that keep it out.

**Time**: about 15 minutes
**Cost**: effectively $0 — the exercises here run on invented sample data, not a real dataset, and don't call a metered API beyond what Session 1 already set up

> **Disclaimer.** This is educational guidance, not legal advice. Whether a given dataset or use case complies with GDPR, CCPA, or another privacy law is a legal question that depends on jurisdiction, contract terms, and specifics this document can't see — consult qualified counsel or your organization's data protection lead for an actual compliance decision. Following the guidelines here reduces risk but doesn't make a use case compliant on its own, and the course and its creators are not liable for a PII exposure that happens despite them.

---

## What counts as PII

**PII (personally identifiable information)** is any data that can identify a specific, real person, on its own or combined with other data. In marketing work, PII shows up constantly and often without looking like a compliance problem.

**Directly identifying** data includes a full name, email address, phone number, physical address, a customer ID tied to a real person, or a payment card number. **Identifying in combination** covers data that only becomes personal once fields are joined: a ZIP code plus a birth date plus a gender can narrow down to one person even with no name attached, which is why "anonymized" data can quietly become identifying again once several fields are combined. **Special category data**, GDPR's term for information treated more strictly than ordinary PII, covers health information, race or ethnicity, religious belief, sexual orientation, and political opinion. A "why did you cancel" survey response that mentions a medical condition falls into this category the moment it's collected.

The synthetic dataset used elsewhere in this course, Session 2's five-category feedback set, has none of this by design; it's invented text with no real person behind it. The moment a real CRM export, a support ticket log, or a survey response file is swapped in, PII is very likely inside it.

---

## Why this matters in an AI pipeline

An AI API call is a request to a third party. Anything pasted into a prompt, or included in a dataframe passed to an API, leaves the sender's machine and reaches the provider's servers, the same way an email leaves an inbox the moment it's sent.

Session 1 covered that data-use policy depends on whether a key is billed under a "Paid tier" project: Google's terms say Paid tier Gemini usage isn't used for training and skips human review, but that protection is about the model *learning from* the data, not about whether the data was *sent* at all. A PII exposure isn't only "did the model train on this," it's also "did this personal data leave a system it was never supposed to leave," which can matter to a client contract or a privacy law regardless of what the AI provider does with it afterward.

Session 3's API key guide covered one version of this already: pasting a real key into a debugging chat. The same habit, copying something real "just to test" or "just to debug," is exactly how PII gets into a prompt too, and it's usually not the planned, obvious use case (like a listed "process customer feedback" prompt) that leaks it. It's the unplanned one: a quick support-ticket summary, a "can you fix this error" paste that includes an API response with real customer fields, a CRM export opened to check formatting before the real anonymization pass.

---

## Guidelines: how not to expose it

1. **Default to synthetic or anonymized data for testing and coursework.** This is the course's own rule, and it's the right default for any prompt development: get the prompt working on invented data first, and only point it at real data once the prompt itself is proven.
2. **Redact before pasting.** The same habit as Session 3's key guide: before pasting a support ticket, a CRM row, or an error trace into a prompt or an AI chat, scan it for names, emails, phone numbers, and addresses, and replace them with placeholders.
3. **Aggregate instead of pasting raw rows, whenever the task allows it.** "What's the average time-to-resolution for the shipping-delay category" doesn't need a single row of real customer data — a pre-aggregated summary answers it just as well and never touches PII in the first place.
4. **Strip identifying columns programmatically, not by eye.** Scanning a spreadsheet visually for anything that looks personal misses things. Drop or mask the columns by name in code before the dataframe reaches an API call — the course notebook demonstrates the pattern.
5. **Know the data-use policy before a real run, not after.** Same principle as Session 1: check whether the key in use is on a "Paid tier" project, and confirm with the organization whether real customer data is permitted through this API at all — that's a policy decision, not a technical one, and it should be made before the first real request, not discovered afterward.
6. **Treat special category data as a hard stop, not a redaction problem.** Health, religious, or similarly sensitive mentions inside a message aren't safely handled by swapping out a name. If a dataset is expected to contain this kind of content regularly, that's a conversation with whoever owns data protection at the organization, not a prompt-engineering problem.

---

## A simple PII redactor

The same shape-matching idea from the API key guide applies here: a regular expression can catch anything that's structured, an email address, a phone number, because those follow a predictable pattern. It can't reliably catch a name in free text, because names don't have a fixed shape the way an email address does — a real limitation worth knowing rather than trusting the tool past what it can do. The course notebook builds this as a small function, `redact_pii()`, run over a handful of invented example messages to make the idea concrete.

---

## Scrubbing a dataframe before it reaches an API call

Redacting free text catches some PII. The more reliable pattern, when data is already in a table, is to decide which columns are identifying and drop or mask them before anything is sent, rather than trying to catch every mention after the fact. The course notebook works through this on an invented customer dataframe: a `PII_COLUMNS` list naming the identifying fields explicitly, then `.drop(columns=PII_COLUMNS)` before the data goes anywhere near an API call.

---

## If PII gets exposed anyway

1. **Stop the pipeline, don't just fix the prompt.** If a run just sent real PII somewhere it shouldn't have, pause the loop or job before it processes more rows — a fix applied mid-run still lets the remaining rows through on the old behavior.
2. **Check what actually left the building.** Which rows, which fields, sent to which provider, under which account. This is the fact-finding step before anyone decides what else needs to happen.
3. **Follow the organization's incident process, if one exists.** Most companies handling customer data have a data protection lead, a DPO, or a documented breach process for exactly this. If this is coursework rather than a live client engagement, there's no such process to follow, but building the habit of checking for one is the point.
4. **Don't guess at legal exposure.** Whether a given exposure triggers a notification requirement under GDPR, CCPA, or another law is genuinely fact-specific and outside what this guide, or a prompt-engineering habit, can answer. That's what the disclaimer at the top means in practice: a real exposure is a legal and organizational question, not just a technical cleanup.
5. **Fix the process gap, not just the instance.** If real PII made it into a prompt once, the guideline that would have caught it (redact first, aggregate first, drop the columns in code) is the thing worth reinforcing, not just deleting the one bad message.

---

## Wrap-up

PII shows up in marketing data far more often than it looks like a compliance problem: a name in a support ticket, an email in a CRM export, a health mention in a churn survey. The guidelines here are the same shape as the API key guide before it: default to synthetic or aggregated data, redact or strip before anything real reaches a prompt, and know the data-use policy before the first real request rather than after.

**Next in Session 3**: identifying risk points across a full AI pipeline and applying data-loss-prevention (DLP) principles end to end, not just at the prompt.

Questions belong in the Circle community.
