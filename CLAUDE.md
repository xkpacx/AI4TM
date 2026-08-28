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
- **Primary API**: Google Gemini. A genuine no-card free tier still exists in most regions, including the US (confirmed 2026-08-27; verify before restating, this has been wrong before, see Notes for AI Assistants below). This course has students enable billing anyway, everywhere, because that's what puts usage on Google's "Paid tier" data-handling terms (not used for training, skips human review), not because Google forces it outside the EEA/UK/Switzerland, where it does. Usage on the lightweight models this course uses (Flash / Flash-Lite) runs at pay-as-you-go rates, typically fractions of a cent per request, but confirm current pricing and whether the key is on a "Paid tier" project at aistudio.google.com before planning a budget. Gemini isn't available at all in mainland China or Hong Kong.
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

- **Colab**: installs from an explicit package list written directly into that cell. Not from `week_N/requirements.txt` — opening a notebook via the "Open in Colab" badge loads only that one file, not the rest of the repository, so there's no sibling file to read. Keep this list in sync with `week_N/requirements.txt` by hand when either changes.
- **Local (any IDE — VS Code, PyCharm, etc.)**: installs from `week_N/requirements.txt`, one persistent `uv`-managed venv per session, created once from a terminal before the notebook is opened:

```bash
cd week_N
uv venv
uv pip install -r requirements.txt   # requirements.txt already lists ipykernel
```

Then point the IDE's kernel/interpreter picker at `week_N/.venv/bin/python` (`week_N\.venv\Scripts\python.exe` on Windows). The notebook's own setup cell looks up `requirements.txt` by directory (checking `week_N` is either the cwd, a child, or a sibling of cwd), not by bare filename, since the repo root also has its own `requirements.txt` for whole-repo setup that a naive filename search could grab by mistake. A lighter, IDE-free alternative (`uv run --with jupyter jupyter lab`, run from inside the session folder) still works too, for a disposable browser-tab session instead of a persistent one a kernel picker points at.

Full walkthrough lives in `week_2/lesson08_setup_guide.ipynb`'s "Optional B" section; every other notebook points back to it rather than duplicating the explanation, though each keeps its own copy of the actual uv commands and Colab package list inline (checked against Krasimir's explicit preference for this exact split, 2026-08-16).

## Repository Structure

