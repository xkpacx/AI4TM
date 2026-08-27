# Hands-on: set up your AI assistant

Setup takes about ten minutes. You'll need a Google account and the [GitHub account from Week 1](https://community.teamsimmer.com/c/ai-for-technical-marketers/sections/1009688/lessons/3914861). This course has you enable billing on that account before creating your Gemini API key, even though a genuine no-card free tier still exists in most regions, including the US (see the privacy note below for why). If you're in the EEA, UK, or Switzerland, billing isn't optional either way, Google won't issue a key there without it. Gemini isn't available at all in mainland China or Hong Kong; if that's you, use the OpenAI or Anthropic path in the optional section instead. Actual cost, once billing is on, is low: the lightweight models used in this course run at fractions of a cent per request, and the token cost guide later this session shows how to check before running anything larger.

One Google login covers Colab and Gemini both. If you already hold a paid API key for OpenAI or Anthropic, you can use that instead. See the optional section at the end.

> **A note on privacy, and why this course uses billing even though a free option exists.** A genuine no-card free tier for Gemini is still available in most regions, including the US. This course has you enable billing anyway, because data-use policy depends on it: Google's terms say unpaid/free-quota usage may be reviewed by human moderators and used to train future models, while Paid tier usage is not used for training and skips that review. Check the badge in AI Studio to confirm which applies to your key, rather than assuming. Either way, for this course: work only with public example data, synthetic data, or the anonymised datasets in the materials. Don't send client data, confidential information, or proprietary content through any AI API, paid or not. Week 4 covers this properly.

---

## Part 1: fork the course repository

A **fork** is your own personal copy of the whole course repository, made under your own GitHub account. It looks and works exactly like the original, but changes you make only affect your copy, never the original, and an update to the original never overwrites anything you've saved in your fork.

