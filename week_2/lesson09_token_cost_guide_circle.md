# Hands-on: estimate token costs before you run anything

Lesson 3 covered what tokens are and why everything you pay for is counted in them. This guide turns that into a habit: before running a model over 500 customer reviews, a full spreadsheet, or a folder of documents, work out roughly what the run will cost. There's no free Gemini tier left to fall back on if that step gets skipped — every provider used in this course now requires billing before it issues a key.

Takes about 15 minutes.

---

## Why this matters

Every provider used in this course, Gemini included, now requires a billing method on file before it will issue a key. Every request costs real money, priced per token rather than per request. A single test prompt costs a fraction of a cent, so careful, deliberate testing is cheap. The expensive failure is an unplanned loop: a script that re-sends a fifty-page document on every one of ten thousand iterations can accumulate real cost before anyone notices, with no daily quota wall to stop it early the way a free tier used to. Estimate first, run second, and that doesn't happen.

---

## How pricing works

Tokens come in two kinds, priced differently.

**Input tokens** cover everything sent to the model: your prompt, any pasted data, any instructions. **Output tokens** cover everything the model generates back. Output is typically priced two to six times higher than input, since generating text costs more computationally than reading it.

For a batch job processing many items, whether rows in a spreadsheet, reviews, or emails, total cost follows this shape:

```
number of items × (input tokens per item + output tokens per item) × price per token
```

The asymmetry between input and output prices is worth holding onto when you design a prompt. A long instruction block sent across 50,000 rows makes input the dominant cost, while a prompt that asks for lengthy prose back shifts the weight to output, at several times the rate.

---

## A quick estimate, without calling any API

Lesson 3's rule of thumb, roughly four characters per token in English, is enough to sanity-check a run before you touch the API:

```python
def estimate_tokens_quick(text: str) -> int:
    """Rough token estimate: ~4 characters per token for English text."""
    return len(text) // 4

sample_review = "Great product, but shipping took way too long. Would still recommend to a friend."
print(estimate_tokens_quick(sample_review))  # ~20 tokens
```

Run this the moment you start writing a loop over a dataset, before the API is touched at all.

Two things make this estimate run low: non-English text fragments into more tokens than its English equivalent, and structured strings like SKUs, URLs, and IDs fragment badly. Both are covered in Lesson 3. If your data is heavy on either, treat the quick estimate as a floor and get an exact count.

---

## Getting an exact count, at no per-call charge

When you need a precise number, most providers offer a token counting call with no per-call charge, since it generates nothing — it still needs a valid, billed API key to run.

For Gemini:

```python
from google import genai
from google.colab import userdata

client = genai.Client(api_key=userdata.get('GEMINI_API_KEY'))

response = client.models.count_tokens(
    model='gemini-3.5-flash-lite',
    contents='Analyze this customer review for sentiment: "Great product, slow shipping."'
)
print(response.total_tokens)
```

OpenAI and Anthropic offer equivalent counting endpoints for paid keys. Method names shift as libraries update, so if the code above errors, search "[provider name] count tokens API" for the current syntax.

---

## Current pricing

Prices are quoted per one million tokens and shift as providers release and retire models. Treat the table below as a starting point and check the official pages before budgeting a large run.

| Provider | Model | Input $/1M | Output $/1M | Notes |
|---|---|---|---|---|
| Google Gemini | Gemini 3.5 Flash-Lite | $0.30 | $2.50 | Used in this course |
| Google Gemini | Gemini 3.1 Flash-Lite | $0.25 | $1.50 | Cheaper alternative, older generation |
| Google Gemini | Gemini 2.5 Flash-Lite | $0.10 | $0.40 | Cheapest option, but Google is retiring it (16 October 2026, not yet final) and it's already been unreliable ahead of that date |
| Google Gemini | Gemini 3.5 Flash | $1.50 | $9.00 | More capable, roughly 5x the Flash-Lite input price |
| OpenAI | GPT-4o mini | $0.15 | $0.60 | Budget tier, check the pricing page for current model lineup |
| Anthropic | Claude Haiku 4.5 | $1.00 | $5.00 | Cheapest current Claude |
| Anthropic | Claude Sonnet 5 | $2.00 | $10.00 | Mid-tier. Originally introductory through 31 August 2026 with a planned rise to $3.00/$15.00 after; Anthropic cancelled that increase and made $2.00/$10.00 the standing rate |

Official pricing pages, always more current than a static table:

