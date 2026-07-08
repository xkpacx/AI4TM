# Setting Up Your AI Marketing Toolkit

Welcome to AI4TM! This guide will help you set up the three essential tools for the masterclass:
- **Claude Code** - Your AI coding assistant
- **GitHub** - Version control for your projects
- **Google Colab** - Cloud-based Python notebooks with free GPUs

## Why These Tools?

**Claude Code** provides AI-powered assistance for coding, debugging, and learning AI concepts in real-time.

**GitHub** allows you to track experiments, build a portfolio, and collaborate with other learners.

**Google Colab** gives you free access to GPUs and a zero-setup Python environment perfect for AI/ML work.

---

## Part 1: Claude Code Setup

### Installation

**macOS/Linux:**
```bash
curl -fsSL https://claude.ai/install.sh | sh
```

**Windows (PowerShell):**
```powershell
irm https://claude.ai/install.ps1 | iex
```

### Verification
```bash
claude --version
```

### Authentication
```bash
claude login
```

This opens a browser window. Sign in with your Anthropic account.

### Initialize Your Project
```bash
cd /path/to/your/project
claude init
```

### Quick Tips
- Start interactive chat: `claude chat`
- Ask questions: "Explain this error message"
- Generate code: "Create a function to calculate customer lifetime value"
- Review code: "Is this code following best practices?"

---

## Part 2: GitHub Setup

