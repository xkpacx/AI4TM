# Reviewing your own architecture for compliance

> **Disclaimer.** We are practitioners who build AI systems, not lawyers, data protection officers, or regulatory counsel. What follows is our reading of the law and does not constitute legal advice. Where a decision carries commercial or legal exposure, take it to your legal counsel and your DPO.

---

## Introduction

The EU AI Act primer (Lesson 1) gave the criteria: what a system is allowed to do, who carries responsibility, and where the AI Act meets GDPR.

This lesson takes one enterprise system — built at Monks for Starbucks across EMEA — and reads it against those criteria.

The material comes from our own work. What appears here is limited to the architecture, the goals, and the reasoning. The performance numbers are confidential and are not reproduced as part of this lesson.

The public write-up is on the Monks site, if you want the client-facing version.

---

## The challenge

Starbucks had invested in its loyalty app across Europe and the Middle East: new features, a unified experience across more than 30 markets. The question behind the investment was whether any of it worked. Were people using the new features, and did the work translate into customers who were more loyal and more satisfied?

The raw material for answering that was the reviews stream itself, from Android and iOS, which was large and messy. Hundreds of thousands of written reviews arrive across the app stores and other channels, in many languages, at a rate no human team can read. The obvious shortcut is to trust the star rating attached to each review and skip the text. That shortcut fails, for reasons the next section works through.

---

## Why the star rating is not the signal

A star rating and the sentiment of the review text disagree more often than most teams expect. A manual examination of 8,600 Android app reviews found that around 20% carried a rating that contradicted the sentiment of the text, and later work across several app datasets reports the same order of magnitude. People leave five stars and then describe a crash, or one star with "OMG I love this app," because they meant to praise it and misfired on the control.

The Starbucks reviews showed exactly this. A one-star review reading "OMG! I love this app! I specifically like how easy it is to order!" and a five-star review reading "This app is not what I was expecting. I am having issues using it. Do not recommend." are both, on their star rating alone, filed under the wrong heading.

Plotting sentiment against star rating across the corpus, the boxes overlap heavily: strongly positive text appears inside one-star ratings, strongly negative text inside five-star ratings. The star rating is a noisy proxy for how the customer feels, and building an optimisation roadmap on a noisy proxy sends the roadmap to the wrong problems.

That leaves the review text as the real source of truth, which returns the original difficulty. The text is where the signal is, and the text is what no one can read at volume. The system exists to read it.

> **Good to know.** The displayed star rating is already a moving target. Neither store shows a plain lifetime average, which is a second reason the number is a weak signal.
>
> On the App Store, the summary rating is calculated per territory and shown globally, and Apple lets a developer reset it entirely when releasing a new version, which wipes the visible average while leaving the written reviews in place. Recent ratings also carry more weight than old ones.
>
> On Google Play, the rating is a recency-weighted average with no reset option: newer ratings move the number more than older ones, so a bad release can pull the score down quickly and a fix can pull it back up, and the full history stays on record.
>
> The practical consequence for this project is that the same underlying reviews can produce different headline numbers depending on the store, the territory, whether a reset has happened, and how recently each rating landed. A dashboard number built on that is not a stable measure of how customers feel, which is one more reason the analysis works from the review text rather than the star ratings.

---

## From text to something you can act on

A business question sits behind all of this: is the product getting better for the people who use it? You cannot answer that from raw text. Text is not a variable you can average, segment, or track over time, and a chart needs variables.

The work of the system is the conversion: turning hundreds of thousands of free-form reviews into structured fields you can aggregate and follow. Normally that's done in three layers. Each layer is built on the one under it, and the sequence is the same descriptive-then-diagnostic-then-predictive progression that analytics maturity models have described for years, applied here to unstructured customer voice.

**Understand — start with a measure: how does the customer feel?** Before you can improve anything you need a dependent variable, a single readout that stands in for customer sentiment and moves when the experience changes. That's sentiment analysis, and it answers *what* is happening. On its own it's a thermometer. It tells you there's a fever without saying where the infection is, so it's a KPI with no lever attached.

