# Estimating token costs: how not to bankrupt yourself

Before running an AI model on a large batch of data — 500 customer reviews, a full spreadsheet, a folder of documents — it's worth knowing roughly what that run will cost. This guide builds the habit of previewing a cost before spending anything, whether that spending is real money on a paid API or your daily allowance on the free Gemini tier.

**Time**: about 15 minutes
**Cost to read this guide**: $0

---

## Why this matters

On the free Gemini tier, the default for this course, there is no way to run up a credit card bill, because no card is attached to it. There is, however, a way to burn through a daily quota in minutes: looping over a large dataset without checking its size first, and then being locked out until the quota resets.

On a paid API, whether OpenAI, Anthropic, or Gemini's paid tier, every request costs real money, priced per **token** rather than per request. A single test prompt costs a fraction of a cent, so the risk isn't in careful, deliberate testing. It's in an unplanned loop: a script that re-sends a fifty-page document on every one of ten thousand iterations can accumulate real cost before anyone notices. The habit this guide builds — estimate first, run second — is what prevents that.

---

## What is a token?

AI models don't read text as words or letters. They read it as **tokens**, small chunks of text the model was trained to recognize. A token is often a word, though frequently it's only a piece of one; "marketing," for instance, might split into two tokens, `market` and `ing`. As a rough rule for English text, one token is approximately four characters, or about three-quarters of a word. A hundred-word email, by that rule, comes to roughly 130 to 150 tokens. This estimate isn't exact, but it's close enough to sanity-check a cost before committing to a run.

Tokens come in two kinds, priced differently. **Input tokens** cover everything sent to the model: the prompt, any pasted data, and any instructions. **Output tokens** cover everything the model generates in response, and these are typically priced two to six times higher than input tokens, since generating text costs more computationally than reading it.

For a batch job processing many items, whether rows in a spreadsheet, customer reviews, or emails, the total cost follows roughly this shape:

```
number of items × (input tokens per item + output tokens per item) × price per token
```

---

## A quick estimate, without calling any API

A rough token count doesn't require an API call at all. In Python:

```python
def estimate_tokens_quick(text: str) -> int:
    """Rough token estimate: ~4 characters per token for English text."""
    return len(text) // 4

sample_review = "Great product, but shipping took way too long. Would still recommend to a friend."
print(estimate_tokens_quick(sample_review))  # ~20 tokens
```

Run this the moment a loop over a dataset is being written, before the API is touched at all.

---

## Getting an exact count, for free

When a precise number is needed rather than a rough guess, most providers offer a **token counting** call that costs nothing and doesn't count against the usage quota, since it doesn't generate anything.

For Gemini:

```python
from google import genai
from google.colab import userdata

client = genai.Client(api_key=userdata.get('GEMINI_API_KEY'))

response = client.models.count_tokens(
    model='gemini-2.5-flash-lite',
    contents='Analyze this customer review for sentiment: "Great product, slow shipping."'
)
print(response.total_tokens)
```

OpenAI and Anthropic offer equivalent counting endpoints for anyone using a paid key with those providers. Method names shift occasionally as libraries update, so if the code above errors, a search for "[provider name] count tokens API" should surface the current syntax.

---

## Current pricing

Prices are quoted per one million tokens, and they shift over time as providers release new models. The table below is a starting point, not a fixed reference — check the official pricing page linked underneath before budgeting a large run.

