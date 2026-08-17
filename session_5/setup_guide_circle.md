# Setup: your knowledge graph environment

This is the text companion to the setup notebook. **Time**: about 15 minutes. **Cost**: effectively $0 to set up — the graph database has a permanently free tier, and the test calls in the notebook use a handful of tokens on Gemini's cheapest model.

This week adds one more moving part to the pipeline built up since Session 1. Every notebook so far has talked to one thing: an AI model, over an API. This week also talks to a **graph database** — a separate system that stores nodes (entities) and the typed relationships between them, and answers questions by traversing those relationships rather than by scanning rows. That means two accounts instead of one, and two connections worth testing before touching any real content.

---

## What Neo4j is, and why Aura rather than installing something

**Neo4j** is a graph database, software built specifically to store nodes and relationships and to answer traversal queries efficiently, the way a spreadsheet is built specifically for rows and formulas. **Cypher** is Neo4j's query language, the graph equivalent of SQL — this week's overview linked its [getting-started guide](https://neo4j.com/docs/getting-started/cypher/), and the course notebook writes real Cypher against a real instance.

Neo4j runs two ways: installed locally (Neo4j Desktop, a program on a personal machine), or hosted (**Aura**, Neo4j's managed cloud service). This course defaults to Colab specifically to avoid local installation, and Aura is the same default applied to the database. **AuraDB Free** is a permanently free, hosted instance, no credit card required, created with a Google or GitHub login in about a minute, per [Neo4j's own documentation](https://neo4j.com/docs/aura/). Anyone already running Neo4j Desktop locally can point the same notebook at that instead — the connection code works the same way against a local address, and the only thing that changes is which URI goes into Secrets. This guide assumes Aura, since it needs nothing installed and matches the course's Colab-first default throughout.

> **What the free tier actually gives, and where Neo4j's own documentation disagrees with itself.** One official source states 50,000 nodes and 175,000 relationships as the free-tier ceiling; another states 200,000 nodes and 400,000 relationships. Either number covers this week's exercise many times over — the course graph holds well under 100 nodes — so the discrepancy doesn't affect coursework, but it's a reason not to quote a specific number to a stakeholder without checking the current limit in the Aura console first. Both sources agree on one point worth remembering: a Free instance **auto-pauses after 72 hours of inactivity** (data is retained; opening the console resumes it in under a minute) and is **deleted if left paused for 90 days**. A connection that worked yesterday and fails today with no code changes is very likely a paused instance, not a bug.

**Steps to create an instance:**

1. Go to [console.neo4j.io](https://console.neo4j.io) and sign in — Google, GitHub, or email, no card required.
2. Click **New instance**, choose **AuraDB Free**.
3. Name it (for example, `ai4tm-session5`) and pick a nearby region.
4. Click **Create**. Neo4j shows a connection URI, username, and a generated password exactly once — download the credentials file or copy all three immediately. The password isn't stored anywhere to look up later; losing it means resetting it from the console.
5. Wait for the instance status to show **Running**, usually under a minute.

---

## Storing the credentials

Same pattern as Session 1's API key: Colab **Secrets** (the key icon in the left sidebar), never pasted directly into a cell. Three secrets go in:

- `NEO4J_URI` — starts with `neo4j+s://`
- `NEO4J_USERNAME` — usually `neo4j`
- `NEO4J_PASSWORD` — the generated password from instance creation

The access toggle needs to be on for each, the same as for `GEMINI_API_KEY`. The course notebook includes a check cell that confirms all three load correctly before moving on.

---

## Connecting

`neo4j` is Neo4j's official Python driver, the library that turns Python function calls into Cypher queries sent to the Aura instance — the same role `google-genai` plays for Gemini. The notebook installs it, opens a connection using the three secrets above, and runs a minimal query, `RETURN 1 AS ok`, purely to confirm the connection works before anything else depends on it.

If that step fails, check in order: the instance status is **Running** in the Aura console, not paused or still provisioning; the URI starts with `neo4j+s://` with no typo; and the three secret names match exactly, with their access toggles on.

---

## A swappable LLM connection

Session 1 set up a single Gemini connection. This week's task adds a requirement that matters beyond the exercise itself: the rest of the pipeline should be able to switch which model provider does the extraction and answering by changing **one config value**, not by rewriting the calling code. A real organization building on an LLM pipeline wants that option too, whether the reason is cost, latency, or a model being deprecated out from under them.

The pattern is one variable, `LLM_PROVIDER`, and one function, `call_llm(prompt)`, that branches on it. Every other cell in the pipeline notebook calls `call_llm()` and never names Gemini, OpenAI, or Anthropic directly — changing the single `LLM_PROVIDER` line is what changes which one actually runs. Anyone using `openai` or `anthropic` in place of Gemini adds `OPENAI_API_KEY` or `ANTHROPIC_API_KEY` to Secrets the same way `GEMINI_API_KEY` went in — these remain the optional paid alternatives for anyone who already holds a key with one of those providers, the same framing as every other session. Model names change over time across every provider; an error that names a specific model rather than a connection problem means checking that provider's current model list rather than assuming the notebook is wrong.

---

## What's next

Both connections working — a graph database ready to hold the knowledge graph, and a swappable LLM call for extraction and querying — is everything the pipeline notebook, [`knowledge_graph_pipeline.ipynb`](knowledge_graph_pipeline.ipynb), assumes going in.

**Troubleshooting reference:**

| Symptom | Likely cause |
|---|---|
| `NEO4J_URI not found` | Secret name typo, or access toggle off |
| Connection hangs or times out | Instance paused (open the Aura console to resume it) or still provisioning |
| `Unauthorized` from Neo4j | Password copied incorrectly at creation — reset it from the console |
| `call_llm` errors mentioning a model name | That provider renamed or retired the model; check their current model list |
| Everything worked yesterday, fails today with no code changes | Aura Free auto-pauses after 72 hours idle — check the console status first |