**Classify — add structure: what are they talking about?** A measure gets useful once you can break it down by the things it varies across. Topic classification supplies those dimensions. It turns "sentiment" into "sentiment by topic," which answers *why* it's happening. "Complaints are up" is a number you watch. "Payment complaints are up on Android in this market since the last release" is a number you act on. This layer comes second because a topic label is only worth having once there's a sentiment reading to attach it to.

**Predict — then look ahead: what do they need that they're not getting?** With sentiment broken out by topic and tracked over time, you can turn from describing the present to reading the stream for unmet needs: features people keep asking for, friction that hasn't yet become a review-bombing event, requests that fit no existing label. This is the same classification machinery pointed at a forward-looking question. It depends on the first two layers, because the signal for an unmet need often lives in the reviews that don't classify cleanly into the pain points you already know.

---

## Understand: sentiment, and what a single score hides

We ran sentiment analysis using Google Cloud Natural Language API, a pre-trained service that returns two numbers for a piece of text. The score runs from −1.0 to +1.0 and gives direction — negative through neutral to positive. The magnitude runs from 0 upward and gives the total weight of emotion, regardless of direction. (Because of the variance in the sentiment scores, we developed bespoke sentiment intervals and magnitudes for this project, outside the classic "negative, positive, neutral.")

A review scoring near zero can be flat — "it's fine, nothing special" — or genuinely mixed: love for the rewards followed by fury at a broken payment step, where the two halves cancel to a middling average. Score alone can't tell those apart. Magnitude can: the flat review has low magnitude, the mixed review has high magnitude. A pipeline reading score alone would file the furious mixed review next to the indifferent one, and lose the customer who's halfway out the door.

That's the ceiling on sentiment, and for this goal it's low. Sentiment gives you the temperature of a review. It doesn't tell you what the review is about, it gives a corpus no structure, and it points to no action. Knowing complaints are up this month is not knowing which feature to fix. For that, every review has to be sorted by subject.

---

## Classify: topic, and why it needed its own model

Topic classification turns a heap of temperature readings into a map of the product. Sort each review by what it concerns — payment, login, rewards — and the corpus becomes something a team can prioritise: this many payment complaints this month, trending this way, concentrated in these markets and app versions.

The categories have to be the product's own. A generic classifier sorts text into whatever taxonomy its vendor shipped; this needed the specific failure surface of this app. A first pass over the reviews surfaced the recurring pain points, and those became the label set: reward, payment, login, registration, sign-up, order, scan. No off-the-shelf taxonomy contains that list, which forced a model we could point at its own labels.

The technique that allows it is zero-shot classification. A model pre-trained on natural language inference — trained to judge whether one sentence follows from another — becomes a topic classifier when each label is rewritten as a hypothesis. The review goes in as the premise, and the model scores how strongly "this text is about payment" follows from it, then "this text is about login," across the label set. The model was never trained on those categories and still scores them, which is what zero-shot means: no labelled examples of the target categories are needed. A BERT-family model fine-tuned on the MNLI dataset is a standard choice.

Topic classification does three things sentiment cannot. It sorts feedback into pain points a team can own, so improvement is targeted. It gives the corpus structure, so analysis is prioritisation instead of reading. And it makes recurring problems visible early, while they're still a rising line on a chart.

*PS: check Week 2 content and the classification use case for more on this.*

---

## Predict: unmet needs

Once reviews are scored for sentiment and sorted by topic, the reviews that resist the existing labels become interesting in their own right. A cluster of negative reviews that doesn't fit reward, payment, login, or any known pain point is often a request for something the app doesn't yet do.

Technically, this is the same zero-shot classifier from the previous layer, pointed at a different label set. Instead of scoring reviews against known failure surfaces, it scores them against candidate unmet needs, and the reviews that score low on every existing pain point get surfaced for a human to read. The technique doesn't change. What changes is the question: not "which known problem is this," but "is this something we haven't named yet."

