# Setting up your free AI marketing toolkit

This guide sets up everything the course requires, at no cost: no credit card, no subscription, nothing to cancel later. By the end of it, three things will be in place — Google Colab for running code in a browser, a free Gemini API key for talking to Google's AI models, and a GitHub account for saving and sharing your work. The whole process takes about ten minutes, and the only requirement is a Google account, the same one behind Gmail.

## Your free learning path

This course defaults to Google's free tier for Gemini: no credit card, a daily allowance of AI requests that resets automatically, and no expiration date on the key itself. The same Google login covers Colab, Gemini, and everything else you'll touch in this course. If you already hold a paid API key for OpenAI or Anthropic, you can use that instead — see the optional section near the end of this guide.

> **A note on privacy.** Any free AI API, including Google's, OpenAI's free tier, and similar offerings, may have your requests reviewed by human moderators, and your data may be used to train future models. For this course, work only with public example data, made-up (synthetic) data, or anonymized datasets provided in the materials. Do not send client data, confidential information, or proprietary content through a free-tier API. If you need full privacy for sensitive work, a paid API plan removes this exposure, though it is not required for this course.

---

## Part 1: open notebooks in Google Colab

**Google Colab** is a free tool from Google for writing and running code directly in a browser. It behaves something like a word processor for code: nothing to install, work saves automatically, and a link is enough to share it with someone else.

To open a course notebook:

1. Go to the course GitHub repository (the link is provided by your instructor).
2. Click on any `.ipynb` file, such as `setup_guide.ipynb`.
3. Click the "Open in Colab" badge at the top of the file.
4. The notebook opens in a new browser tab.

A notebook in Colab is built from two kinds of cells. Code cells carry a play button and run when you click it; text cells hold explanations and are meant only for reading. To save your work, use File → Save a copy in Drive, or File → Save a copy in GitHub if you'd rather commit it to your repository directly.

---

## Part 2: get your free Gemini API key

An **API key** functions like a password: it is what lets your code authenticate with Google's AI models on your behalf. It takes the form of a long string starting with `AIza`.

To generate a free key:

1. Go to [aistudio.google.com/apikey](https://aistudio.google.com/apikey).
2. Sign in with your Google account.
3. Click "Create API Key."
4. Let Google create a new project for you (click "Continue" when prompted).
5. Copy the key once it appears.

The whole process takes about two minutes.

> **If you're in the EEA, UK, or Switzerland:** Google may ask you to enable billing for regulatory reasons before issuing a key. You will not be charged as long as you stay within the free quota.

**Do not paste an API key directly into your code.** Anyone who sees that code, including a shared notebook or a public GitHub repository, would then have your key. Colab Secrets exists specifically to avoid this:

1. In Colab, open the left sidebar and click the key icon, labeled "Secrets."
2. Click "+ Add new secret."
3. Set the name to `GEMINI_API_KEY`.
4. Paste your key as the value.
5. Toggle the switch to grant this notebook access.
6. Click "Save."

---

## Part 3: test your setup

**Google's AI library** is a package of pre-written code that handles the details of talking to Gemini, so you do not have to build that connection from scratch. Install it by running the following in any code cell:

```python
!pip install -q google-genai
```

The leading `!` tells Colab to run this line as a system command rather than Python code; `-q` asks for quiet output, suppressing the installation log.

Test the connection:

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

A response from Gemini means the setup worked. If you see an error instead, check three things: that the key was added to Secrets, that it is named exactly `GEMINI_API_KEY`, and that the toggle granting notebook access was switched on.

---

## Part 4: understand your free quota

The free tier includes ten requests per minute and 250 requests per day, with the daily count resetting every 24 hours and the key itself never expiring. A typical session in this course uses somewhere between 20 and 50 requests, so even with experimentation, the limit is unlikely to be a problem. If you do hit it, the API returns an error; wait until the quota resets the next day, or move to a paid plan if you need more headroom sooner.

---

## Part 5: GitHub — save your work

**GitHub** serves a role for code similar to what cloud storage does for documents: it keeps a history of versions so mistakes can be undone, it makes sharing a project straightforward, and over time it becomes a visible record of the work you've done, useful if you're building a portfolio around applied AI skills.

To create an account, go to [github.com](https://github.com), click "Sign up," choose a username (something like `yourname-marketing` reads well professionally), and verify your email. This takes about three minutes.

A **repository** (usually shortened to "repo") is the folder that holds a project's files. To create one for this course:

1. On GitHub, click the "+" icon in the top right.
2. Select "New repository."
3. Name it `ai4tm-masterclass`.
4. Add a description, such as "AI for Marketing projects."
5. Choose Public or Private, depending on your preference.
6. Check "Add a README file."
7. Under "Add .gitignore," choose "Python."
8. Click "Create repository."

To save a Colab notebook into that repository: in Colab, go to File → Save a copy in GitHub. The first time you do this, Colab will ask to authorize access to your GitHub account. Select the repository (`your-username/ai4tm-masterclass`), set the file path to `session_1/setup_guide.ipynb`, write a commit message such as "Complete setup guide," and click OK. From that point on, the notebook is saved and backed up outside of Colab.

---

## Part 6: quick reference

The pattern below is the basic template for calling Gemini, and it's the shape most of your prompts in this course will follow:

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

Two worked examples show the pattern applied to marketing tasks. The first analyzes sentiment in a customer review:

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

The second generates marketing copy from a short brief:

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

## Setup complete

At this point, Google Colab is set up, a free Gemini API key is generated and stored securely, the connection has been tested, a GitHub account exists, and a first notebook has been saved. Before each session going forward, open that session's notebook in Colab, confirm the API key is still present in Secrets, and run the setup cells at the top before continuing.

A few habits will keep the rest of the course running smoothly: save your work to GitHub regularly rather than relying on Colab's autosave alone, experiment by changing the example code rather than only reading it, keep an eye on your daily quota (250 requests is generous, but not unlimited), and use only public or synthetic data, never real client or confidential information.

---

## Optional: advanced setups

Everything below this point is optional. It's worth reading only if you already hold a paid API key for OpenAI or Anthropic, if you'd prefer to run code on your own machine rather than in Colab, or if you have prior experience and want more control over your environment.

---

### Optional A: using paid APIs

This section applies only if you already have an API key from OpenAI or Anthropic.

**OpenAI (ChatGPT):**

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

Store the OpenAI key in Secrets under the name `OPENAI_API_KEY`.

**Anthropic (Claude):**

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

Store the Anthropic key in Secrets under the name `ANTHROPIC_API_KEY`.

---

### Optional B: local setup (advanced)

This path is for experienced users who prefer running code on their own machine instead of in Colab. Most learners in this course should stay with Colab.

Before starting, you'll need Python 3.8 or later installed, familiarity with the command line, and a working understanding of file paths.

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

Then load it in code:

```python
from dotenv import load_dotenv
import os

load_dotenv()
api_key = os.getenv('GEMINI_API_KEY')
```

The `.env` file is already listed in `.gitignore`, so it will not be uploaded to GitHub.

---

## Resources

For help, the course's Circle community is the first stop, alongside the [Google Colab FAQ](https://research.google.com/colaboratory/faq.html) and [GitHub Guides](https://guides.github.com). For documentation, see [Google AI Studio](https://aistudio.google.com) and the [Gemini API docs](https://ai.google.dev/gemini-api/docs). To check your key and usage at any point, visit [Your API Keys](https://aistudio.google.com/app/apikey).

---

With setup complete, the next step is the neural networks notebook.