| Provider | Model | Input $/1M tokens | Output $/1M tokens | Notes |
|---|---|---|---|---|
| Google Gemini | Gemini 2.5 Flash-Lite | $0.10 | $0.40 | Cheapest paid Gemini option |
| Google Gemini | Gemini 3.1 Flash-Lite | $0.25 | $1.50 | Budget-friendly, more capable |
| Google Gemini | Gemini 3.5 Flash | $1.50 | $9.00 | Most capable "Flash" tier |
| Google Gemini | Free tier | $0 | $0 | No dollar cost, but rate-limited (requests per minute and per day). Limits change; check your quota on [Google AI Studio](https://aistudio.google.com/apikey) |
| OpenAI | GPT-4o mini | $0.15 | $0.60 | Cheapest OpenAI option |
| Anthropic | Claude Haiku 4.5 | $1.00 | $5.00 | Cheapest Claude option |
| Anthropic | Claude Sonnet 5 | $3.00 | $15.00 | Mid-tier, higher quality |

Official pricing pages, worth bookmarking since they're always more current than a static table:

- Google Gemini: [ai.google.dev/gemini-api/docs/pricing](https://ai.google.dev/gemini-api/docs/pricing)
- OpenAI: [openai.com/api/pricing](https://openai.com/api/pricing/)
- Anthropic: [claude.com/pricing](https://claude.com/pricing)

> **A free tier is not an unlimited tier.** The free Gemini tier carries no dollar cost, but it does have a daily and per-minute request cap that Google adjusts over time. Anyone planning to process more than a handful of items should check the current quota on the AI Studio dashboard before starting, rather than trusting a fixed number from any guide, including this one.

---

## Building a cost calculator

Before running anything on a paid tier, the total cost is worth estimating directly:

```python
def estimate_cost(
    num_items: int,
    avg_input_chars: int,
    avg_output_tokens: int,
    price_per_1m_input: float,
    price_per_1m_output: float,
) -> float:
    """Estimate total USD cost for a batch job, before running it."""
    input_tokens_per_item = avg_input_chars / 4  # rough char-to-token estimate
    total_input_tokens = num_items * input_tokens_per_item
    total_output_tokens = num_items * avg_output_tokens

    input_cost = (total_input_tokens / 1_000_000) * price_per_1m_input
    output_cost = (total_output_tokens / 1_000_000) * price_per_1m_output
    return input_cost + output_cost

cost = estimate_cost(
    num_items=500,
    avg_input_chars=250,       # ~a 50-word review plus your prompt instructions
    avg_output_tokens=40,      # a short sentiment + confidence + key phrases response
    price_per_1m_input=0.15,   # GPT-4o mini
    price_per_1m_output=0.60,
)
print(f"Estimated cost: ${cost:.4f}")
```

---

## A worked example: 500 customer reviews

Consider running sentiment analysis on 500 customer reviews, each roughly 50 words (about 250 characters), with a short structured response (about 40 tokens) returned per review.

| Model | Estimated cost for 500 reviews |
|---|---|
| Gemini free tier | $0 (check the daily request quota first — 500 calls may exceed it) |
| Gemini 2.5 Flash-Lite | ~$0.011 |
| GPT-4o mini | ~$0.017 |
| Claude Haiku 4.5 | ~$0.131 |
| Claude Sonnet 5 | ~$0.394 |

Even the costliest option here comes in well under a dollar for 500 reviews. The real risk isn't a single, well-planned batch — it's an unplanned loop: retrying on every error without a cap, accidentally re-processing the same data twice, or pasting an entire multi-page document into every single request in a loop instead of once. Estimating the total before pressing run matters most precisely when the item count is large or the prompts carry a lot of pasted context.

---

## Rules to avoid surprise bills

Estimate before looping: multiply item count by tokens per item by price, before writing the `for` loop that calls the API. Test on the cheapest model first, prototyping a prompt on the free tier or the cheapest paid model (Flash-Lite, GPT-4o mini, Haiku), and only upgrading to a pricier model when quality genuinely requires it. Cap the output length: most APIs accept a `max_tokens` (or similarly named) parameter, and setting a sane ceiling means a model that starts rambling can't run up an unexpectedly large output bill.

Avoid re-sending large context repeatedly. If the same large document is referenced across many calls, summarize it once, or use the provider's caching feature, rather than pasting the whole document into every request. Finally, set a spending limit in the provider's dashboard. OpenAI, Anthropic, and Google Cloud all allow a hard monthly spending cap or a usage alert to be configured, and turning one on for any paid key provides a safety net for when an estimate turns out to be wrong.

---

## Where to watch real spending

- **Google Gemini**: [aistudio.google.com/apikey](https://aistudio.google.com/apikey) — shows key usage and quota
- **OpenAI**: [platform.openai.com/usage](https://platform.openai.com/usage) — usage dashboard and spending limits
- **Anthropic**: [console.anthropic.com](https://console.anthropic.com) — Console → Usage and Cost

These are worth checking before a large run, not only after.

---

## Quick reference

```python
# Rough token estimate — no API call needed
def estimate_tokens_quick(text: str) -> int:
    return len(text) // 4

# Cost estimate for a batch job — run this before your real loop
def estimate_cost(num_items, avg_input_chars, avg_output_tokens,
                   price_per_1m_input, price_per_1m_output):
    total_input_tokens = num_items * (avg_input_chars / 4)
    total_output_tokens = num_items * avg_output_tokens
    input_cost = (total_input_tokens / 1_000_000) * price_per_1m_input
    output_cost = (total_output_tokens / 1_000_000) * price_per_1m_output
    return input_cost + output_cost
```

The habit that matters most: before running anything against a real dataset, ask how many items, how many tokens each, and at what price — and do that multiplication before running the loop.

---

Questions belong in the Circle community.
