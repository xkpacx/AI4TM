# What is the EU AI Act, and why you should care about it

> **Disclaimer.** We are practitioners who build AI systems, not lawyers, data protection officers, or regulatory counsel. What follows is our reading of the law as it stands on 27 July 2026 and does not constitute legal advice. Every substantive claim below links to a primary source. Where a decision carries commercial or legal exposure, take it to your legal counsel and your DPO.

---

## Introduction

This week we learn technical controls: keeping secrets out of a repository, keeping personal data out of an API call, deciding where a model runs.

A control only makes sense once you know what it protects against, and who answers for it when it fails.

Every pipeline in this course, including the AI assistant and the topic classification use case from earlier sessions, raises three questions worth having in mind from this point on, every time you build with AI:

1. What role am I playing?
2. What risk category does this fall into?
3. What do I owe the people on the other end of it?

---

## The state of the law as of this week (27 July 2026)

The AI Act, Regulation (EU) 2024/1689, entered into force on 1 August 2024 and applies in stages.

On 24 July 2026 the Official Journal published Regulation (EU) 2026/1744, the Digital Omnibus on AI, which amended it in several places that matter for this lesson:

- rewrote the application timetable in Article 113
- softened the AI literacy duty in Article 4
- inserted a new Article 4a on processing special-category data for bias detection
- added two prohibitions to Article 5

It also expanded the AI Office's enforcement powers and adjusted obligations for SMEs, which this lesson does not cover. That amending regulation entered into force on 27 July 2026.

> Commentary written before late July 2026, including much of what currently ranks well in search results, describes either the original timetable or a negotiating text that changed before adoption. The publication date on anything you read about this determines whether its dates are still correct.

---

## What counts as an AI system

The scope of the whole regulation rests on one definition. Under Article 3(1), an AI system is a machine-based system designed to operate with varying levels of autonomy, that may exhibit adaptiveness after deployment, and that, for explicit or implicit objectives, infers from the input it receives how to generate outputs such as predictions, content, recommendations, or decisions capable of influencing physical or virtual environments.

Inference sets the boundary. A rule you wrote by hand, where every branch is specified in advance, falls outside the definition, while a model that derives its behaviour from data falls inside it, whether that model is a logistic regression scoring leads or a large language model writing product descriptions. The regulation does not turn on how sophisticated the model is; it turns on whether the system infers, and on what that inference gets used for.

---

## The risk categories

Obligations scale with potential harm, which means your compliance burden follows from the purpose a system serves rather than from the model you happened to call.

### Prohibited practices

Prohibited practices under Article 5 have been unlawful since 2 February 2025, and several of them sit close to ordinary marketing technology.

The list covers manipulative or deceptive techniques that materially distort behaviour and cause significant harm, exploitation of vulnerabilities arising from age, disability, or a specific social or economic situation, social scoring, untargeted scraping of facial images from the internet or CCTV to build facial recognition databases, emotion inference in workplaces and educational institutions, and biometric categorisation used to deduce race, political opinions, trade union membership, religious or philosophical beliefs, sex life, or sexual orientation. A personalisation strategy that depends on inferring something a person never told you and would not want inferred belongs in this section.

> **Good to know.** The facial image scraping prohibition describes the practice Clearview AI became known for after a 2020 New York Times investigation. Between 2022 and 2024 the French CNIL, the Italian Garante, the Greek Hellenic DPA, and the Dutch DPA each found the practice unlawful and imposed fines, totalling roughly €100 million, with the Dutch fine alone set at €30.5 million. Those actions ran under GDPR rather than the EU AI Act, since they predate it, so data protection law already reached this conduct before a dedicated AI prohibition existed. Clearview contested jurisdiction on the basis that it had no establishment or customers in the EU, and has largely not paid, which separates a fine on paper from a fine collected.

Regulation 2026/1744 added two further prohibitions: on AI systems generating or manipulating realistic intimate imagery of identifiable people without their explicit consent, and on the generation of child sexual abuse material. These apply from 2 December 2026, rather than from this week.

### High-risk systems

High-risk systems, under Article 6 and Annex III, cover biometrics, critical infrastructure, education, employment and worker management, access to essential private and public services including creditworthiness assessment and insurance pricing, law enforcement, migration and border control, and the administration of justice and democratic processes.

Most marketing work sits outside these categories, and it stops sitting outside at the point where an output starts determining access rather than describing a preference. A propensity model ranking an audience for a newsletter stays outside; the same model deciding who gets offered credit terms moves inside.