```
AI4TM/
├── week_2/              # AI Fundamentals
│   ├── lesson08_setup_guide.ipynb          # Colab + Gemini setup (default path, billing required)
│   ├── lesson08_setup_guide_circle.md      # Text version for Circle platform
│   └── [neural networks, tokenization, embeddings notebooks - TBD]
├── week_3/              # Evaluation (regression, classification, semantic search)
├── week_4/              # Compliance
├── week_5/              # Synthetic Data
├── week_6/              # Knowledge Graphs
├── week_7/              # Agentic Workflows
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

## Current SDK Versions (Verified 2025, model updated 2026-08)

### Primary (Low-Cost, Billing Required)
- **Google Gemini**: `google-genai` (NOT `google-generativeai` - deprecated Nov 2025)
  - Install: `pip install google-genai`
  - Import: `from google import genai`
  - Model: `gemini-3.5-flash-lite` (fast, low-latency; not the cheapest Gemini option, Google's official 2.5-flash-lite successor `gemini-3.1-flash-lite` costs less, see [week_2/lesson09_token_cost_guide_circle.md](week_2/lesson09_token_cost_guide_circle.md), but 3.5 is what the notebooks call. Confirm current pricing before quoting a figure)

### Optional (Paid APIs)
- **OpenAI**: `openai>=1.0.0`
- **Anthropic**: `anthropic>=0.18.0`

### Data Science Libraries
- pandas, numpy, matplotlib, seaborn (pre-installed in Colab)

## Workflow for Students

1. Fork the repository (`github.com/YOUR_USERNAME/AI4TM`) — one time, at the start of Session 1
2. Get a Gemini API key from aistudio.google.com/apikey (requires enabling billing — see Session 1 setup guide)
3. Open notebook in Colab (one-click from GitHub)
4. Store API key in Colab Secrets (🔑 icon)
5. Run notebook cells
6. Save work back to GitHub, into your fork (File → Save to GitHub)

Students are never added as collaborators on the origin repo; forking is the only mechanism they use to get a personal, writable copy.

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
3. **Claiming this course uses Gemini's free tier**: it doesn't, by design, even though a genuine no-card free tier does still exist in most regions. This course has students enable billing everywhere regardless, for the data-handling reason (paid usage isn't used for training), so frame the choice as deliberate, not as Google leaving no option. Don't restate "no free tier exists" as a bare fact; it's specifically false and has caused real content errors before (see Notes for AI Assistants)
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
- **Re-verify billing/free-tier/pricing claims against a live source before restating them, even ones already written in this file or elsewhere in the repo.** The "Google requires billing everywhere, no free tier left" claim sat in this file's Core Design Decisions section for an unknown stretch of time and got extended into a dozen more files across a full session before anyone checked it against Google's actual docs (2026-08-27). It was false: a genuine no-card free tier exists in most regions including the US; billing is only Google-mandated in the EEA/UK/Switzerland, for data-handling reasons, not a money reason; Gemini isn't available at all in mainland China or Hong Kong. Model pricing drifts the same way, a price checked earlier the same day can be stale by the afternoon (Claude Sonnet 5's "introductory" rate quietly became permanent). Treat any absolute claim ("no longer", "everywhere", "always", "the standard rate") as worth one fresh check before it gets copied into new content.
- **Jupyter/Colab notebooks (`.ipynb`) run markdown through MathJax, which treats a bare `$` as an inline-math delimiter.** A markdown table (or any cell) containing literal dollar signs (`$0.30`, `Input $/1M tokens`) will render broken, mangled, or silently absorb surrounding table structure into a math span. Escape every literal `$` as `\$` in notebook markdown cells. This does **not** apply to the `_circle.md` companion files or any other plain `.md` file in the repo, only to `.ipynb` markdown cells actually rendered through Colab/Jupyter.
- **When editing `.ipynb` files directly (not through a notebook tool), prefer `json.load` then modify the Python string value then `json.dump(nb, f, indent=1, ensure_ascii=False)`**, over raw string/regex substitution on the file text. This preserves the original cell formatting (single-string vs list-of-lines `source`) with no reformatting noise in the diff, and sidesteps manual backslash-escaping mistakes (a raw-text edit needs `\\\\$` in a non-raw Python string literal to produce a decoded `\$`, easy to get wrong and silently write invalid JSON). If you do use raw string substitution, run `json.load()` on the result before considering the edit done. If a batch of substitutions is written as "collect all, assert none missing, write," a single missing pattern silently discards the *entire* batch, not just the one that didn't match, so re-verify with a fresh read after any such script reports success.
- **File naming as of the 2026-08-27 folder rename**: lesson files are `week_N/lesson{NN}_slug.ext` (e.g. `week_2/lesson08_setup_guide.ipynb`), the week number lives only in the folder name, not repeated in the filename. Only rename a file into this pattern once you have a Circle-confirmed lesson number for it; don't invent one. After any path rename, grep the *entire* repo (README, CLAUDE.md, every other week's "Optional B" pointer links, notebooks' own self-referencing Colab badge URLs) for the old path, and check `.gitignore` for stale path-prefixed patterns: a rename without updating `.gitignore` can make previously-excluded generated/local-only files (datasets, model checkpoints, caches) suddenly stageable again under the new path.
- **This repo's folders now say "week_N" (matching Circle), but prose throughout still says "Session N"** (notebook headers, README section titles, CLAUDE.md's own "6 sessions" framing above). This was a deliberate, explicit decision by Krasimir, not an oversight. Don't silently rename folders back to "session_N", and don't silently rewrite "Session" prose to "Week" either; both are the user's call to make, not something to infer from the mismatch.
- **GitHub PR gotcha, easy to miss**: a PR's commits land in `main` only if its *base* branch is `main` at merge time. If you keep pushing to a branch whose PR already merged, those later commits do **not** retroactively appear in `main`, even though `gh pr view` on the old PR still says `MERGED`. This happened three separate times in the 2026-08-27 session, each caught only because the user asked "is it pushed to main" or pointed at GitHub's file browser. Before assuming a push is "in main," check `git log origin/main..origin/<branch> --oneline` for anything still ahead, and always confirm a new PR's `--base` is actually `main`, not a stale sibling branch.
