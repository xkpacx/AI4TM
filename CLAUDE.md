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

### Default Path: Free and Accessible
- **Primary environment**: Google Colab (cloud-based, no installation required)
- **Primary API**: Google Gemini (free tier - 250 requests/day, no credit card)
- **Paid APIs**: OpenAI/Anthropic are optional alternatives for users who already have keys
- **Local setup**: Available only as an optional "advanced" path for experienced users

### Language and Accessibility
- Avoid unexplained jargon (e.g., "venv", "SDK", "runtime", "environment variable")
- Define technical terms the first time they appear
- Write for marketing professionals learning AI, not software engineers
- Use analogies and plain language

### Privacy and Security
- Clearly flag that free-tier API usage may be used to improve models
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

Only for experienced users who prefer local control.

```bash
# Create virtual environment (isolated Python installation)
python -m venv venv
source venv/bin/activate  # macOS/Linux
# or
venv\Scripts\activate  # Windows

# Install dependencies
pip install -r requirements.txt

# Launch Jupyter
jupyter notebook
```

## Repository Structure

```
ai4tm-masterclass/
├── session_1/              # AI Fundamentals
│   ├── setup_guide.ipynb          # FREE Colab + Gemini setup (default path)
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
- Default setup: Google Colab + free Gemini API
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
- Always preview token costs before running expensive models
- Default to free Gemini API (250 requests/day)
- Use smaller models for testing and iteration
- Consider cost-effective alternatives (e.g., Gemini 2.0 Flash vs larger models)

### Data Privacy
- Free-tier APIs (Google, OpenAI, etc.) may use data for model training
- Human reviewers may read inputs/outputs on free tiers
- **Only use public, synthetic, or anonymized data** in course materials
- Flag this prominently in all setup documentation
- Paid tiers available for production/confidential data (not required for course)

### Production Failure Modes
- Session 1 teaches concepts through the lens of production failures
- When building examples, consider real-world edge cases
- Document common pitfalls and how to avoid them
- Focus on what can go wrong in marketing AI applications

## Current SDK Versions (Verified 2025)

### Primary (Free Tier)
- **Google Gemini**: `google-genai` (NOT `google-generativeai` - deprecated Nov 2025)
  - Install: `pip install google-genai`
  - Import: `from google import genai`
  - Model: `gemini-2.0-flash-exp` (fast, free)

### Optional (Paid APIs)
- **OpenAI**: `openai>=1.0.0`
- **Anthropic**: `anthropic>=0.18.0`

### Data Science Libraries
- pandas, numpy, matplotlib, seaborn (pre-installed in Colab)

## Workflow for Students

1. Get free Gemini API key from aistudio.google.com/apikey
2. Open notebook in Colab (one-click from GitHub)
3. Store API key in Colab Secrets (🔑 icon)
4. Run notebook cells
5. Save work back to GitHub (File → Save to GitHub)

## Workflow for Content Creators

When creating new notebooks:
1. Design for Google Colab first (default path)
2. Use `google-genai` for AI calls (free tier)
3. Explain technical terms when first used
4. Include privacy warnings for free-tier usage
5. Provide working code examples with marketing context
6. Test in Colab before committing
7. Include "Open in Colab" badge in README

## Common Pitfalls to Avoid

1. **Assuming local setup**: Most students will use Colab, not local Jupyter
2. **Jargon without explanation**: Define terms like "runtime", "kernel", "cell", "environment"
3. **Paid APIs as default**: Default to free Gemini, offer paid as optional
4. **Ignoring privacy**: Always warn about free-tier data usage
5. **Out-of-date package names**: Verify current SDK names (google-genai, not google-generativeai)
6. **Complex dependencies**: Keep it simple, use Colab's pre-installed libraries when possible

## Notes for AI Assistants (Claude Code, etc.)

- When generating code for this repo, default to Google Colab context
- Use `from google.colab import userdata` for API keys (not os.getenv unless local)
- Explain what each piece of code does in plain language
- Provide marketing-relevant examples, not generic demos
- Include privacy warnings when using free-tier APIs
- Verify package names are current before suggesting installation commands
- Frame paid APIs as "if you already have a key" not "you need to get"
