# Reviewing your own architecture for compliance

A worksheet, not an essay: six steps you run against any AI system you build or use, so you know what role you're playing, what risk category the system falls into, and what you owe the people on the other end of it. It's the practical follow-through on the EU AI Act primer earlier in this session — read that first if you haven't, since every step below leans on a definition from it.

**Time**: about 20 minutes per system reviewed
**Cost**: $0 — this is a document, not code

> **Disclaimer.** This worksheet is educational guidance, not legal advice, and we are practitioners, not lawyers or regulatory counsel. It helps you ask the right questions about a system; it does not answer them for you with legal certainty. Where a system's classification carries real commercial or legal exposure, take your filled-in worksheet to your legal counsel and your DPO rather than treating this as a final answer.

---

## How to use this

Run one system through all six steps at a time. "System" means anything that infers from data to produce an output someone acts on — a lead-scoring model, a chatbot, a content generator, a classifier, not just something with "AI" in its name. Keep the filled-in worksheet. Re-run it whenever the system's purpose changes, since a modification can move a system into a different risk category without anyone deciding that on purpose.

---

## Step 1: describe the system in plain terms

Before anything else, write down what the system actually does, without the marketing language. A vague description hides the parts that matter.

- What does it produce — a prediction, a piece of generated content, a recommendation, a decision?
- What data goes in, and does any of it identify a real person?
- Who or what triggers it: a person using it directly, or a backend process running on a schedule?
- Who sees or acts on the output?
- Does it infer from data, or does it follow rules you specified by hand? This is the AI Act's own boundary test (Article 3(1)): a system that only executes branches you wrote in advance is outside its scope entirely. A system that derives its behavior from data — however simple the model — is inside it.

If the last answer is "it follows rules I wrote," stop here. The rest of this worksheet doesn't apply.

---

## Step 2: determine your role — provider or deployer

- **Did you build this system, or commission someone to build it, and put it into service under your own name?** If yes, you're a **provider** of it (Article 3(3)) — the obligations that attach to providers are yours, not the vendor's whose model or API you called.
- **Are you using a system someone else built, without commissioning it or rebranding it?** If yes, you're a **deployer** (Article 3(4)) — a narrower set of obligations applies, mostly around informing people and using the system as intended.
- **Did you take a system someone else built, put your name or trademark on it, substantially modify it, or repurpose it for something that makes it high-risk?** These specific moves can re-attribute provider status to you even for a system you didn't build from scratch (Article 25) — but only for high-risk systems, so this question matters most once Step 3 below flags something as high-risk.

Calling an API does not, by itself, transfer the model provider's obligations to you, and it does not transfer your obligations to them either. You're answering for the system you assembled and the purpose it serves; the model vendor answers for the model.

---

## Step 3: classify the risk category

Three checks, run in order. Stop at the first "yes."

**Check A — is it prohibited?** Does the system do any of the following: use manipulative or deceptive techniques that materially distort someone's behavior and cause them significant harm; exploit a vulnerability tied to someone's age, disability, or social or economic situation; assign people a social score; scrape faces from the internet or CCTV to build a facial recognition database; infer emotion in a workplace or school; or use biometric data to infer someone's race, political views, union membership, religion, sex life, or sexual orientation?

*If yes: stop.* This is not something you can put into service in the EU, full stop, regardless of any answer below.

**Check B — is it high-risk?** Does the system's output determine whether someone gets access to something — credit, insurance pricing, a job, an essential public or private service, education, or involvement in law enforcement, migration, or the administration of justice? The line that matters: a system that *describes a preference* (this audience is more likely to buy) stays outside; a system that *determines access* (this applicant gets credit terms or doesn't) moves inside.

*If yes:* the high-risk obligations apply. Some of these are currently deferred rather than eliminated — check the primer's timeline section for the current date — but flag this to legal/DPO now regardless, since the documentation and assessment work involved takes real lead time.

**Check C — does the transparency requirement (Article 50) apply?** Does the system interact directly with a person, generate synthetic audio/image/video/text, or perform emotion recognition or biometric categorization? A huge share of marketing AI work lands here.

