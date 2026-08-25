# Protecting personal data when you use AI

This is the text companion to the personal data protection notebook. Every AI pipeline in this course runs on data, and marketing data is full of **personal data**: under GDPR, "any information relating to an identified or identifiable natural person" ([GDPR Article 4(1)](https://gdpr-info.eu/art-4-gdpr/)). This piece covers what counts as personal data, the specific ways it ends up inside an AI prompt without anyone deciding to put it there, and the habits that keep it out.

**Time**: about 15 minutes
**Cost**: effectively $0. The exercises here run on invented sample data, not a real dataset, and don't call a metered API beyond what Session 1 already set up

> **Disclaimer.** This is educational guidance, not legal advice. Whether a given dataset or use case complies with GDPR, CCPA, or another privacy law is a legal question that depends on jurisdiction, contract terms, and specifics this document can't see. Consult qualified counsel or your organization's data protection lead for an actual compliance decision. Following the guidelines here reduces risk but doesn't make a use case compliant on its own, and the course and its creators are not liable for a personal-data exposure that happens despite them.

---

## What counts as personal data

> **Personal data**, per GDPR Article 4(1): "any information relating to an identified or identifiable natural person." A person counts as identifiable if they can be identified "directly or indirectly," including "by reference to an identifier such as a name, an identification number, location data, an online identifier." That covers more than fields that name someone outright.

**PII (personally identifiable information)** is the term most marketing and data teams reach for first, and it's worth knowing why this piece leads with GDPR's term instead. PII doesn't appear anywhere in the GDPR text. It's a term drawn from US frameworks (NIST's definition among them) and it's typically read narrowly, as fields that themselves point to one specific person: a name, an email, a government ID number. GDPR's "personal data" is deliberately broader. It doesn't require a field that identifies someone by itself; it only requires that the information *relates to* a person who could be identified, directly or with other data. A team that only screens for classic PII fields will miss real categories of personal data GDPR still regulates, which is a genuine compliance gap, not a pedantic distinction. In marketing work, personal data shows up constantly and often without looking like a compliance problem.

**Directly identifying** data includes a full name, email address, phone number, physical address, a customer ID tied to a real person, or a payment card number. This is the category classic PII definitions cover well.

**Identifying in combination** covers data that only becomes personal once fields are joined: a ZIP code plus a birth date plus a gender can narrow down to one person even with no name attached, which is why "anonymized" data can quietly become identifying again once several fields are combined.

**Online and behavioral identifiers** are the category a PII-only lens tends to miss entirely. GDPR Article 4(1) names "an online identifier" directly as grounds for identifiability. An IP address, a device ID, a cookie ID, an advertising ID, or a browsing or purchase history tied to any of those, none of them look like a name, and none of them identify someone the way an email address does. Under GDPR they're personal data anyway, because they relate to an identifiable person and, in most marketing stacks, can be linked back to one.

**Special category data**, GDPR's term for information treated more strictly than ordinary personal data, covers health information, race or ethnicity, religious belief, sexual orientation, and political opinion. A "why did you cancel" survey response that mentions a medical condition falls into this category the moment it's collected.

The synthetic dataset used elsewhere in this course, Session 2's five-category feedback set, has none of this by design; it's invented text with no real person behind it. The moment a real CRM export, a support ticket log, an ad-platform export, or a survey response file is swapped in, personal data is very likely inside it, and "PII" still shows up through the rest of this lesson as a familiar shorthand for the directly-identifying fields specifically. Treat it as one visible slice of the broader personal-data category GDPR actually regulates, not as a synonym for it.

---

## Why this matters in an AI pipeline

An AI API call is a request to a third party. Anything pasted into a prompt, or included in a dataframe passed to an API, leaves the sender's machine and reaches the provider's servers, the same way an email leaves an inbox the moment it's sent.

Session 1 covered that data-use policy depends on whether a key is billed under a "Paid tier" project: Google's terms say Paid tier Gemini usage isn't used for training and skips human review, but that protection is about the model *learning from* the data, not about whether the data was *sent* at all. A personal-data exposure isn't only "did the model train on this," it's also "did this personal data leave a system it was never supposed to leave," which can matter to a client contract or a privacy law regardless of what the AI provider does with it afterward.

Session 3's API key guide covered one version of this already: pasting a real key into a debugging chat. The same habit, copying something real "just to test" or "just to debug," is exactly how personal data gets into a prompt too, and it's usually not the planned, obvious use case (like a listed "process customer feedback" prompt) that leaks it. It's the unplanned one: a quick support-ticket summary, a "can you fix this error" paste that includes an API response with real customer fields, a CRM export opened to check formatting before the real anonymization pass.

---

## Guidelines: how not to expose it

1. **Default to synthetic or anonymized data for testing and coursework.** This is the course's own rule, and it's the right default for any prompt development: get the prompt working on invented data first, and only point it at real data once the prompt itself is proven.
2. **Redact before pasting.** The same habit as Session 3's key guide: before pasting a support ticket, a CRM row, or an error trace into a prompt or an AI chat, scan it for names, emails, phone numbers, and addresses, and for identifiers that don't look like classic PII but still count as personal data, an account ID, a device or cookie ID, an IP address, and replace them with placeholders.
3. **Aggregate instead of pasting raw rows, whenever the task allows it.** "What's the average time-to-resolution for the shipping-delay category" doesn't need a single row of real customer data. A pre-aggregated summary answers it just as well and never touches personal data in the first place.
4. **Strip identifying columns programmatically, not by eye.** Scanning a spreadsheet visually for anything that looks personal misses things, and it misses online identifiers especially, since a device ID or a cookie value doesn't read as "personal" the way a name does. Drop or mask the columns by name in code before the dataframe reaches an API call. The course notebook demonstrates the pattern.
5. **Know the data-use policy before a real run, not after.** Same principle as Session 1: check whether the key in use is on a "Paid tier" project, and confirm with the organization whether real customer data is permitted through this API at all. That's a policy decision, not a technical one, and it should be made before the first real request, not discovered afterward.
6. **Treat special category data as a hard stop, not a redaction problem.** Health, religious, or similarly sensitive mentions inside a message aren't safely handled by swapping out a name. If a dataset is expected to contain this kind of content regularly, that's a conversation with whoever owns data protection at the organization, not a prompt-engineering problem.

---

## A simple personal-data redactor

The same shape-matching idea from the API key guide applies here: a regular expression can catch anything that's structured, an email address, a phone number, because those follow a predictable pattern. It can't reliably catch a name in free text, because names don't have a fixed shape the way an email address does, and it won't catch an online identifier like a cookie ID unless that identifier has a predictable pattern too, a real limitation worth knowing rather than trusting the tool past what it can do. The course notebook builds this as a small function, `redact_pii()`, run over a handful of invented example messages to make the idea concrete; the name reflects what the function actually catches, the classic directly-identifying fields, not the full personal-data surface this lesson covers.

---

## Scrubbing a dataframe before it reaches an API call

Redacting free text catches some personal data. The more reliable pattern, when data is already in a table, is to decide which columns are identifying, directly, in combination, or as an online identifier, and drop or mask them before anything is sent, rather than trying to catch every mention after the fact. The course notebook works through this on an invented customer dataframe: a `PII_COLUMNS` list naming the identifying fields explicitly, then `.drop(columns=PII_COLUMNS)` before the data goes anywhere near an API call.

---

## If personal data gets exposed anyway

1. **Stop the pipeline, don't just fix the prompt.** If a run just sent real personal data somewhere it shouldn't have, pause the loop or job before it processes more rows. A fix applied mid-run still lets the remaining rows through on the old behavior.
2. **Check what actually left the building.** Which rows, which fields, sent to which provider, under which account. This is the fact-finding step before anyone decides what else needs to happen.
3. **Follow the organization's incident process, if one exists.** Most companies handling customer data have a data protection lead, a DPO, or a documented breach process for exactly this. If this is coursework rather than a live client engagement, there's no such process to follow, but building the habit of checking for one is the point.
4. **Don't guess at legal exposure.** Whether a given exposure triggers a notification requirement under GDPR, CCPA, or another law is genuinely fact-specific and outside what this guide, or a prompt-engineering habit, can answer. That's what the disclaimer at the top means in practice: a real exposure is a legal and organizational question, not just a technical cleanup.
5. **Fix the process gap, not just the instance.** If real personal data made it into a prompt once, the guideline that would have caught it (redact first, aggregate first, drop the columns in code) is the thing worth reinforcing, not just deleting the one bad message.

---

## Wrap-up

Personal data shows up in marketing data far more often than it looks like a compliance problem: a name in a support ticket, an email in a CRM export, a cookie ID riding along in an ad-platform export, a health mention in a churn survey. GDPR's definition covers all of it, not just the directly-identifying fields "PII" usually brings to mind. The guidelines here are the same shape as the API key guide before it: default to synthetic or aggregated data, redact or strip before anything real reaches a prompt, and know the data-use policy before the first real request rather than after.

**Next in Session 3**: identifying risk points across a full AI pipeline and applying data-loss-prevention (DLP) principles end to end, not just at the prompt.

Questions belong in the Circle community.
