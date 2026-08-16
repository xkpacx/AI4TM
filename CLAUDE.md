# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AI4TM is a masterclass repository covering AI for Marketing across 6 sessions:
1. AI fundamentals
2. Evaluation
3. Compliance
4. Synthetic data
5. Knowledge graphs
6. Agentic workflows

The repository contains educational content, notebooks, and guides for teaching AI applications in marketing contexts.

## Core Design Decisions

### Default Path: Low-Cost and Accessible
- **Primary environment**: Google Colab (cloud-based, no installation required)
- **Primary API**: Google Gemini — as of 2026, Google requires a billing method on file before issuing an API key; there is no longer a no-cost, no-card tier. Usage on the lightweight models this course uses (Flash / Flash-Lite) runs at pay-as-you-go rates, typically fractions of a cent per request, but confirm current pricing and whether the key is on a "Paid tier" project at aistudio.google.com before planning a budget
- **Paid APIs**: OpenAI/Anthropic are optional alternatives for users who already have keys
- **Local setup**: Available only as an optional "advanced" path for experienced users

### Language and Accessibility
- Avoid unexplained jargon (e.g., "venv", "SDK", "runtime", "environment variable")
- Define technical terms the first time they appear
- Write for marketing professionals learning AI, not software engineers
- Use analogies and plain language

### Privacy and Security
- Clearly flag that unpaid/free-quota AI usage may be reviewed by humans or used to improve models; Google's terms say billed ("Paid tier") Gemini usage is not used for training and skips human review for that purpose, but confirm the key is actually on a Paid tier project before telling students that protection applies to them
- Warn users not to paste confidential/client data during the course
- Use only public, synthetic, or anonymized datasets in examples
- Never commit API keys (use Colab Secrets or `.env` for local)

## Technology Stack

- **Primary Environment**: Google Colab
- **Primary Language**: Python 3.8+
- **Primary AI API**: Google Gemini (`google-genai` package)
- **Notebook Format**: Jupyter notebooks (.ipynb)
- **Version Control**: GitHub
- **Optional Local**: Jupyter Lab/Notebook

## Development Environment

### Default: Google Colab (Recommended for Students)
- Provides free GPU access for model training
- Pre-installed data science libraries
- Easy sharing with students via links
- One-click "Open in Colab" from GitHub
- No local installation required

### Optional: Local Development (Advanced Users Only)

Only for experienced users who prefer local control. As of 2026-08, new local-setup instructions should use [uv](https://docs.astral.sh/uv/) rather than a manually created `venv`, since it gives each session its own disposable, isolated environment (matching the isolation Colab already provides for free) without the student needing to create or activate anything by hand, and it manages Python itself if none is installed. As of 2026-08-16, every session notebook's own setup cell auto-detects Colab vs. local (`import google.colab`) and installs accordingly, in one of two ways:

- **Colab**: installs from an explicit package list written directly into that cell. Not from `session_N/requirements.txt` — opening a notebook via the "Open in Colab" badge loads only that one file, not the rest of the repository, so there's no sibling file to read. Keep this list in sync with `session_N/requirements.txt` by hand when either changes.
- **Local (any IDE — VS Code, PyCharm, etc.)**: installs from `session_N/requirements.txt`, one persistent `uv`-managed venv per session, created once from a terminal before the notebook is opened:

```bash
cd session_N
uv venv
uv pip install -r requirements.txt   # requirements.txt already lists ipykernel
```

Then point the IDE's kernel/interpreter picker at `session_N/.venv/bin/python` (`session_N\.venv\Scripts\python.exe` on Windows). The notebook's own setup cell looks up `requirements.txt` by directory (checking `session_N` is either the cwd, a child, or a sibling of cwd), not by bare filename, since the repo root also has its own `requirements.txt` for whole-repo setup that a naive filename search could grab by mistake. A lighter, IDE-free alternative (`uv run --with jupyter jupyter lab`, run from inside the session folder) still works too, for a disposable browser-tab session instead of a persistent one a kernel picker points at.

Full walkthrough lives in `session_1/setup_guide.ipynb`'s "Optional B" section; every other notebook points back to it rather than duplicating the explanation, though each keeps its own copy of the actual uv commands and Colab package list inline (checked against Krasimir's explicit preference for this exact split, 2026-08-16).

## Repository Structure

```
ai4tm-masterclass/
├── session_1/              # AI Fundamentals
│   ├── setup_guide.ipynb          # Colab + Gemini setup (default path, billing required)
│   ├── setup_guide_circle.md      # Text version for Circle platform
│   └── [neural networks, tokenization, embeddings notebooks - TBD]
├── session_2/              # Evaluation (regression, classification, semantic search)
├── session_3/              # Compliance
├── session_4/              # Synthetic Data
├── session_5/              # Knowledge Graphs
├── session_6/              # Agentic Workflows
├── data/                   # Sample datasets (public/synthetic only)
├── utils/                  # Reusable Python modules
├── .gitignore
├── CLAUDE.md              # This file
├── README.md
└── requirements.txt       # For optional local setup only
```

