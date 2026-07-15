# Setting Up Your Free AI Marketing Toolkit

Welcome to AI4TM! This guide will help you set up everything you need — **100% free, no credit card required**.

## What You'll Set Up (10 minutes total)

1. **Google Colab** - Write and run code in your browser
2. **Free Gemini API** - Access Google's AI (250 requests/day, free forever)
3. **GitHub** - Save and share your projects

**Requirements**: Just a Google account (same as Gmail)

---

## 🎯 Your Free Learning Path

This course uses **Google's free tier** by default:
- ✅ No credit card needed
- ✅ 250 AI requests per day
- ✅ Resets daily
- ✅ Never expires
- ✅ Same Google login for everything

**Already paying for OpenAI or Anthropic?** You can use those instead — see the optional section at the end.

### ⚠️ Important Privacy Note

With **any free AI API** (Google, OpenAI free tier, etc.):
- Your requests may be read by human reviewers
- Your data may be used to train the AI models

**For this course, use only**:
- Public example data
- Made-up (synthetic) data
- Anonymized datasets we provide

**Never send**: Client data, confidential info, or proprietary content.

*Need full privacy? Upgrade to a paid API plan (not required for this course).*

---

## Part 1: Open Notebooks in Google Colab

### What is Google Colab?

Colab is Google's free tool for writing and running code in your browser. Think "Google Docs for code" — no installation, auto-saves, shareable links.

### How to Open Course Notebooks

1. Go to the course GitHub repository (link provided by instructor)
2. Click on any `.ipynb` file (like `setup_guide.ipynb`)
3. Click the **"Open in Colab"** badge at the top
4. The notebook opens in a new tab

### Quick Colab Basics

- **Code cells**: Have a ▶️ play button. Click to run them.
- **Text cells**: Just explanations, for reading only.
- **Save your work**: File → Save a copy in Drive (or Save to GitHub)

---

## Part 2: Get Your Free Gemini API Key

### What is an API key?

An API key is like a password that lets your code talk to Google's AI. It looks like: `AIzaSyD...`

### Get Your Free Key (2 minutes)

