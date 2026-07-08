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

## Technology Stack

- **Primary Language**: Python
- **Notebook Environment**: Jupyter notebooks (.ipynb)
- **Expected Tools**: Data science and ML libraries (likely pandas, scikit-learn, transformers, etc.)

## Development Environment

**Primary**: Google Colab (cloud-based, no setup required)
- Provides free GPUs for model training
- Pre-installed data science libraries
- Easy sharing with students

**Alternative**: Local Jupyter notebooks

### Local Development Commands

```bash
# Set up virtual environment
python -m venv venv
source venv/bin/activate  # On macOS/Linux
# or
venv\Scripts\activate  # On Windows

# Install dependencies (once requirements.txt is created)
pip install -r requirements.txt

# Launch Jupyter notebooks
jupyter notebook
# or
jupyter lab

# Run Python scripts
python <script_name>.py
```

### Claude Code Integration

```bash
# Initialize Claude Code in the repository
claude init

# Start interactive session
claude chat

# Common use cases:
# - Ask for code explanations
# - Debug errors
# - Generate example code for marketing use cases
# - Review and optimize existing code
```

## Repository Structure

```
ai4tm-masterclass/
├── session_1/              # AI Fundamentals
│   ├── setup_guide.ipynb          # Claude Code, GitHub, Colab setup
│   ├── setup_guide_circle.md      # Setup guide for Circle platform
│   └── [neural networks, tokenization, embeddings notebooks - TBD]
├── session_2/              # Evaluation (regression, classification, semantic search)
├── session_3/              # Compliance
├── session_4/              # Synthetic Data
├── session_5/              # Knowledge Graphs
├── session_6/              # Agentic Workflows
├── data/                   # Sample datasets
├── utils/                  # Reusable Python modules
├── .gitignore
├── CLAUDE.md              # This file
└── README.md
```

### Session Details

**Session 1: Essential Concepts**
- Focus: Neural networks, attention, tokenization, embeddings
- Approach: Production failure modes (light theory)
- Includes: Setting up Claude Code, GitHub, Google Colab
- Deliverables: Interactive notebooks + Circle content

**Sessions 2-6**: Content to be developed covering evaluation, compliance, synthetic data, knowledge graphs, and agentic workflows

## Key Considerations

### Educational Focus
- Code should be well-commented and accessible to learners
- Notebooks should be self-contained and executable in sequence
- Examples should be relevant to marketing use cases (customer segmentation, sentiment analysis, content generation, etc.)
- Balance educational clarity with practical applicability

### API Keys and Security
- **NEVER commit API keys** to the repository
- Use environment variables (`.env` file) for local development
- Use Google Colab Secrets for cloud development
- Add `.env` to `.gitignore` (already configured)

### Token Cost Management
- Session 1 includes a guide on estimating token consumption and costs
- Always preview token costs before running expensive models
- Use smaller models for testing and iteration
- Consider cost-effective alternatives (e.g., Claude Haiku vs Opus for simple tasks)

### Production Failure Modes
- Session 1 teaches concepts through the lens of production failures
- When building examples, consider real-world edge cases
- Document common pitfalls and how to avoid them
