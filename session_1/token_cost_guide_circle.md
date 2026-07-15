# Estimating Token Costs: How Not to Bankrupt Yourself

Before you run an AI model on a big batch of data — 500 customer reviews, a whole spreadsheet, a folder of documents — you should know roughly what it's going to cost. This guide teaches you to **preview the cost before you spend anything**, whether that's real dollars on a paid API or your daily quota on the free Gemini tier.

**Time**: ~15 minutes
**Cost to read this guide**: $0

---

## Why This Matters

On the **free Gemini tier** (the default for this course), you can't run up a credit card bill — there's no card attached. But you *can* burn through your daily quota in minutes if you loop over a large dataset without checking the size first, and then you're locked out until it resets.

On a **paid API** (OpenAI, Anthropic, or Gemini's paid tier), every request costs real money — priced per **token**, not per request. A single test prompt costs a fraction of a cent. A loop that accidentally re-sends a 50-page document on every one of 10,000 iterations can cost real money before you notice. The habit this guide builds — **estimate first, run second** — is what prevents that.

---

## What Is a Token?

AI models don't read text as words or letters — they read it as **tokens**, small chunks of text the model was trained to recognize. A token is often a word, but frequently it's a piece of a word.

For example, "marketing" might become two tokens: `market` + `ing`. As a rough rule for English text:

- **1 token ≈ 4 characters**
- **1 token ≈ ¾ of a word**

So a 100-word email is roughly 130–150 tokens. This isn't exact, but it's close enough to sanity-check a cost before you commit to running something.

**Two kinds of tokens, priced differently:**

- **Input tokens** — everything you send: your prompt, any pasted data, instructions.
- **Output tokens** — everything the model generates back. These are usually priced **2–6× higher** than input tokens, because generating text costs more than reading it.

If you're processing many items (rows in a spreadsheet, reviews, emails), your total cost is roughly:

```
number of items × (input tokens per item + output tokens per item) × price per token
```

---

## Quick Estimate — No API Call Needed

You don't need to call any API to get a ballpark. In Python:

```python
def estimate_tokens_quick(text: str) -> int:
    """Rough token estimate: ~4 characters per token for English text."""
    return len(text) // 4

sample_review = "Great product, but shipping took way too long. Would still recommend to a friend."
print(estimate_tokens_quick(sample_review))  # ~20 tokens
```

Use this the moment you're about to write a loop over a dataset — before you touch the API at all.

---

## Getting an Exact Count (Free — Doesn't Use Your Quota)

When you want a precise number instead of a rough guess, most providers offer a **token counting** call that's free and doesn't count against your usage quota, because it doesn't generate anything.

**Gemini:**

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

OpenAI and Anthropic have equivalent counting endpoints — check their docs if you're using a paid key. The exact method names change occasionally, so if the code above errors, search "[provider name] count tokens API" for the current version.

---

## Current Pricing (verify before you commit — these change often)

Prices are quoted **per 1 million tokens** and shift over time as providers release new models. Treat this table as a starting point, not gospel — check the official pricing page linked below before budgeting a large run.

| Provider | Model | Input $/1M tokens | Output $/1M tokens | Notes |
|---|---|---|---|---|
| Google Gemini | Gemini 2.5 Flash-Lite | $0.10 | $0.40 | Cheapest paid Gemini option |
| Google Gemini | Gemini 3.1 Flash-Lite | $0.25 | $1.50 | Budget-friendly, more capable |
| Google Gemini | Gemini 3.5 Flash | $1.50 | $9.00 | Most capable "Flash" tier |
| Google Gemini | **Free tier** | $0 | $0 | No dollar cost — but rate-limited (requests/minute and requests/day). Limits change; check your quota on [Google AI Studio](https://aistudio.google.com/apikey) |
| OpenAI | GPT-4o mini | $0.15 | $0.60 | Cheapest OpenAI option |
| Anthropic | Claude Haiku 4.5 | $1.00 | $5.00 | Cheapest Claude option |
| Anthropic | Claude Sonnet 5 | $3.00 | $15.00 | Mid-tier, higher quality |

**Official pricing pages** (bookmark these — they're always more current than any table):
- Google Gemini: [ai.google.dev/gemini-api/docs/pricing](https://ai.google.dev/gemini-api/docs/pricing)
- OpenAI: [openai.com/api/pricing](https://openai.com/api/pricing/)
- Anthropic: [claude.com/pricing](https://claude.com/pricing)

⚠️ **Free tier ≠ unlimited.** The free Gemini tier has no dollar cost, but it does have a daily and per-minute request cap that Google adjusts over time. If you're planning to process more than a handful of items, check your current quota on the AI Studio dashboard before you start — don't assume a number from an old guide (including this one) is still accurate.

---

## Build Your Own Cost Calculator

Before running anything on a paid tier, estimate the total:

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

## Worked Example: 500 Customer Reviews

Say you want to run sentiment analysis on 500 customer reviews, each about 50 words (~250 characters), with a short structured response (~40 tokens) per review.

| Model | Estimated cost for 500 reviews |
|---|---|
| Gemini free tier | $0 (but check your daily request quota first — 500 calls may exceed it) |
| Gemini 2.5 Flash-Lite | ~$0.011 |
| GPT-4o mini | ~$0.017 |
| Claude Haiku 4.5 | ~$0.131 |
| Claude Sonnet 5 | ~$0.394 |

Even the "expensive" option here costs well under a dollar for 500 reviews. The real risk isn't a single well-planned batch — it's an **unplanned loop**: retrying on every error without a cap, accidentally re-processing the same data twice, or pasting an entire multi-page document into every single request in a loop instead of once. Estimate the total *before* you press run, especially when `num_items` is large or your prompts include a lot of pasted context.

---

## 5 Rules to Avoid Surprise Bills

1. **Estimate before you loop.** Multiply item count × tokens per item × price, *before* writing the `for` loop that calls the API.
2. **Test on the cheapest model first.** Prototype your prompt on the free tier or the cheapest paid model (Flash-Lite, GPT-4o mini, Haiku). Only upgrade to a pricier model if quality genuinely requires it.
3. **Cap your output length.** Most APIs accept a `max_tokens` (or similar) parameter. Set a sane ceiling so a model that starts rambling can't run up an unexpectedly large output bill.
4. **Don't re-send large context repeatedly.** If you're referencing the same big document across many calls, summarize it once or use the provider's caching feature instead of pasting the whole thing into every request.
5. **Set a spending limit in your provider dashboard.** OpenAI, Anthropic, and Google Cloud all let you set a hard monthly spending cap or usage alert. Turn one on if you're using a paid key — it's the safety net for when your estimate is wrong.

---

## Where to Watch Your Real Spending

- **Google Gemini**: [aistudio.google.com/apikey](https://aistudio.google.com/apikey) — shows your key usage and quota
- **OpenAI**: [platform.openai.com/usage](https://platform.openai.com/usage) — usage dashboard and spending limits
- **Anthropic**: [console.anthropic.com](https://console.anthropic.com) — Console → Usage and Cost

Check these *before* a large run, not just after.

---

## Quick Reference

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

**The habit that matters most**: before running anything against a real dataset, ask "how many items, how many tokens each, at what price?" — and do that multiplication before you do the loop.

---

**Questions?** Post in the Circle community.