### Transparency obligations

Transparency obligations under Article 50 attach to systems that interact with people, that generate synthetic content, or that perform emotion recognition and biometric categorisation, whatever risk category they otherwise fall into. Most marketing work lives here, and this is the part we'll discuss in the upcoming weeks.

Systems outside all of these carry no new direct obligations beyond the general ones.

---

## Provider or deployer, and where the boundary actually sits

Under Article 3(3), a provider develops an AI system, or has one developed, and places it on the market or puts it into service under its own name or trademark, whether for payment or free of charge. Under Article 3(4), a deployer uses an AI system under its own authority in the course of its activity.

There's a widely repeated claim that putting your name on any AI system converts you from deployer to provider. The rule that says so, Article 25, is narrower than the claim, because it addresses the re-attribution of provider status for high-risk systems specifically, covering three situations:

1. putting your name or trademark on a high-risk system already on the market
2. making a substantial modification to one
3. modifying the intended purpose of a system so that it becomes high-risk

Article 25 also sits in Chapter III, Section 3, which means its application is deferred along with the rest of the high-risk regime.

That said, the general definition in Article 3(3) does its own work and is not deferred. If you commission or build a system and put it into service under your own name, you are a provider of that system, and the Article 50 provider duties attach to you rather than to the vendor whose model you called.

Where that line falls when you build on someone else's model is one of the genuinely contested questions in this area. The Commission's Article 50 guidelines, Communication C(2026) 5054 final of 20 July 2026, address which actors are covered. They are non-binding, and the Court of Justice remains the ultimate interpreter, which makes them the best available reading rather than settled law.

Calling someone else's API does not transfer responsibility to whoever owns the API. The model provider carries general-purpose AI model obligations for the model. Responsibility for the system you assembled, and for the purpose you assembled it to serve, is a separate question with a separate answer.

---

## The timeline as it stands

The deferral reaches only Sections 1, 2, and 3 of Chapter III, meaning the requirements for high-risk systems, the operator obligations attached to them, and the rules on notified bodies. Everything else keeps its original date. The new deadlines are fixed rather than conditional: the mechanism the Commission originally proposed, which would have tied application to the availability of harmonised standards, was dropped during negotiation.

> **Good to know.** Article 4 on AI literacy was rewritten. Providers and deployers must now take measures to support the development of AI literacy among their staff and others operating systems on their behalf, and the amended text states expressly that this does not require them to guarantee any specific level of AI literacy in any individual. The Commission and member states are committed to supporting that effort. Article 4 has never carried a standalone penalty, so the softening affects documentary burden more than sanctioning exposure.

---

## What happens (maybe) on 2 August 2026

Article 50 imposes four sets of duties, each attaching to work a marketing team does regularly.

1. Providers must design systems intended to interact directly with people so that those people are informed they're dealing with an AI system, unless that would be obvious to a reasonably well-informed and observant person.
2. Providers of systems generating synthetic audio, image, video, or text must ensure the outputs are marked in a machine-readable format and detectable as artificially generated or manipulated.
3. Deployers of emotion recognition or biometric categorisation systems must inform the people exposed to them.
4. Deployers publishing deepfakes must disclose them as artificially generated, and deployers publishing AI-generated or AI-manipulated text to inform the public on matters of public interest must disclose that too — unless the publication went through human review and someone holds editorial responsibility for it.

> **Important to know.** The Commission published the final Code of Practice on Transparency of AI-Generated Content on 10 June 2026, structured in a section for providers covering marking and detection and a section for deployers covering labelling, and the Commission and the AI Board have since confirmed it as an adequate voluntary tool for demonstrating compliance. The Article 50 guidelines followed on 20 July 2026. The EU has also published a set of icons that deployers may use when labelling AI-generated content.
>
> The status of the code is narrow. Signing is voluntary, and signatories can point to its measures to demonstrate compliance, which gives them predictability across member states. Anyone complying by other means has to show those means are adequate, assessed individually by national market surveillance authorities. The code does not create a formal presumption of conformity, and the amending regulation removed the Commission's power to approve it by implementing act for that reason.
>
> Under the guidelines, content generated before 2 August 2026 does not require retroactive marking, although text generated before that date and published on or after it does require labelling.

