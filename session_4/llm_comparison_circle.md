# Work with an LLM

*Comparing this pipeline against just prompting an LLM for synthetic data*

Everything else this session built took a deliberate path to a synthetic table: a real dataset, checked and typed, then a generative model trained and auto-tuned specifically on it. A marketing team also has a faster-looking shortcut available: describing the table to a large language model and asking it to produce one directly, no preprocessing tool, no tuned search. This piece is where that shortcut gets tested against the same task, rather than assumed to be worse, or good enough, without checking.

**Time**: about 15 to 20 minutes, once a Gemini API key is set up
**Cost**: a small number of Gemini API calls, at Flash-Lite's pay-as-you-go rate, fractions of a cent per request. Confirm current pricing at [aistudio.google.com](https://aistudio.google.com) before running

---

## Why send a description instead of the real table

CTGAN and TVAE never send the real customer table anywhere. Training happens locally or on Colab's own runtime, and the real data stays there the whole time. Prompting an LLM breaks that pattern: whatever goes into the prompt travels to a third party's API, so it's worth being deliberate about what actually goes in.

The companion notebook sends a description of the table, not the table itself: column names, what each one means, and summary statistics computed from the training split, category proportions, numeric ranges and averages. No individual customer's row is ever included in a prompt. It's the same habit as reading a preprocessing log before sending it, from earlier this session: check what's actually leaving before it leaves, not after.

---

## Why the table comes back in pieces, not all at once

CTGAN and TVAE spend their time up front, during training, and then generate rows almost instantly afterward: once a model is trained, asking it for a full-size table is a single, fast step. Prompting an LLM works differently. Each request has a practical limit to how many rows it can be asked to generate reliably in one pass, so the notebook builds its synthetic table across several smaller requests instead of one large one, stopping once it reaches roughly the same size as the real training split.

That's a real, structural difference worth weighing alongside whatever the accuracy numbers turn out to be. A model that trains once and then samples new rows for free, as many times as needed, is a different tool than one that costs a little more, in time and in requests, every time a larger table is needed.

---

## What this comparison can and cannot tell you

The companion notebook trains the same propensity model twice, once on real data and once on the LLM-generated table, and tests both against the same real, held-out customers this session has used throughout, the identical split CTGAN and TVAE were already tested against. Read the result the same way this session has read every other result: accuracy first, then set it aside, since a model that predicts "no" for every customer already scores misleadingly well on a table this imbalanced. Precision, recall, and F1 are what actually show whether the LLM-generated table taught the model anything real.

Whatever that result turns out to be on a given run, it says something narrow: a Gemini Flash-Lite model, prompted this way, with this particular summary of the data, produced rows that did or didn't transfer to real customers, for this one target column. It says nothing about a different target, a different prompt, or a different model, the same boundary this session has named for CTGAN and TVAE, just for a different method.

One difference is worth naming regardless of how the accuracy numbers land. CTGAN and TVAE train directly on the real table's actual joint distribution, learning how columns move together from the data itself. An LLM prompted this way works from a written summary that cannot fully capture how columns relate to each other, filled in by whatever the model already knows about e-commerce data in general. That difference, not just which approach happened to score higher on one run, is the more durable reason to expect one method or the other to fit a given real task.

---

## Wrap-up

The same test this session has applied to every synthetic table, real accuracy first set aside, then precision, recall, and F1 against the same held-out customers, applies here too, to a genuinely different way of getting a synthetic table in the first place. Run it, read the result the way this session has taught, and weigh it against the cost and scale trade-off named above, not the accuracy numbers alone.

**Also in Session 4**: when to use synthetic data at all, how a synthetic dataset gets made with a dedicated platform, and what its metrics do and don't protect.

Questions belong in the Circle community.