- Google Gemini: [ai.google.dev/gemini-api/docs/pricing](https://ai.google.dev/gemini-api/docs/pricing)
- OpenAI: [openai.com/api/pricing](https://openai.com/api/pricing/)
- Anthropic: [claude.com/pricing](https://claude.com/pricing)

> **Model retirement affects this course.** The course notebooks call `gemini-3.5-flash-lite`. They used to call `gemini-2.5-flash-lite`, but that model started erroring out for some students well ahead of Google's own retirement date (16 October 2026, and not yet final), so the notebooks were moved off it early. If `gemini-3.5-flash-lite` ever starts failing on the model name for you, check [AI Studio](https://aistudio.google.com) for the current lineup; `gemini-3.1-flash-lite` is Google's official migration target for 2.5 and costs noticeably less than 3.5 on both input and output, so it's worth trying first if 3.5 is also unavailable or you want to cut cost.

> **There is no free tier left.** Google removed Pro models from the free tier in April 2026, leaving only Flash and Flash-Lite free — and since then has moved to requiring a billing method on file before issuing any Gemini API key at all, closing that remaining free path too. Billed usage still carries a daily and per-minute rate limit that Google adjusts over time; before processing more than a handful of items, check the current limits on the AI Studio dashboard rather than trusting a fixed number from any guide, this one included.

---

## Building a cost calculator

Before running anything on a paid tier, estimate the total directly:

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

Sentiment analysis on 500 reviews, each roughly 50 words (about 250 characters), with a short structured response of about 40 tokens per review.

| Model | Estimated cost for 500 reviews |
|---|---|
| Gemini 3.5 Flash-Lite | ~$0.059 |
| Gemini 3.1 Flash-Lite | ~$0.038 |
| Gemini 2.5 Flash-Lite | ~$0.011 |
| GPT-4o mini | ~$0.017 |
| Claude Haiku 4.5 | ~$0.131 |
| Claude Sonnet 5 | ~$0.263 |

Even the costliest option lands well under a dollar for 500 reviews. A single well-planned batch is rarely the problem. Costs run away in the unplanned cases: retrying on every error without a cap, re-processing the same data twice, or pasting a multi-page document into every request in a loop rather than once. Estimating the total before pressing run matters most when the item count is large or the prompts carry a lot of pasted context.

---

## Rules to avoid surprise bills

**Estimate before looping.** Multiply item count by tokens per item by price, before writing the `for` loop that calls the API.

**Test on the cheapest model first.** Prototype a prompt on the cheapest available model, then upgrade only when quality genuinely requires it — none of these models are free anymore, but the cheapest ones cost little enough to iterate on freely.

**Cap the output length.** Most APIs accept a `max_tokens` parameter or similar. Setting a sane ceiling means a model that starts rambling can't run up an unexpected output bill, and output is the expensive side.

**Avoid re-sending large context.** If the same large document is referenced across many calls, summarise it once or use the provider's caching feature rather than pasting it into every request. This is the single most common cause of a runaway bill, and it connects to the context window point in Lesson 5: stuffing everything in costs more and often works worse.

**Set a spending limit in the provider's dashboard.** OpenAI, Anthropic, and Google Cloud all allow a hard monthly cap or a usage alert. Turn one on for any paid key, as a safety net for when an estimate turns out wrong.

---

## Where to watch real spending

- **Google Gemini**: [aistudio.google.com/apikey](https://aistudio.google.com/apikey), shows key usage and quota
- **OpenAI**: [platform.openai.com/usage](https://platform.openai.com/usage), usage dashboard and spending limits
- **Anthropic**: [console.anthropic.com](https://console.anthropic.com), Console then Usage and Cost

Check these before a large run, not only after.

---

## Quick reference

```python
# Rough token estimate, no API call needed
def estimate_tokens_quick(text: str) -> int:
    return len(text) // 4

# Cost estimate for a batch job, run this before your real loop
def estimate_cost(num_items, avg_input_chars, avg_output_tokens,
                  price_per_1m_input, price_per_1m_output):
    total_input_tokens = num_items * (avg_input_chars / 4)
    total_output_tokens = num_items * avg_output_tokens
    input_cost = (total_input_tokens / 1_000_000) * price_per_1m_input
    output_cost = (total_output_tokens / 1_000_000) * price_per_1m_output
    return input_cost + output_cost
```

Before running anything against a real dataset: how many items, how many tokens each, at what price. Do that multiplication before the loop, not after.

Questions belong in the course space or office hours.