1. Go to the course GitHub repository (link provided by your instructor): [github.com/xkpacx/AI4TM](https://github.com/xkpacx/AI4TM).
2. Click **Fork**, near the top right of the page.
3. Click **Create fork**. GitHub creates `github.com/YOUR_USERNAME/AI4TM`, your own copy.

You won't need your fork again until Part 6, where you save your first notebook into it.

---

## Part 2: open notebooks in Colab

To open a course notebook:

1. Go to the course GitHub repository (link provided by your instructor).
2. Click any `.ipynb` file, such as `setup_guide.ipynb`.
3. Click the "Open in Colab" badge at the top of the file.

Notebooks are built from **code cells**, which have a play button and run when clicked, and **text cells**, which are there to be read. You'll mostly run code cells in order, top to bottom.

To save: **File → Save a copy in Drive**, or **File → Save a copy in GitHub** to commit straight to your fork (Part 6).

---

## Part 3: get your Gemini API key

Your API key is what authenticates your code as you. Anyone who has it can spend against your account, so treat it accordingly.

1. Go to [aistudio.google.com/apikey](https://aistudio.google.com/apikey).
2. Sign in with your Google account.
3. Click "Create API Key."
4. Enable billing. If you're in the EEA, UK, or Switzerland, Google requires this before it will issue a key at all. Everywhere else, including the US, a no-card free tier exists and Google won't force this step, but enable it anyway: it's what puts your usage on Google's "Paid tier" data-handling terms (see the privacy note above). If you don't already have a Google Cloud billing account, [this guide walks through creating one](https://cloud.google.com/billing/docs/how-to/create-billing-account) (a credit or debit card, added once).
5. Let Google create a new project for you (click "Continue" when prompted).
6. Copy the key. It starts with `AIza`.

> **On cost:** once billing is on, usage is billed at pay-as-you-go rates, fractions of a cent per request for the lightweight models used in this course. With billing enabled, there's no free quota behind your usage, so a mistaken large batch job costs real money instead of just hitting a daily limit. That's the tradeoff for the privacy benefit above; the token cost guide covers checking before you run one.

### Store the key in Colab Secrets

**Never paste an API key into your code.** Anyone who sees that code, whether in a shared notebook or a public repository, then has your key. Colab Secrets prevents this:

1. In Colab, open the left sidebar and click the key icon, labelled "Secrets."
2. Click "+ Add new secret."
3. Name it `GEMINI_API_KEY`.
4. Paste your key as the value.
5. Toggle the switch to grant this notebook access.
6. Click "Save."

Week 4 goes further into key handling, including `.env` files and `.gitignore` for work outside Colab.

---

## Part 4: test your setup

Install Google's Gemini library in a code cell:

```python
!pip install -q google-genai
```

The `!` runs the line as a system command rather than Python, and `-q` suppresses the installation log.

Then test the connection:

```python
from google import genai
from google.colab import userdata

# Get your API key from Secrets
client = genai.Client(api_key=userdata.get('GEMINI_API_KEY'))

# Ask Gemini a question
response = client.models.generate_content(
    model='gemini-3.5-flash-lite',
    contents='In one sentence, what is customer segmentation?'
)

print(response.text)
```

If you get an error, check that the key is in Secrets, that it's named exactly `GEMINI_API_KEY`, and that the access toggle is on. If the error mentions the model name, check [AI Studio](https://aistudio.google.com) for current model IDs, since Google renames and retires these periodically.

---

## Part 5: your usage and rate limits

Billed usage still has per-minute and per-day rate limits: paying doesn't mean unlimited. Google changes these limits periodically and they vary by model, so check your live limits in [AI Studio](https://aistudio.google.com) rather than relying on a number quoted here.

A typical session in this course uses roughly 20 to 50 requests, which costs a small fraction of a cent on the lightweight models we use. If you hit a rate limit, the API returns a 429 error. Wait for the reset, or check whether the account needs a higher tier.

Estimating token costs before running a job is covered in this week's cost guide.

---

## Part 6: save your work to GitHub

You forked the repository in Part 1. Save this notebook into that fork:

**File → Save a copy in GitHub**. Colab will ask to authorise access the first time. Then select `YOUR_USERNAME/AI4TM` (the fork you created in Part 1), keep the file path Colab suggests (`week_2/lesson08_setup_guide.ipynb`), write a commit message, and click OK.

Repeat this at the end of every session, into that same fork, at that same file path. This gives you version history you can roll back, a copy outside Colab, and over time a record of your applied AI work.

---

## Part 7: quick reference

The basic pattern for calling Gemini, and the shape most of your prompts in this course will take:

```python
from google import genai
from google.colab import userdata

# Connect to Gemini
client = genai.Client(api_key=userdata.get('GEMINI_API_KEY'))

# Ask something
response = client.models.generate_content(
    model='gemini-3.5-flash-lite',
    contents='Your question here'
)

print(response.text)
```

**Sentiment analysis on a customer review**, a classification task in the sense covered in [Lesson 6](https://community.teamsimmer.com/c/ai-for-technical-marketers/sections/1029997/lessons/4202494):

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
    model='gemini-3.5-flash-lite',
    contents=prompt
)

print(response.text)
```

**Generating marketing copy from a brief:**

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
    model='gemini-3.5-flash-lite',
    contents=prompt
)

print(response.text)
```

Both prompts describe the task without including worked examples, which makes them zero-shot ([Lesson 6](https://community.teamsimmer.com/c/ai-for-technical-marketers/sections/1029997/lessons/4202494)). Adding two or three labelled examples is the first thing to try when output is inconsistent.

---

## Setup complete

Colab is ready, your key is stored, the connection is tested, and your first notebook is saved to GitHub. Before each session: open that session's notebook, confirm the key is still in Secrets, and run the setup cells at the top.

Save to GitHub regularly rather than relying on Colab's autosave. Change the example code and rerun it, since reading it teaches less than breaking it. Keep an eye on your usage and rate limits. There's no free quota behind it anymore. Use only public or synthetic data.

---

## Optional: advanced setups

The rest applies only if you already hold a paid API key for OpenAI or Anthropic, or you'd rather run code on your own machine than in Colab.

### Optional A: using paid APIs

**OpenAI:**

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

Store the key in Secrets as `OPENAI_API_KEY`.

**Anthropic:**

```python
!pip install -q anthropic

from anthropic import Anthropic
from google.colab import userdata

client = Anthropic(api_key=userdata.get('ANTHROPIC_API_KEY'))

response = client.messages.create(
    model="claude-haiku-4-5",
    max_tokens=1024,
    messages=[{"role": "user", "content": "What is customer segmentation?"}]
)

print(response.content[0].text)
```

Store the key in Secrets as `ANTHROPIC_API_KEY`.

Model names change. If a call errors on the model ID, check the provider's current documentation.

### Optional B: local setup

You'll need to be comfortable with the command line and file paths. You do not need Python already installed, the tool below handles that itself.

This course uses [uv](https://docs.astral.sh/uv/) for local setup rather than a hand-built `venv`, since it creates each session's environment automatically, with nothing to create or activate by hand.

```bash
# Clone the fork you created in Part 1
git clone https://github.com/YOUR_USERNAME/AI4TM.git
cd AI4TM

# Install uv (one time)
curl -LsSf https://astral.sh/uv/install.sh | sh
# Windows (PowerShell): powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

From here, pick whichever of the two matches how you like to work.

**Recommended: a persistent environment, one per session, for VS Code, PyCharm, or any other IDE.** Each session folder gets its own `.venv`, created once from a terminal:

```bash
cd week_2
uv venv
uv pip install -r requirements.txt
```

`requirements.txt` already lists `ipykernel`, so this one command installs everything the session needs and registers the environment as a selectable Jupyter kernel, no separate step. Then open the notebook in your IDE and point its kernel/interpreter picker at what you just created: **VS Code** → Select Kernel (top right) → Python Environments → the `.venv` inside `week_2`. **PyCharm** → Settings → Project → Python Interpreter → Add Interpreter → Existing → `week_2/.venv/bin/python` (`week_2\.venv\Scripts\python.exe` on Windows). Any other IDE: point its interpreter/kernel picker at that same path.

Repeat `cd week_N && uv venv && uv pip install -r requirements.txt` for each session as you reach it, environments are deliberately per-session, not shared, so one session's packages never conflict with another's.

**Alternative: a disposable browser tab, like Colab, no IDE or kernel-picking involved.** Each time you want to work on a session:

```bash
cd week_2
uv run --with jupyter jupyter lab
```

This opens Jupyter Lab in your browser, with a fresh environment built just for that session, torn down again once you close it, nothing persists between runs the way the recommended option's `.venv` does.

### Storing API keys locally

To store an API key locally, create a `.env` file in the project folder:

```
GEMINI_API_KEY=your_key_here
```

Then load it:

```python
from dotenv import load_dotenv
import os

load_dotenv()
api_key = os.getenv('GEMINI_API_KEY')
```

`.env` is listed in `.gitignore`, so it won't be uploaded to GitHub.

---

## Resources

For help, post in the course space or bring it to office hours. Otherwise: the [Google Colab FAQ](https://research.google.com/colaboratory/faq.html), [GitHub Guides](https://guides.github.com), [Google AI Studio](https://aistudio.google.com), and the [Gemini API docs](https://ai.google.dev/gemini-api/docs). To check your key and usage, visit [your API keys](https://aistudio.google.com/app/apikey).
