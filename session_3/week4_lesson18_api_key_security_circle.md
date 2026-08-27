# Setting up and protecting your API keys

This is the text companion to the API key security notebook. Session 1 showed how to store your Gemini key in Colab Secrets instead of typing it into code. This piece turns that into a full set of guidelines: how to set a key up securely in the first place, the specific ways it gets out even when everything was done "right," and what to do if one leaks.

**Time**: about 15 minutes
**Cost**: effectively $0 — nothing here calls a metered API beyond what Session 1 already set up, and there's no free quota behind it anymore, so keep usage light

> **Disclaimer.** This is educational guidance, not professional security or legal advice. Following it reduces risk but can't eliminate it. Each learner is responsible for their own API keys, billing, and account security. The course and its creators are not liable for costs, data exposure, or other consequences resulting from a leaked, misused, or mishandled key — if one leaks, act on "If a key leaks anyway" below immediately.

---

## Why this matters

An API key is a password. Anyone who has it can spend against the associated account, and on GitHub, "anyone" includes automated bots built specifically to find keys the moment they go public.

GitHub scans every public push for text that matches the shape of a known key format, and for over a hundred providers, including Google Cloud, OpenAI, and Anthropic, it reports a match straight to that provider, who revokes it automatically, often before anyone would notice anything was wrong. That's the good outcome. The bad outcome is what happens with everything else: independent scanners run the same search, and they aren't trying to help.

Two documented cases show the timeline. In one widely cited incident, a developer accidentally committed AWS credentials to a public repo; attackers found and used them in about six minutes. In another, bots used leaked AWS credentials to spin up over 500 cloud instances for cryptocurrency mining within twenty minutes — by the time the owner noticed, the bill was $72,000. Nothing about this course's usage is immune to the same pattern; a leaked key just gets someone cut off, or billed, faster than they can react.

---

## Setup guidelines

1. **Get the key with billing enabled.** As of 2026, Google requires a billing method on file before it will issue a Gemini key at all — see Session 1's setup guide for the walkthrough. The lightweight models this course uses cost fractions of a cent per request, but there's no free quota to fall back on, so watch usage from the start.
2. **Store it in Colab Secrets, never in code.** The key icon in Colab's left sidebar, not a line of Python. Session 1 covers the exact steps.
3. **Working locally instead of Colab?** Use a `.env` file in the project folder:

   ```
   GEMINI_API_KEY=your_key_here
   ```

   loaded in Python with:

   ```python
   from dotenv import load_dotenv
   import os

   load_dotenv()
   api_key = os.getenv('GEMINI_API_KEY')
   ```

4. **Confirm `.env` is in `.gitignore`.** This is what actually keeps the key out of git, not the `.env` file by itself — `.gitignore` tells git which files to never track or commit, even when a broad `git add` is run. Repositories created the way Session 1 describes, checking "Add .gitignore" and choosing "Python," already have this handled. The course notebook includes a small check that reads a `.gitignore` file and confirms `.env` is covered.
5. **Set a spending limit or usage alert** in the provider's dashboard (AI Studio, OpenAI, Anthropic all offer this). It's the safety net for when every other guideline here fails.

---

## Protection guidelines: three ways a key gets out, even with Secrets in place

**Never type it directly into a code cell.** `api_key = "AIzaSy..."` works exactly as well as Colab Secrets, right up until the notebook is saved to GitHub — at which point the key sits in plain text in a public file, permanently, in the commit history, even after the line is deleted in a later commit. *Do instead*: load it from Secrets or `.env`, always.

**Never print it in full.** A notebook file saves its cell outputs along with the code, not just the code that was written, so if a cell prints the whole key, that value gets written into the `.ipynb` file itself and pushed with it. This is true even when the key was loaded safely from Secrets. *Do instead*: print a truncated preview, the way Session 1's setup notebook does — `api_key[:10] + "..."` — any time printing something that came from Secrets or an `.env` file.

**Never paste it into an AI chat unredacted.** Debugging often means pasting an error message, a config file, or a `.env` into a chatbot to ask what's wrong, and it is easy for a real key to be sitting inside that paste unnoticed, especially in a traceback that happens to print variable values. Google's terms say Gemini usage billed under a "Paid tier" project isn't used for training and skips human review — check the badge in AI Studio to confirm a given key qualifies — but that protection doesn't necessarily extend to every tool or provider pasted into, and a key pasted into a chat never touches GitHub, so none of the scanning described above catches it either. *Do instead*: scan a paste once for anything starting with a provider's key prefix (`AIza`, `sk-`, and similar) and replace it with `[REDACTED]` before sending it.

---

## What a scanner is actually looking for

GitHub's scanner, and every bot copying its approach, isn't reading code for meaning; it's pattern-matching on shape. A Gemini key starts with `AIza` followed by a long run of letters, digits, hyphens, and underscores; other providers have their own fixed prefixes. The course notebook builds a small version of that same check with a regular expression, run over a handful of example lines, some safe and some not, to make the abstract idea concrete.

> **A note on examples**: any example key shown in the notebook or here is invented and will not work against a real API. Never paste a real key into a notebook, a chat, or this document, even as a "just checking" test.

---

## If a key leaks anyway

1. **Revoke it first, investigate later.** Go to [AI Studio](https://aistudio.google.com/app/apikey), or the relevant provider console, and delete the key immediately. Every minute it stays active is a minute a bot can use it.
2. **Don't rely on removing it from git history.** Force-pushing a fix that deletes the key from an old commit does not undo a leak — if the repository was ever public, the key may already be scraped and stored elsewhere, independent of the repository's current state. Revocation, not cleanup, is what actually stops the exposure.
3. **Generate a new key and update it everywhere it's used** — Colab Secrets, any local `.env` files, anywhere else it was referenced.
4. **Check for damage.** Look at the usage or billing dashboard for the affected account for requests that weren't made intentionally, and raise any unexpected charges with the provider directly.

There's no free quota to soften the worst case anymore — a leaked key is billed from the first unauthorized request. That's exactly why revoking fast matters more now than it used to, and why the disclaimer at the top of this document means what it says: each learner is responsible for watching their own key; this guide can only say how.

---

## Wrap-up

Setup guidelines get a key stored safely in the first place — Secrets or `.env` plus `.gitignore`, billing enabled from the start. Protection guidelines keep it safe afterward: never hardcoded, never printed in full, never pasted into a chat unredacted. Neither one is a setting switched on once — both are habits that need the same attention every session.

**Next in Session 3**: identifying where personal data, not just API keys, can leak through an AI pipeline, and applying the same kind of check to that risk.

Questions belong in the Circle community.
