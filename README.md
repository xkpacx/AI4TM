# AI4TM - AI for Marketing Masterclass

Content, notebooks, and guides for the AI for Marketing (AI4TM) masterclass — 6 sessions covering AI fundamentals, evaluation, compliance, synthetic data, knowledge graphs, and agentic workflows.

## 🚀 Getting Started (Low-Cost, No Local Install)

This course uses **Google Gemini's pay-as-you-go API** — as of 2026, Google requires a billing method on file before it will issue a key at all, so there's no card-free option anymore. Usage on the lightweight models this course uses runs at low, pay-as-you-go rates (fractions of a cent per request is typical, but confirm current pricing before you start — Google changes it).

### Quick Start (10 minutes)

1. **Get a Gemini API key**:
   Go to [aistudio.google.com/apikey](https://aistudio.google.com/apikey), enable billing when prompted, and create a key

2. **Open notebooks in Google Colab**:
   Click any `.ipynb` file in this repository → Look for "Open in Colab" badge → Click it

3. **Start with the setup guide**:
   [session_1/setup_guide.ipynb](session_1/setup_guide.ipynb)

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
- [Setup Guide (Notebook)](session_1/setup_guide.ipynb) - Colab + Gemini API + GitHub
- [Setup Guide (Text)](session_1/AI_Assistant_setup_guide_circle.md) - For Circle platform
- [Token Cost Guide (Notebook)](session_1/token_cost_guide.ipynb) / [Text](session_1/token_cost_guide_circle.md) - Estimate cost before running a batch job

### Session 2: Evaluation
**Topics**: Regression, classification, semantic search
**Content**:
- [Evaluation Template (Notebook)](session_2/evaluation_template.ipynb) / [Text](session_2/evaluation_template_circle.md) - Accuracy, precision, recall, F1, confusion matrix

### Session 3: Compliance
**Topics**: AI compliance and regulatory considerations
**Content**:
- [Setting Up and Protecting Your API Keys (Notebook)](session_3/api_key_security.ipynb) / [Text](session_3/api_key_security_circle.md) - Setup and protection guidelines, keeping keys out of GitHub and out of AI chat prompts

### Session 4: Synthetic Data
**Topics**: Generating and using synthetic data for marketing

### Session 5: Knowledge Graphs
**Topics**: Building knowledge graphs for marketing insights

### Session 6: Agentic Workflows
**Topics**: Creating autonomous AI agents for marketing automation

## 🛠️ Tools Used

- **Google Colab** (default) - Cloud-based notebooks, no installation
- **Google Gemini API** (default, low-cost pay-as-you-go, billing required) - AI model access
- **GitHub** - Version control and portfolio

**Already have OpenAI or Anthropic API?** You can use those instead — see the optional section in the setup notebook.

## 📖 For Instructors

- [CLAUDE.md](CLAUDE.md) - Repository guidance for AI assistants
- All notebooks designed for Google Colab (one-click from GitHub)
- Local setup available as optional/advanced path

## 🤝 For Students

### Before Each Session
1. Open the session notebook in Colab
2. Make sure your API key is in Secrets (🔑 icon)
3. Run the setup cells

### Tips
- **Save often**: File → Save a copy in GitHub
- **Experiment**: Try changing code to see what happens
- **Watch your spend**: no free quota to fall back on — check pricing at [AI Studio](https://aistudio.google.com/apikey) and use the token cost guide before a big batch job
- **No real data**: Only use public or made-up data

## ⚠️ Important Notes

- **Never commit API keys** - use Colab Secrets or `.env` files (local). See [session_3/api_key_security.ipynb](session_3/api_key_security.ipynb) for how keys leak and how to catch it.
- Billing must be enabled on your Google account before you can get a Gemini API key — this now applies everywhere, not just the EU/UK/Switzerland
- Data-use policy (human review, model training) depends on whether your key is on a "Paid tier" project — check the badge in AI Studio, don't assume

## Optional: Local Setup

For experienced users who prefer local development, see the "Optional B: Local Setup" section in [session_1/setup_guide.ipynb](session_1/setup_guide.ipynb).

Most learners should use Google Colab (the default path).

## 📧 Questions?

Post in the Circle community or open an issue in this repository.

---

**Ready to learn AI for Marketing? Start here:** [Session 1 Setup Guide](session_1/setup_guide.ipynb) 🎯
