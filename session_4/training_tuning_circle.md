# Training and auto-tuning a synthetic data model

This is the text companion to the second half of the synthetic data pipeline notebook. The preprocessing piece left off with a clean, correctly typed customer table. This piece trains CTGAN and TVAE against it, using an automatic hyperparameter search rather than hand-tuning, and generates the synthetic table the rest of this week's work depends on.

**Time**: about 25 minutes, most of it spent waiting on training rather than writing code
**Cost**: $0. Everything here runs on Colab's own CPU/GPU, no API key involved

---

## One setup, already behind you

The environment setup for this whole week, Colab or local, happens once, in the notebook's very first cell, and the preprocessing piece covers exactly what it does and why. If that cell hasn't been run yet, or its check cell didn't print "Setup complete," that's the place to go before this piece, not this one.

---

## What the search is actually doing

Training a generative model adjusts millions of internal values automatically. A much smaller set of values, the hyperparameters, has to be decided before training starts: how many rounds to run, how large a step the model takes each time it corrects itself, how many layers deep its networks are. `SyntheticDataModelTuner` replaces hand-picking those values with a search: each trial samples a combination of hyperparameters, trains a full model with them, scores the result against the metrics named up front, and records everything. After the requested number of trials, the search hands back the best-scoring model, its hyperparameters, and the evaluation behind that choice.

This notebook requests one metric from each of the four families this week's earlier lesson described, sanity, statistical fidelity, suitability, and privacy, all four defined so that higher is better, which is what lets the search optimise their combined average in a single, consistent direction. Optuna, the library doing the actual sampling, treats every trial as an independent full training run, which is why the platform's own documentation calls this "time and resource consuming" and means it literally: five trials means five complete models trained from scratch, once for TVAE and once again for CTGAN.

---

## Reading the search, not just its answer

Two plots make the search inspectable rather than a black box that hands back a single winner. The parallel-coordinates plot draws one line per trial across every hyperparameter axis and the score each trial achieved, so a band of lines converging in the same region of an axis shows where the good scores actually came from. The importances plot ranks each hyperparameter by how much varying it moved the score, separating the settings that mattered on this dataset from the ones that could have stayed at their defaults the whole time.

Between CTGAN and TVAE, the more useful comparison isn't the search's blended objective, it's the metric this week's actual use case depends on: `performance.xgb`, the suitability check that trains a real model on the synthetic data and tests it on real customers. A model that scores well on fidelity and privacy but poorly there hasn't solved this week's problem, whatever its other numbers say.

---

## Wrap-up

The tuner in this notebook is the same tool run twice, once per model family, and the winner is whichever one actually transfers to real customers, not whichever one the search's blended score happened to favor. The next piece reads that winner's full evaluation report, and puts it to the test this whole exercise was built around: does a propensity model trained on the synthetic table actually work on real people?

**Also in Session 4**: when to use synthetic data at all, and preprocessing real first-party data before it meets a model.

Questions belong in the Circle community.