On consequences: Article 99(4)(g) places non-compliance with Article 50 in the band of administrative fines up to €15 million or 3% of total worldwide annual turnover for the preceding financial year, whichever is higher. That's the middle tier. Breaches of the Article 5 prohibitions sit higher, at €35 million or 7%, and supplying incorrect or misleading information to authorities sits lower, at €7.5 million or 1%. For SMEs and start-ups, Article 99(6) inverts the calculation so the fine is capped at the lower of the percentage or the fixed amount. For a company with €2 million in turnover, that changes the ceiling on a transparency breach from €15 million to €60,000.

---

## Where the EU AI Act meets GDPR, and where it does not

The two regulations answer different questions. GDPR governs personal data: the lawful basis for processing it, the minimisation of what you collect, and the rights of the person the data describes. The EU AI Act governs the system: its safety, its documentation, and its transparency, whether or not personal data is involved. When an AI system processes personal data, both apply simultaneously, and a prohibited practice involving personal data will generally breach both.

The practical consequence, which is the bridge into this week's content, is that an employee pasting a customer segment into a public chat interface creates a GDPR problem sitting with your organisation as controller, and the EU AI Act has very little to say about it. Governance, tooling, and data loss prevention are what address that risk.

Regulation 2026/1744 also inserted a new Article 4a, providing a route for processing special categories of personal data where that's strictly necessary for detecting and correcting bias, subject to six cumulative conditions including the impossibility of achieving the purpose with synthetic or anonymised data, strict access controls, and deletion once the bias is corrected. Recital 9 ties it to Article 9(2)(g) GDPR, which makes it a specification of substantial public interest rather than a free-standing legal basis, and it creates no obligation to perform bias detection at all. This connects to Session 4: synthetic data is one of the alternatives Article 4a expects you to have ruled out before relying on it.

One caveat remains live. The parallel data omnibus, COM(2025) 837, which would amend GDPR and contains the much-discussed legitimate interest basis for AI training, is still a proposal under negotiation, with agreement anticipated during 2027. Until it's adopted and published, GDPR applies exactly as written today. The AI omnibus took eight months to travel from proposal to Official Journal; the data omnibus has been in negotiation for twenty. Anything that lives only in a proposal has no force yet, whatever a news headline implies.

---

## Some general misunderstandings to keep in mind

The headline most people took from this summer is that the EU delayed the AI Act. Sixteen months of relief went to the high-risk obligations, which most marketing systems never trigger, while nothing moved for transparency, which is the surface marketing work touches daily.

Where the field genuinely disagrees is on whether the delay was necessary or corrosive.

The Council's position is that the compliance infrastructure — harmonised standards, designated national authorities, conformity assessment tools — had not arrived, and that enforcement without it would have been unworkable. Industry associations including DIGITALEUROPE broadly supported the direction while pressing for further alignment across overlapping digital rules. Against that, the Centre for Democracy and Technology argues that the package represents a step back for fundamental rights, and Liberties characterised the postponement as a delay to protections that were due to take effect in August 2026.

A third reading complicates the simplification framing on its own terms. Alongside the deferrals, the same regulation created two new absolute prohibitions, a new legal basis for processing sensitive data, and a substantially expanded set of investigative and sanctioning powers for the AI Office, including inspection powers and periodic penalty payments of up to 5% of average daily worldwide turnover. The burden lightened for operators on documentation while it grew at Union level on enforcement, which reads as redistribution rather than reduction.

---

## Before 2 August

Inventory every AI system you run or build, including the ones nobody in the building calls AI.

For each one, work out the role you occupy and the purpose the system serves. Find every place where a generative system produces something a person will read, see, or hear, and decide how disclosure and marking get handled there. Check which of your vendors have signed the code of practice, since their marking behaviour is what you inherit. Then look at whether any output determines access to something, because that's the line that moves a system into a category with an entirely different set of obligations, on a clock that now runs to December 2027.

---

## Sources

- Regulation (EU) 2024/1689, the AI Act
- Regulation (EU) 2026/1744, the Digital Omnibus on AI, OJ 24 July 2026
- Council press release on final adoption, 29 June 2026
- Commission guidelines on Article 50 transparency obligations
- Code of Practice on Transparency of AI-Generated Content
- EU icons for labelling AI-generated content
- Commission overview of the AI Act regulatory framework
- European Parliament legislative train, digital omnibus package status
- Centre for Democracy and Technology on the final omnibus text
- Dutch DPA decision on Clearview AI