### Session Details

**Session 1: Essential Concepts**
- Focus: Neural networks, attention, tokenization, embeddings
- Approach: Production failure modes (light theory)
- Default setup: Google Colab + Gemini API (billing enabled, low pay-as-you-go cost)
- Deliverables: Interactive notebooks + Circle content
- Includes token cost estimation guide

**Sessions 2-6**: Content to be developed covering evaluation, compliance, synthetic data, knowledge graphs, and agentic workflows

## Key Considerations

### Educational Focus
- Code should be well-commented and accessible to learners without CS backgrounds
- Notebooks should be self-contained and executable in sequence
- Examples should be relevant to marketing use cases (customer segmentation, sentiment analysis, content generation, campaign optimization)
- Balance educational clarity with practical applicability
- Define technical terms when first used (e.g., "An API key is like a password...")

### API Keys and Security
- **NEVER commit API keys** to the repository
- Default: Use Google Colab Secrets (for Colab notebooks)
- Alternative: Use `.env` file for local development
- Add `.env` to `.gitignore` (already configured)
- Verify `.env` is in .gitignore before any commit

### Token Cost Management
- Session 1 includes a guide on estimating token consumption and costs
- Always preview token costs before running expensive models — there is no free tier to fall back on if an estimate is skipped
- Default to Gemini's lightweight models (Flash / Flash-Lite) for the lowest per-request cost; confirm current pricing and any rate limits at aistudio.google.com, since Google adjusts both
- Use smaller models for testing and iteration
- Consider cost-effective alternatives (e.g., Flash-Lite vs larger models)

### Data Privacy
- Unpaid/free-quota AI usage (Google, OpenAI, etc.) may be used for model training and reviewed by humans; Google's terms say billed Gemini usage on a "Paid tier" project is not used for training and skips human review, but this must be verified per account rather than assumed
- **Only use public, synthetic, or anonymized data** in course materials
- Flag this prominently in all setup documentation
- Never assume a provider's current data-use policy — link to the provider's live terms rather than restating a remembered one, since these change

### Production Failure Modes
- Session 1 teaches concepts through the lens of production failures
- When building examples, consider real-world edge cases
- Document common pitfalls and how to avoid them
- Focus on what can go wrong in marketing AI applications

## Current SDK Versions (Verified 2025)

### Primary (Low-Cost, Billing Required)
- **Google Gemini**: `google-genai` (NOT `google-generativeai` - deprecated Nov 2025)
  - Install: `pip install google-genai`
  - Import: `from google import genai`
  - Model: `gemini-2.5-flash-lite` (fast, cheapest available model — confirm current pricing before quoting a figure)

### Optional (Paid APIs)
- **OpenAI**: `openai>=1.0.0`
- **Anthropic**: `anthropic>=0.18.0`

### Data Science Libraries
- pandas, numpy, matplotlib, seaborn (pre-installed in Colab)

## Workflow for Students

1. Get a Gemini API key from aistudio.google.com/apikey (requires enabling billing — see Session 1 setup guide)
2. Open notebook in Colab (one-click from GitHub)
3. Store API key in Colab Secrets (🔑 icon)
4. Run notebook cells
5. Save work back to GitHub (File → Save to GitHub)

## Workflow for Content Creators

When creating new notebooks:
1. Design for Google Colab first (default path)
2. Use `google-genai` for AI calls (billing required, low pay-as-you-go cost)
3. Explain technical terms when first used
4. Include a privacy note on the provider's current data-use policy for the tier the key is on
5. Provide working code examples with marketing context
6. Test in Colab before committing
7. Include "Open in Colab" badge in README

## Common Pitfalls to Avoid

1. **Assuming local setup**: Most students will use Colab, not local Jupyter
2. **Jargon without explanation**: Define terms like "runtime", "kernel", "cell", "environment"
3. **Claiming Gemini is free**: It isn't anymore — billing must be enabled to get a key at all; default to Gemini for its low pay-as-you-go cost and Colab convenience, not because it's free, and offer paid alternatives (OpenAI/Anthropic) as optional
4. **Ignoring privacy**: Always warn that data-use policy depends on which tier the key is billed under, and verify rather than assume
5. **Out-of-date package names**: Verify current SDK names (google-genai, not google-generativeai)
6. **Complex dependencies**: Keep it simple, use Colab's pre-installed libraries when possible

## Notes for AI Assistants (Claude Code, etc.)

- When generating code for this repo, default to Google Colab context
- Use `from google.colab import userdata` for API keys (not os.getenv unless local)
- Explain what each piece of code does in plain language
- Provide marketing-relevant examples, not generic demos
- Include privacy warnings that reflect the actual tier/billing status of the API key in use, not a remembered "free tier" default
- Verify package names are current before suggesting installation commands
- Frame paid APIs as "if you already have a key" not "you need to get"