*If yes:* go to Step 4.

**None of the above?** No new direct obligations beyond the general ones. Still worth documenting in Step 6, since "we checked and it's out of scope" is itself useful to have on record — and worth re-checking if the system's purpose ever changes.

---

## Step 4: if Article 50 applies, which duties actually bite

Check the block that matches your Step 2 answer. It's common for both to apply, if you're the provider of one part of a system and the deployer of another.

**Provider checklist**
- [ ] Talks to people directly → disclose it's an AI (unless already obvious)
- [ ] Generates synthetic content → mark it as AI-generated

**Deployer checklist**
- [ ] Emotion or biometric recognition → inform the people it's used on
- [ ] Publishing a deepfake or public-interest AI text → disclose it (unless human-reviewed with editorial responsibility)

Nothing checked? No Article 50 duty here — re-check if the system's purpose changes.

---

## Step 5: check the GDPR overlap

The AI Act and GDPR ask different questions and both can apply at once. Run this regardless of what Step 3 found, since a system can be entirely outside the AI Act's risk categories and still be a live GDPR question.

- Does any personal data pass through this system, at input, output, or in what gets logged? If yes, your organization is very likely the **controller** of that data, and answers for everywhere it travels.
- What's the lawful basis for processing that data through this system? "We already had the data" is not, on its own, a basis for a new use of it.
- Where does the data actually go — a hosted API call to a third party, a model deployed inside your own infrastructure, a log file, a downstream tool? (This is exactly what Session 3's key, PII, and pipeline-risk-points guides walk through in more technical detail — use this step as the prompt to go run that checklist for real, not a substitute for it.)
- If a vendor processes personal data on your behalf, do you have an agreement covering that, and do you know their retention and training policy for it?

---

## Step 6: document it

One row per system, kept somewhere the team can find it later. Reuse this shape:

| System | Role (provider/deployer) | Risk category | Article 50 duties triggered | Touches personal data? | Flagged to legal/DPO? | Last reviewed |
|---|---|---|---|---|---|---|
| *(example — see below)* | | | | | | |

A register with one row is still worth starting. It gets more valuable every time you add a system to it, and it's the artifact you actually hand to legal or your DPO when something needs a real answer.

---

## Worked example

**System**: an email subject-line generator that writes three options from a campaign brief, and a marketer picks one before sending.

1. **Describe it**: produces generated text (a set of subject lines); input is a short campaign brief with no personal data; triggered by a marketer on demand; a human picks and sends the output. It infers from data (the model wasn't hand-coded with subject-line rules), so it's in scope.
2. **Role**: the marketing team built this internal tool by calling a third-party model API and put it into service under their own name. That makes them the *provider* of this system — even though they're only a deployer of the underlying model API itself. Easy to get wrong: "we just called an API" feels like deployer, but Step 2's test is about the tool assembled, not the model underneath it.
3. **Risk category**: not manipulative or deceptive by design, doesn't determine access to anything, so it clears Checks A and B. It generates synthetic text — Check C is a yes.
4. **Article 50 duties**: as provider, the synthetic-content marking duty is worth taking seriously — even internal-use text can end up on a customer-facing surface unmarked. The interaction-disclosure duty doesn't apply here, since the tool talks to the marketer using it, not to an end customer.
5. **GDPR overlap**: no personal data in the brief as described; if a future version pulled in real customer names for personalization, this answer changes and the worksheet should be re-run.
6. **Documented**: one row added to the register, marked reviewed today, flagged to legal for a second opinion on whether the marking duty actually reaches subject lines a human reviews before every send.

---

## Wrap-up

Six steps, run consistently, turn "is this compliant?" from a vague worry into an answer you can actually point to. None of the individual checks are hard; the value is in running all six on every system, including the ones that don't feel like "AI" and the ones nobody remembers to check twice.

**Also in Session 3**: the EU AI Act primer this worksheet operationalizes, a real-world case study applying the same criteria to an enterprise system, and the API key, PII, and pipeline risk-point guides referenced in Step 5.

Questions belong in the Circle community.