1. Go to: [aistudio.google.com/apikey](https://aistudio.google.com/apikey)
2. Sign in with your Google account
3. Click **"Create API Key"**
4. Let it create a new project (click "Continue")
5. **Copy the key** (starts with `AIza`)

### ⚠️ EU/UK/Switzerland Note

If you're in the EEA, UK, or Switzerland, Google may ask you to enable billing for regulatory reasons. You won't be charged if you stay within the free quota (250 requests/day).

### Store Your Key Securely in Colab

**Never paste API keys directly in code!** Use Colab Secrets:

1. In Colab, look at the left sidebar
2. Click the **🔑 key icon** (called "Secrets")
3. Click **"+ Add new secret"**
4. **Name**: `GEMINI_API_KEY`
5. **Value**: Paste your API key
6. **Toggle the switch** to allow this notebook access
7. Click **"Save"**

---

## Part 3: Test Your Setup

### Install Google's AI Library

In any code cell, run:

```python
!pip install -q google-genai
```

The `!` means "run this as a command, not code." The `-q` means "quiet" (less output).

### Test Your Connection

```python
from google import genai
from google.colab import userdata

# Get your API key from Secrets
client = genai.Client(api_key=userdata.get('GEMINI_API_KEY'))

# Ask Gemini a question
response = client.models.generate_content(
    model='gemini-2.5-flash-lite',
    contents='In one sentence, what is customer segmentation?'
)

print(response.text)
```

**If you see a response**: 🎉 It works!

**If you see an error**: Check:
- Did you add the key to Secrets? (🔑 icon)
- Named it exactly `GEMINI_API_KEY`?
- Toggled the switch to allow access?

---

## Part 4: Understanding Your Free Quota

### What You Get

- **10 requests per minute**
- **250 requests per day**
- **Resets every 24 hours**
- **Never expires**

### Is This Enough?

**Yes!** Each session uses 20-50 requests. Even with experimentation, you won't hit the limit.

### If You Hit the Limit?

You'll get an error. Just wait until tomorrow (quota auto-resets) or upgrade to paid if needed.

---

## Part 5: GitHub - Saving Your Work

### Why GitHub?

GitHub is like Google Drive for code. It:
- Saves different versions (undo mistakes)
- Lets you share projects
- Builds your AI portfolio

### Create Account (3 minutes)

1. Go to [github.com](https://github.com)
2. Click **"Sign up"**
3. Choose username (like `yourname-marketing`)
4. Verify email

### Create Your Repository

A "repository" (repo) is a folder for your project files.

1. On GitHub, click **"+"** (top right)
2. Select **"New repository"**
3. **Name**: `ai4tm-masterclass`
4. **Description**: "AI for Marketing projects"
5. **Public** or **Private** (your choice)
6. Check ✅ **"Add a README"**
7. **gitignore**: Choose "Python"
8. Click **"Create repository"**

### Save Colab Notebooks to GitHub

1. In Colab: **File → Save a copy in GitHub**
2. **Authorize** Colab (first time only)
3. **Select repo**: `your-username/ai4tm-masterclass`
4. **File path**: `session_1/setup_guide.ipynb`
5. **Commit message**: "Complete setup guide"
6. Click **OK**

Your work is now saved and backed up!

---

## Part 6: Quick Reference

### Basic Code Template

```python
from google import genai
from google.colab import userdata

# Connect to Gemini
client = genai.Client(api_key=userdata.get('GEMINI_API_KEY'))

# Ask something
response = client.models.generate_content(
    model='gemini-2.5-flash-lite',
    contents='Your question here'
)

print(response.text)
```

### Example: Analyze Customer Sentiment

```python
from google import genai
from google.colab import userdata

client = genai.Client(api_key=userdata.get('GEMINI_API_KEY'))

review = "Great product, but shipping was slow."

prompt = f'''Analyze this customer review: "{review}"

Provide:
1. Sentiment (positive/negative/mixed)
2. Confidence (0-100%)
3. Key phrases
'''

response = client.models.generate_content(
    model='gemini-2.5-flash-lite',
    contents=prompt
)

print(response.text)
```

### Example: Generate Marketing Copy

```python
from google import genai
from google.colab import userdata

client = genai.Client(api_key=userdata.get('GEMINI_API_KEY'))

prompt = '''Write 3 social media captions (max 280 chars each) for:
Product: eco-friendly water bottle
Audience: environmentally conscious millennials

Make them engaging with a call-to-action.
'''

response = client.models.generate_content(
    model='gemini-2.5-flash-lite',
    contents=prompt
)

print(response.text)
```

---

## ✅ Setup Complete!

You've now:
- Set up Google Colab
- Got a free Gemini API key
- Tested your AI connection
- Created a GitHub account
- Saved your first notebook

### Before Each Session

1. Open the session notebook in Colab
2. Check your API key is in Secrets (🔑)
3. Run the setup cells at the top

### Tips for Success

- **Save often**: File → Save to GitHub
- **Experiment**: Try changing the code
- **Stay in quota**: 250/day is plenty
- **No real data**: Only public/synthetic data

---

## Optional: Advanced Setups

The sections below are **NOT required**. Only read if:
- You already have a paid API key (OpenAI/Anthropic)
- You want to run code locally instead of Colab
- You're experienced and want more control

---

### Optional A: Using Paid APIs

**Only if you already have an API key.**

#### OpenAI (ChatGPT)

```python
!pip install -q openai

from openai import OpenAI
from google.colab import userdata

client = OpenAI(api_key=userdata.get('OPENAI_API_KEY'))

response = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[{"role": "user", "content": "What is customer segmentation?"}]
)

print(response.choices[0].message.content)
```

*Store your OpenAI key in Secrets as: `OPENAI_API_KEY`*

#### Anthropic (Claude)

```python
!pip install -q anthropic

from anthropic import Anthropic
from google.colab import userdata

client = Anthropic(api_key=userdata.get('ANTHROPIC_API_KEY'))

response = client.messages.create(
    model="claude-3-5-haiku-20241022",
    max_tokens=1024,
    messages=[{"role": "user", "content": "What is customer segmentation?"}]
)

print(response.content[0].text)
```

*Store your Anthropic key in Secrets as: `ANTHROPIC_API_KEY`*

---

### Optional B: Local Setup (Advanced)

**For experienced users only.** Most learners should use Colab.

#### Prerequisites

- Python 3.8+ installed
- Comfortable with command line
- Understand file paths

#### Steps

```bash
# Download the repository
git clone https://github.com/YOUR_USERNAME/ai4tm-masterclass.git
cd ai4tm-masterclass

# Create isolated Python environment
python -m venv venv

# Turn it on
source venv/bin/activate  # macOS/Linux
venv\Scripts\activate     # Windows

# Install packages
pip install -r requirements.txt

# Start Jupyter
jupyter notebook
```

#### Store API Key Locally

Create `.env` file in project folder:

```
GEMINI_API_KEY=your_key_here
```

Use it in code:

```python
from dotenv import load_dotenv
import os

load_dotenv()
api_key = os.getenv('GEMINI_API_KEY')
```

*The `.env` file won't upload to GitHub (already in `.gitignore`)*

---

## Resources

### Get Help
- Course Circle community
- [Google Colab FAQ](https://research.google.com/colaboratory/faq.html)
- [GitHub Guides](https://guides.github.com)

### Documentation
- [Google AI Studio](https://aistudio.google.com)
- [Gemini API Docs](https://ai.google.dev/gemini-api/docs)

### Check Usage
- [Your API Keys](https://aistudio.google.com/app/apikey)

---

**You're ready! Move on to the neural networks notebook. 🚀**
