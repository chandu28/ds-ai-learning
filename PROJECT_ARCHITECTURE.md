# Project Architecture

This document describes `ds-ai-learning` as it exists today and the target
layout for later Data Science and applied ML work. Sections labeled
**Today** describe the repository as it is. Sections labeled **Planned**
describe conventions that are not implemented yet. Nothing in the planned
sections implies that those folders, modules, or pipelines already exist.

## 1. Current project structure

**Today.** The repository is a single installable package managed by `uv`.
Python is 3.14 or newer. Setuptools discovers packages under `src`.

```text
ds-ai-learning/
  AGENTS.md
  README.md                          # empty
  pyproject.toml                     # project metadata, dependencies, pytest, Ruff
  src/ds_ai_learning/
    __init__.py                      # empty
    utils.py                         # greet()
  notebooks/
    01_environment_test.ipynb        # pandas DataFrame and a Matplotlib line plot
  tests/
    __init__.py
    test_basic.py                    # asserts greet("Chandu")
```

`pyproject.toml` declares the runtime stack (NumPy, pandas, SciPy,
scikit-learn, Matplotlib, Seaborn, Plotly, Jupyter, ipykernel) and a `dev`
dependency group (`pytest`, `ruff`). Pytest collects `tests/test_*.py`.
Ruff uses an 88-character line length and targets Python 3.14.

There is no data directory, no model code, no feature pipeline, and no
evaluation module. The environment notebook does not import
`ds_ai_learning`.

## 2. Future target architecture

**Planned.** New work stays in this repository and this package. A topic
gets modules under `src/ds_ai_learning/`, one or more notebooks, and tests.
These paths are the target layout. They are not in the tree yet, and this
document does not create them.

```text
src/ds_ai_learning/
  data/            # load, validate, split
  features/        # transforms shared by training and inference
  models/          # fit, predict, persist
  evaluation/      # metrics and comparison against a baseline
notebooks/
  NN_topic.ipynb   # exploration; calls the package once logic is kept
tests/
  test_<topic>.py  # small, deterministic checks of the same functions
```

Rules for that layout:

- A notebook may import the package. The package does not import notebooks.
- A transform, split, metric, or training step that is worth keeping moves
  into `src/ds_ai_learning/` with type hints and a docstring.
- Randomness uses a fixed seed.
- Raw datasets, secrets, and virtual environments stay out of git.
- Dependencies are added in `pyproject.toml` through `uv`.

## 3. Notebook versus production-code responsibilities

| | Notebooks (`notebooks/`) | Package code (`src/ds_ai_learning/`) |
|---|---|---|
| **Today** | `01_environment_test.ipynb` builds a five-row revenue table and plots it inline. | `utils.greet` is the only reusable function. |
| **Planned** | Look at data, try an approach, record plots, tables, and the validation write-up. | Named functions for load, split, features, fit, predict, metrics, and persistence. |
| **Planned** | Temporary, linear cells are acceptable while exploring. | Typed, documented, and covered by pytest once the approach is chosen. |
| **Planned** | Call package functions instead of copying their logic into cells. | No dependency on a notebook path or cell output. |

**Today**, that split is only a convention: the notebook and the package do
not share code. **Planned**, training and later inference import the same
feature and model functions the notebook used, so a result can be reproduced
outside Jupyter. Hyperparameters, seeds, and metric definitions that matter
live next to the code that uses them.

## 4. ML testing and validation standards

**Today.** The only automated check is `tests/test_basic.py`, which checks
the greeting string. There is no model to validate.

**Planned.** Code tests and model validation are separate, and both are
required before a model is treated as done.

Code tests run with pytest on small synthetic frames. They cover:

- Schema checks: expected columns and dtypes, and rejection of empty or malformed input.
- Splits: no train/test leakage, and stable membership when the seed is fixed.
- Features: a known input maps to a known output, and train and predict use the same function.
- A training smoke test: the estimator fits on a tiny set and `predict` returns the expected shape.
- Metrics: a hand-computed case matches the function.
- Persistence: save and load yields the same predictions.

Model validation answers whether the model is useful. It is reported with
the experiment, usually from the topic notebook, using the same package
functions:

- A held-out split or cross-validation, with the seed recorded.
- A simple baseline on that same split (majority class, mean predictor, or a linear model).
- The decision metric, plus one check the metric can hide (calibration, per-class recall, or residuals, depending on the task).
- A refit with the same seed to confirm the metric stays put.
- A short note on where the model fails.

Pytest stays fast and deterministic. Heavier validation stays in the
notebook. After a change to pipeline functions, the related tests are run
before the work is called finished.

## 5. Coding standards and dependency management

**Today and ongoing**, from `AGENTS.md` and `pyproject.toml`:

- Python 3.14 or newer. `uv` is the dependency manager.
- Readable, modular Python with type hints and useful docstrings.
- Do not duplicate kept logic inside notebooks.
- Use reproducible random seeds.
- Do not commit secrets, virtual environments, or raw datasets.
- Explain important implementation decisions, and explain the plan before
  significant changes.
- Read a file before overwriting it.
- Run the relevant pytest tests after code changes.
- Ruff lints and formats. Line length is 88. The Ruff target is `py314`.
- Runtime dependencies belong in `[project].dependencies`. Test and lint
  tools belong in the `dev` dependency group.
- The build backend is setuptools, with package discovery rooted at `src`.