This layer only works because we did sentiment analysis and topic classification first. Without sentiment you can't tell a feature request buried in a happy review from one buried in an angry one, and the angry ones move first. Without topic structure you have no way to see that a group of reviews sits outside every category you track. The prediction rides entirely on the quality of the measure and the structure, which is why the order matters and why the first two layers carry most of the weight.

---

## The architecture, end to end

Reviews are ingested from the app stores and other channels into cloud storage, land in BigQuery as the structured store, and split into the two analysis paths. Sentiment goes out to the hosted NLP API and returns as score and magnitude. Topic classification runs on the deployed model inside the tenant, scoring each review against the custom labels. The two results rejoin, so every review carries both a temperature and a subject, and that feeds reporting, where teams read it as evidence for what to fix and what to build.

---

## What came out of it

The output was not a sentiment number. It was a structured, queryable read on the customer voice: the main issues across markets and app versions, distinct pictures for iOS and Android users, the recurring problems paired with the fixes they pointed to, and a roadmap that could weigh sustainment work against new features on evidence.

Starbucks could run a real learn-and-decide loop: find an issue in the reviews, form a hypothesis, ship a change, and read the next wave to see whether sentiment and topic moved the intended way. The customer voice became a measurement instrument instead of a wall of text nobody had time to read.

---

## What about privacy?

The reviews are customer text, and customer text can carry personal data: a name in a signature, a location, an account detail typed into the box. Under GDPR, Starbucks is the controller of that data and answers for everywhere it travels.

The hosted sentiment call leaves the tenant, but not into the open. Per Google's data usage terms, the API processes text in memory, does not store customer content, and does not train on it. That is a controlled processor relationship under an enterprise agreement, a different thing from pasting the same text into a public chat interface, which is the exposure Lesson 1 flagged and Krasimir's PII lesson takes up. A hosted API on no-retention terms is a legitimate choice.

The topic model sits one step further in. It runs inside Starbucks' own tenant, so the review text never leaves infrastructure Starbucks controls. No external processor to contract, no data egress to account for, no third-party retention terms to read: the data and the model share one governed environment. For a business across more than thirty markets under strict internal rules on privacy and data handling, keeping the step that touches the full corpus in-tenant cuts the external relationships to govern and the borders the data crosses.

The custom label set forced a deployed model, and a deployed model sits in the tenant. The requirement and the privacy benefit reinforce each other.

---

## What "privacy by design" names here

Privacy by design, in GDPR's own language (Article 25), is the duty to build data protection into a system from the start through technical and organisational measures, rather than bolt it on once the system runs. Here it took the form of concrete decisions: put each model where its data allows, keep the step that touches the full corpus inside the controlled environment, and contract the one external call on no-retention terms. These were settled at architecture time, before a single review flowed through, not patched in later.

The anti-pattern is the same capability without that discipline: the employee who pastes a customer segment into a public tool for a quick summary. Text in, analysis out, and a completely different governance outcome, because no one decided in advance where the data was allowed to go.

---

## The wider point

The case sits on a larger shift. Natural language processing changed what a company can do with the unstructured things its customers say. Feedback that used to be too voluminous to read, or readable only through a sampling exercise that threw most of it away, becomes a dataset you can measure, segment, and track. For a customer-facing business, that reaches the core of how it understands the people it serves.

The tool you reach for is set by the task and the data together, not by which is easiest to call: a solved problem on safe terms goes to a hosted model, a task needing your own labels and tighter data control goes to a model you deploy. Where a component runs is a privacy decision before it's a performance one. And the controller stays responsible at every step, in the hosted call and in the tenant alike.

---

## Sources

- Monks case study, AI Customer Voice Analysis
- Google Cloud Natural Language API, sentiment basics
- Google Cloud Natural Language API, data usage FAQ
- "Fault in your stars," analysis of 8,600 Android reviews
- Article 25, GDPR, data protection by design and by default
