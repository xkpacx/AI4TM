# AI4TM - AI for Marketing Masterclass

Content, notebooks, and guides for the AI for Marketing (AI4TM) masterclass — 6 sessions covering AI fundamentals, evaluation, compliance, synthetic data, knowledge graphs, and agentic workflows.

## 🚀 Getting Started (Low-Cost, No Local Install)

This course uses **Google Gemini's pay-as-you-go API** — as of 2026, Google requires a billing method on file before it will issue a key at all, so there's no card-free option anymore. Usage on the lightweight models this course uses runs at low, pay-as-you-go rates (fractions of a cent per request is typical, but confirm current pricing before you start — Google changes it).

### Quick Start (10 minutes)

1. **Fork this repository**:
   Click "Fork" at the top right of this page. This gives you your own copy at `github.com/YOUR_USERNAME/AI4TM` to save your work into as you go, without touching the original.

2. **Get a Gemini API key**:
   Go to [aistudio.google.com/apikey](https://aistudio.google.com/apikey), enable billing when prompted, and create a key

3. **Open notebooks in Google Colab**:
   Click any `.ipynb` file in this repository → Look for "Open in Colab" badge → Click it

4. **Start with the setup guide**:
   [week_2/lesson08_setup_guide.ipynb](week_2/lesson08_setup_guide.ipynb) — it walks through all of the above in order, plus how to save your work into your fork

**That's it!** No local installation, no complex setup — just a Google account with billing enabled.

### What You Get

✅ Google Colab - Run code in your browser, no installation
✅ Gemini API - low pay-as-you-go cost on the lightweight models this course uses (billing must be enabled)
✅ GitHub - Save and share your work
✅ Free GPUs in Colab (a separate Colab feature — unaffected by the Gemini API billing change above)

### ⚠️ Privacy Note

Unpaid/free-quota AI usage (Google, OpenAI, etc.) may be reviewed by humans or used to improve models. Google's terms say billed ("Paid tier") Gemini usage is not used for training and skips human review for that purpose — check the "Paid tier" badge in AI Studio to confirm which applies to your key, don't assume. Either way: **only use public, synthetic, or anonymized data in this course.** Never send confidential or client data.

## 📚 Course Structure

### Session 1: Essential Concepts ✅ Setup Complete
**Topics**: Neural networks, attention, tokenization, embeddings
**Approach**: Production failure modes (light theory)
**Content**:
- [Setup Guide (Notebook)](week_2/lesson08_setup_guide.ipynb) - Colab + Gemini API + GitHub
- [Setup Guide (Text)](week_2/lesson08_setup_guide_circle.md) - For Circle platform
- [Token Cost Guide (Notebook)](week_2/lesson09_token_cost_guide.ipynb) / [Text](week_2/lesson09_token_cost_guide_circle.md) - Estimate cost before running a batch job

### Session 2: Evaluation
**Topics**: Regression, classification, semantic search
**Content**:
- [Evaluation Template (Notebook)](week_3/lesson13_evaluation_template.ipynb) / [Text](week_3/lesson13_evaluation_template_circle.md) - Accuracy, precision, recall, F1, confusion matrix
- [Human-in-the-Loop Validation (Notebook)](week_3/lesson12_human_in_the_loop.ipynb) / [Text](week_3/lesson12_human_in_the_loop_circle.md) - BLEU, ROUGE, LLM-as-judge, and deciding where a person needs to check the model's work

### Session 3: Compliance
**Topics**: AI compliance and regulatory considerations
**Content**:
- [What is the EU AI Act, and Why You Should Care (Text)](week_4/lesson14_eu_ai_act_intro_circle.md) - Risk categories, provider vs. deployer, the 2026 Digital Omnibus timeline, GDPR overlap
- [Reviewing Your Own Architecture for Compliance (Text)](week_4/lesson19_architecture_compliance_worksheet_circle.md) - A reusable six-step worksheet applying the AI Act criteria to any system you build
- [Explore a Real-World Use Case (Text)](week_4/lesson15_real_world_use_case_circle.md) - A real enterprise case study (Starbucks EMEA, built at Monks) read against the AI Act criteria
- [Setting Up and Protecting Your API Keys (Notebook)](week_4/lesson18_api_key_security.ipynb) / [Text](week_4/lesson18_api_key_security_circle.md) - Setup and protection guidelines, keeping keys out of GitHub and out of AI chat prompts
- [Protecting PII When You Use AI (Notebook)](week_4/lesson17_pii_protection.ipynb) / [Text](week_4/lesson17_pii_protection_circle.md) - What counts as PII, redacting it before it reaches a prompt, scrubbing a dataframe before an API call
- [Finding Risk Points in an AI Pipeline (Notebook)](week_4/lesson16_pipeline_risk_points.ipynb) / [Text](week_4/lesson16_pipeline_risk_points_circle.md) - Applying DLP principles across a full pipeline, spot-the-vulnerability exercise

### Session 4: Synthetic Data
**Topics**: Generating and using synthetic data for marketing

### Session 5: Knowledge Graphs
**Topics**: Nodes, edges, and Cypher; turning a set of documents into a graph with an LLM and merging the duplicates it produces; what a graph answers that a table struggles with; GraphRAG and checking whether an answer is actually grounded in the graph
**Content**:
- [Setup: your knowledge graph environment (Notebook)](week_6/setup_guide.ipynb) / [Text](week_6/setup_guide_circle.md) - Neo4j AuraDB Free signup, connecting the graph database, and a swappable `LLM_PROVIDER` config so the rest of the week's notebook can switch model providers by changing one value
- [Building and querying a knowledge graph (Notebook)](week_6/knowledge_graph_pipeline.ipynb) / [Text](week_6/knowledge_graph_pipeline_circle.md) - Extracting entities and relationships from a set of internal documents, merging duplicate entities, loading and querying in Cypher, and GraphRAG with a groundedness check

### Session 6: Agentic Workflows
**Topics**: Search intent taxonomies, clustering a search performance export into content gaps, an agent (tools + a loop) that reads that table, generating a landing page brief and evaluating it against the Week 3 template
**Content**:
- [Building an agentic content pipeline (Notebook)](week_7/agentic_content_pipeline.ipynb) / [Text](week_7/agentic_content_pipeline_circle.md) - Rebuilding a compact content-gap table, then a hand-written tool-calling loop (three read-only tools, two visible stopping conditions) that turns it into grounded content recommendations
- [Generating and evaluating a landing page brief (Notebook)](week_7/landing_page_brief_generator.ipynb) / [Text](week_7/landing_page_brief_generator_circle.md) - A guardrailed brief generator for one content gap, scored by reusing Session 2's evaluation template against a mix of LLM-judged and objectively-recomputed criteria

## 🛠️ Tools Used

- **Google Colab** (default) - Cloud-based notebooks, no installation
- **Google Gemini API** (default, low-cost pay-as-you-go, billing required) - AI model access
- **GitHub** - Version control and portfolio

**Already have OpenAI or Anthropic API?** You can use those instead — see the optional section in the setup notebook.

## 📖 For Instructors

- [CLAUDE.md](CLAUDE.md) - Repository guidance for AI assistants
- All notebooks designed for Google Colab (one-click from GitHub)
- Local setup available as optional/advanced path
- Students work in their own fork, never as collaborators on this repository — nobody but the instructor team can push to it, forking or not. `main` is branch-protected (PR review required, no force-push or deletion).

## 🤝 For Students

### Before Each Session
1. Open the session notebook in Colab
2. Make sure your API key is in Secrets (🔑 icon)
3. Run the setup cells

### Tips
- **Save often**: File → Save a copy in GitHub, into your fork
- **Experiment**: Try changing code to see what happens
- **Watch your spend**: no free quota to fall back on — check pricing at [AI Studio](https://aistudio.google.com/apikey) and use the token cost guide before a big batch job
- **No real data**: Only use public or made-up data

## ⚠️ Important Notes

- **Never commit API keys** - use Colab Secrets or `.env` files (local). See [week_4/lesson18_api_key_security.ipynb](week_4/lesson18_api_key_security.ipynb) for how keys leak and how to catch it.
- Billing must be enabled on your Google account before you can get a Gemini API key — this now applies everywhere, not just the EU/UK/Switzerland
- Data-use policy (human review, model training) depends on whether your key is on a "Paid tier" project — check the badge in AI Studio, don't assume

## Optional: Local Setup

For experienced users who prefer local development, see the "Optional B: Local Setup" section in [week_2/lesson08_setup_guide.ipynb](week_2/lesson08_setup_guide.ipynb).

Most learners should use Google Colab (the default path).

## 📧 Questions?

Post in the Circle community or open an issue in this repository.

---

**Ready to learn AI for Marketing? Start here:** [Session 1 Setup Guide](week_2/lesson08_setup_guide.ipynb) 🎯