### Create Account
1. Go to [github.com](https://github.com)
2. Click "Sign up"
3. Choose a professional username

### Install Git

- **macOS**: Pre-installed, check with `git --version`
- **Windows**: Download from [git-scm.com](https://git-scm.com)
- **Linux**: `sudo apt-get install git`

### Configure Git
```bash
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

### Create Your Repository

**Option A: GitHub Website**
1. Click "+" icon → "New repository"
2. Name: `ai4tm-masterclass`
3. Description: "AI for Marketing masterclass projects"
4. Choose Public/Private
5. Add README and Python .gitignore
6. Click "Create repository"

**Option B: Command Line**
```bash
mkdir ai4tm-masterclass
cd ai4tm-masterclass
git init
echo "# AI4TM Masterclass" >> README.md
git add README.md
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/ai4tm-masterclass.git
git push -u origin main
```

### Essential Commands

**Save your work:**
```bash
git add .                           # Stage changes
git commit -m "Completed session 1" # Commit with message
git push                            # Upload to GitHub
```

**Check status:**
```bash
git status                          # See what's changed
git log                            # View history
```

**Update local code:**
```bash
git pull                           # Download latest changes
```

---

## Part 3: Google Colab Setup

### Access Colab
1. Go to [colab.research.google.com](https://colab.research.google.com)
2. Sign in with Google account

### Create Your First Notebook
1. Click "File" → "New notebook"
2. Rename it: "Session_1_Setup_Test"

### Enable GPU (for later sessions)
1. Click "Runtime" → "Change runtime type"
2. Select "T4 GPU" or "A100 GPU"
3. Click "Save"

**Note:** Free tier is sufficient for this course.

---

## Part 4: Connect GitHub & Colab

### Open GitHub Notebooks in Colab
1. In Colab: "File" → "Open notebook"
2. Select "GitHub" tab
3. Enter: `YOUR_USERNAME/ai4tm-masterclass`
4. Browse and open notebooks

### Save Colab to GitHub
1. In Colab: "File" → "Save a copy in GitHub"
2. Select `ai4tm-masterclass` repository
3. Choose file path (e.g., `session_1/my_notebook.ipynb`)
4. Add commit message
5. Click "OK"

### Clone Repository in Colab
```python
!git clone https://github.com/YOUR_USERNAME/ai4tm-masterclass.git
%cd ai4tm-masterclass
```

---

## Part 5: Typical Workflow

Here's how these tools work together:

### Example: Building a Sentiment Analysis Model

**1. Plan with Claude Code (Local)**
```bash
claude chat
```
Ask: "Help me design a sentiment analysis pipeline for customer reviews"

**2. Develop in Colab (Cloud)**
- Create notebook
- Write and test code
- Use free GPU for training
- Visualize results

**3. Save to GitHub**
- "File" → "Save a copy in GitHub"
- Commit message: "Add sentiment analysis v1"

**4. Iterate with Claude Code**
- Clone repo locally
- Ask Claude to review
- Get optimization suggestions
- Push improvements

**5. Share Results**
- Share Colab link with stakeholders
- No setup required for viewers
- Code backed up in GitHub

---

## Part 6: Project Structure

Recommended folder structure:

```
ai4tm-masterclass/
├── session_1/          # Neural networks, tokenization, embeddings
├── session_2/          # Evaluation and metrics
├── session_3/          # Compliance
├── session_4/          # Synthetic data
├── session_5/          # Knowledge graphs
├── session_6/          # Agentic workflows
├── data/               # Sample datasets
├── utils/              # Reusable Python code
├── .env               # API keys (DON'T COMMIT)
├── .gitignore         # Excluded files
├── requirements.txt   # Dependencies
└── README.md          # Project overview
```

---

## Part 7: API Keys (For Future Sessions)

### In Google Colab (Recommended)
1. Click key icon (🔑) in left sidebar
2. Click "Add new secret"
3. Name: `ANTHROPIC_API_KEY` (or `OPENAI_API_KEY`)
4. Value: Your API key

Access in code:
```python
from google.colab import userdata
api_key = userdata.get('ANTHROPIC_API_KEY')
```

### For Local Development
Create a `.env` file:
```
ANTHROPIC_API_KEY=your_key_here
```

Load in Python:
```python
from dotenv import load_dotenv
load_dotenv()
```

---

## Best Practices

### Before Each Session
- Run `git pull` to get latest changes
- Review previous session's work

### During Sessions
- Work in Colab for interactive development
- Save frequently to GitHub
- Ask Claude Code when stuck
- Add comments explaining your thinking

### After Sessions
- Commit with meaningful messages
- Ask Claude to review your code
- Update README with learnings
- Share insights in Circle

### Important Reminders
- **Never commit API keys** - use environment variables or Colab secrets
- **Meaningful commit messages** - "Add customer segmentation model" not "update"
- **Comment your code** - help your future self
- **Version experiments** - use git branches for major experiments

---

## Troubleshooting

### Can't install Claude Code?
Check you have curl/PowerShell. Visit [claude.ai/code](https://claude.ai/code) for alternatives.

### Git push requires authentication?
Set up Personal Access Token:
1. GitHub Settings → Developer settings → Personal access tokens
2. Generate new token with `repo` scope
3. Use token as password

### Colab disconnects during training?
- Save checkpoints frequently
- Keep browser tab active
- Consider Colab Pro for longer sessions

### Out of memory in Colab?
- Reduce batch size
- Use smaller models
- Clear unused variables: `del large_variable`
- Restart runtime

### Library import errors?
```python
!pip install library_name --upgrade
```
Then restart runtime.

---

## Verification Checklist

Before moving forward, ensure:

- [ ] Claude Code installed and authenticated
- [ ] GitHub account created
- [ ] Git configured (name and email)
- [ ] Repository created: `ai4tm-masterclass`
- [ ] Can access Google Colab
- [ ] Can save Colab notebooks to GitHub
- [ ] Understand basic git commands
- [ ] Know how to enable GPU in Colab

---

## Helpful Resources

### Documentation
- [Claude Code Docs](https://docs.claude.com/claude-code)
- [GitHub Guides](https://guides.github.com)
- [Google Colab FAQ](https://research.google.com/colaboratory/faq.html)

### Learning
- [Git Cheat Sheet](https://education.github.com/git-cheat-sheet-education.pdf)
- [Python for Data Analysis](https://wesmckinney.com/book/)
- [Hugging Face Course](https://huggingface.co/course)

### Support
- AI4TM Circle Community
- GitHub repository discussions
- Stack Overflow for technical questions

---

## You're Ready!

With these tools set up, you're ready to dive into AI for marketing. In the upcoming sessions, you'll learn about neural networks, tokenization, embeddings, and much more.

See you in the next session! 🚀

---

**Questions?** Post in Circle or reach out to your instructors.
