# Train and tune the model

*Training and auto-tuning a synthetic data model*

The preprocessing piece left a clean, correctly typed customer table behind. This piece is where that table actually becomes synthetic: training CTGAN and TVAE on it, and using an automatic search instead of guesswork to decide how each one trains, so the synthetic table this pipeline depends on is one actually worth using.

**Time**: about 25 minutes, most of it spent waiting on training rather than writing code
**Cost**: $0. Everything here runs on Colab's own CPU/GPU, no API key involved

---

## Why a search instead of hand-tuning

Training a generative model adjusts millions of internal values automatically, the model handles that part on its own. What it can't decide on its own is a much smaller set of choices that have to be made before training even starts, called **hyperparameters**: how many rounds to run, how large a step the model takes each time it corrects itself, how many layers deep its networks are. Choosing these by hand means guessing, then waiting to see whether the guess worked. `SyntheticDataModelTuner` replaces that guessing with a search: each trial samples a combination of hyperparameters, trains a full model with them, scores the result, and keeps a record. After the requested number of trials, the search hands back whichever combination scored best, and the evaluation behind that choice.

This notebook scores each trial on one metric from each of the four families this week's earlier lesson described: sanity, statistical fidelity, suitability, and privacy. All four are defined so that higher is always better, which is what lets the search judge every trial by one combined score instead of four separate ones. Each trial is a full, independent training run, and the platform's own documentation warns this gets slow. The notebook's own default is deliberately small, just two trials, so a first run stays practical on a laptop; the same setup step notes to raise that to five or more, ideally on a faster GPU runtime such as Colab's, once the goal is an actual model to keep rather than a quick check that the pipeline runs end to end. Even at two, that's already four full models trained from scratch, two attempts each for TVAE and CTGAN.

---

## What one real trial looked like

Two trials from an actual run of this search make "each trial samples a combination of hyperparameters" concrete instead of abstract. The first trial trained for 400 rounds and took relatively large steps each time it corrected itself. The second trained for fewer rounds, 300, took smaller steps, and, the difference that mattered most, used a much larger batch size, the number of examples it groups together before each correction: 512 at a time against the first trial's 128. On this run, the second trial's combination scored better and became the search's answer.

---

## Why the search shows its work, not just a winner

A search that only hands back one winning model is a black box: trust it or don't. Two plots make it inspectable instead. The parallel-coordinates plot draws one line per trial across every hyperparameter axis and the score it achieved, so the two trials above would show up as two such lines, and a band of lines converging in the same region of an axis shows where the good scores actually came from. The importances plot ranks each hyperparameter by how much varying it moved the score, separating the settings that mattered on this dataset, like that batch size, from the ones that could have stayed at their defaults the whole time.

Between CTGAN and TVAE, the more useful comparison is the metric this week's actual use case depends on, its suitability score, the check that trains a model on the synthetic data and tests it on real customers, rather than the search's blended objective. A model that scores well on fidelity and privacy but poorly there hasn't solved this week's problem, whatever its other numbers say.

---

## Wrap-up

The tuner in this notebook runs twice, once per model family, and the winner is whichever one actually transfers to real customers, not whichever one the search's blended score happened to favor. The next piece reads that winner's full evaluation report, and puts it to the test this whole exercise was built around: does a propensity model trained on the synthetic table actually work on real people?

**Also in Session 4**: when to use synthetic data at all, and preprocessing real first-party data before it meets a model.

Questions belong in the Circle community.
