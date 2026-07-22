# Hands-on: set up your AI assistant

Setup takes about ten minutes and costs nothing: no credit card, no subscription, nothing to cancel later. You need a Google account, and the GitHub account from Week 1.

The course defaults to Google's free tier for Gemini, which has a daily allowance that resets automatically and a key that doesn't expire. One Google login covers Colab and Gemini both. If you already hold a paid API key for OpenAI or Anthropic, you can use that instead. See the optional section at the end.

> **A note on privacy.** Free AI APIs, Google's included, may have requests reviewed by human moderators, and your data may be used to train future models. For this course, work only with public example data, synthetic data, or the anonymised datasets in the materials. Don't send client data, confidential information, or proprietary content through a free-tier API. A paid plan removes this exposure, though it isn't required here. Week 4 covers this properly.

---

## Part 1: open notebooks in Colab

To open a course notebook:

1. Go to the course GitHub repository (link provided by your instructor).
2. Click any `.ipynb` file, such as `setup_guide.ipynb`.
3. Click the "Open in Colab" badge at the top of the file.

Notebooks are built from **code cells**, which have a play button and run when clicked, and **text cells**, which are there to be read. You'll mostly run code cells in order, top to bottom.

To save: **File → Save a copy in Drive**, or **File → Save a copy in GitHub** to commit straight to your repository (Part 5).

---

## Part 2: get your free Gemini API key

Your API key is what authenticates your code as you. Anyone who has it can spend against your account, so treat it accordingly.

1. Go to [aistudio.google.com/apikey](https://aistudio.google.com/apikey).
2. Sign in with your Google account.
3. Click "Create API Key."
4. Let Google create a new project for you (click "Continue" when prompted).
5. Copy the key. It starts with `AIza`.

> **If you're in the EEA, UK, or Switzerland:** Google may ask you to enable billing for regulatory reasons before issuing a key. You won't be charged while you stay within the free quota.

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

## Part 3: test your setup

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
    model='gemini-2.5-flash-lite',
    contents='In one sentence, what is customer segmentation?'
)

print(response.text)
```

If you get an error, check that the key is in Secrets, that it's named exactly `GEMINI_API_KEY`, and that the access toggle is on. If the error mentions the model name, check [AI Studio](https://aistudio.google.com) for current model IDs, since Google renames and retires these periodically.

---

## Part 4: your free quota

The free tier caps requests per minute and per day. Google changes these limits periodically and they vary by model, so check your live quota in [AI Studio](https://aistudio.google.com) rather than relying on a number quoted here.

A typical session in this course uses roughly 20 to 50 requests, well inside the free allowance for the lightweight models we use. If you hit the limit, the API returns a 429 error. Wait for the reset, or move to a paid plan if you need more headroom.

Estimating token costs before running a job is covered in this week's cost guide.

---

## Part 5: save your work to GitHub

Create a repository to hold your course work:

1. On GitHub, click the "+" icon in the top right.
2. Select "New repository."
3. Name it `ai4tm-masterclass`.
4. Add a description, such as "AI for Marketing projects."
5. Choose Public or Private.
6. Check "Add a README file."
7. Under "Add .gitignore," choose "Python."
8. Click "Create repository."

To save a Colab notebook into it: **File → Save a copy in GitHub**. Colab will ask to authorise access the first time. Then select `your-username/ai4tm-masterclass`, set the file path to `session_1/setup_guide.ipynb`, write a commit message, and click OK.

This gives you version history you can roll back, a copy outside Colab, and over time a record of your applied AI work.

---

## Part 6: quick reference

The basic pattern for calling Gemini, and the shape most of your prompts in this course will take:

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

**Sentiment analysis on a customer review**, a classification task in the sense covered in Lesson 6:

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
    model='gemini-2.5-flash-lite',
    contents=prompt
)

print(response.text)
```

Both prompts describe the task without including worked examples, which makes them zero-shot (Lesson 6). Adding two or three labelled examples is the first thing to try when output is inconsistent.

---

## Setup complete

Colab is ready, your key is stored, the connection is tested, and your first notebook is saved to GitHub. Before each session: open that session's notebook, confirm the key is still in Secrets, and run the setup cells at the top.

Save to GitHub regularly rather than relying on Colab's autosave. Change the example code and rerun it, since reading it teaches less than breaking it. Keep an eye on your daily quota. Use only public or synthetic data.

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
    model="claude-3-5-haiku-20241022",
    max_tokens=1024,
    messages=[{"role": "user", "content": "What is customer segmentation?"}]
)

print(response.content[0].text)
```

Store the key in Secrets as `ANTHROPIC_API_KEY`.

Model names change. If a call errors on the model ID, check the provider's current documentation.

### Optional B: local setup

You'll need Python 3.8 or later, the command line, and a working understanding of file paths.

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
