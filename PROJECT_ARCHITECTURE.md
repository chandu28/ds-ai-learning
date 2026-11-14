# Project Architecture

This document describes `ds-ai-learning` as it exists today and the target
layout for later Data Science and applied ML work. Sections labeled
**Today** describe the repository as it is. Sections labeled **Planned**
describe conventions that are not implemented yet. Nothing in the planned
sections implies that those folders, modules, or pipelines already exist.

## 1. Current project structure

**Today.** The repository is a single installable package managed by `uv`.
Python is 3.14 (`.python-version` and `requires-python`). Setuptools discovers
packages under `src`. Dependencies are locked in `uv.lock`.

```text
ds-ai-learning/
  .github/workflows/ci.yml           # lint, test, and security job
  .gitignore                         # .venv, .env, data/raw, models, caches
  .python-version                    # 3.14
  AGENTS.md
  PROJECT_ARCHITECTURE.md
  README.md
  pyproject.toml                     # metadata, dependencies, pytest, Ruff
  uv.lock
  configs/                           # empty
  data/
    raw/                             # gitignored; not present in Git
    processed/                       # empty
  models/                            # gitignored artifact directory
  notebooks/
    01_environment_test.ipynb        # pandas DataFrame and a Matplotlib line plot
  src/ds_ai_learning/
    __init__.py                      # empty
    utils.py                         # greet()
  tests/
    __init__.py
    test_basic.py                    # asserts greet("Chandu")
  visualizations/                    # empty
```

`pyproject.toml` declares the runtime stack (NumPy, pandas, SciPy,
scikit-learn, Matplotlib, Seaborn, Plotly, Jupyter, ipykernel). The `dev`
dependency group contains `bandit`, `pip-audit`, `pytest`, and `ruff`.
Pytest collects `tests/test_*.py`. Ruff uses an 88-character line length and
targets Python 3.14. There is no `[tool.bandit]` block and no Gitleaks
config file.

There is no feature pipeline and no evaluation module. The top-level
`data/` and `models/` directories are storage locations, not Python
packages. The environment notebook does not import `ds_ai_learning`.

## 2. Future target architecture

**Planned.** New work stays in this repository and this package. A topic
gets modules under `src/ds_ai_learning/`, one or more notebooks, and tests.
These package paths are the target layout. They are not in the tree yet.

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

The planned `src/ds_ai_learning/models/` package is source code. The
top-level `models/` directory remains a gitignored place for saved
artifacts.

Rules for that layout:

- A notebook may import the package. The package does not import notebooks.
- A transform, split, metric, or training step that is worth keeping moves
  into `src/ds_ai_learning/` with type hints and a docstring.
- Randomness uses a fixed seed.
- Raw datasets, secrets, virtual environments, and saved model files stay
  out of Git.
- Dependencies are added in `pyproject.toml` through `uv`, and `uv.lock` is
  updated in the same change. CI installs with `uv sync --locked`.

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

**Today.** The only automated test is `tests/test_basic.py`, which checks
the greeting string. There is no model to validate. CI still runs that test
on every pull request to `main` and every push to `main`.

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

- Python 3.14. `uv` is the dependency manager. `.python-version` pins `3.14`.
- Readable, modular Python with type hints and useful docstrings.
- Do not duplicate kept logic inside notebooks.
- Use reproducible random seeds.
- Do not commit secrets, virtual environments, raw datasets, or saved models.
- Explain important implementation decisions, and explain the plan before
  significant changes.
- Read a file before overwriting it.
- Run the relevant pytest tests after code changes.
- Ruff lints. Line length is 88. The Ruff target is `py314`. The CI gate is
  `uv run ruff check .`.
- Runtime dependencies belong in `[project].dependencies`. Pytest, Ruff,
  Bandit, and pip-audit belong in the `dev` dependency group.
- The build backend is setuptools, with package discovery rooted at `src`.
- Adding or changing a dependency updates both `pyproject.toml` and
  `uv.lock`. CI rejects a lockfile that does not match, because it runs
  `uv sync --locked`.

## 6. Quality gates

**Today.** These commands are the gates. They are what CI runs, and the
same commands are the local check before a pull request.

| Gate | Command | Scope |
|---|---|---|
| Lint | `uv run ruff check .` | The repository tree Ruff is pointed at |
| Security lint | `uv run bandit -r src/` | `src/` only, default Bandit rules |
| Dependency audit | `uv run pip-audit --skip-editable` | Installed distributions, skipping this editable package |
| Tests | `uv run pytest` | `tests/test_*.py` |
| Secrets | `gitleaks git --redact` locally; `gitleaks/gitleaks-action@v3` in CI | Git history |

`ruff format` is not a gate. Bandit does not scan `tests/` or `notebooks/`.
pip-audit does not audit the unpublished local project; `--skip-editable`
avoids treating that package as a missing PyPI distribution.

**Planned.** Model-quality checks in section 4 stay outside these gates.
They are reported with the experiment. They are not a GitHub Actions job.

## 7. Security practices

**Today.**

- `.gitignore` excludes `.venv/`, `__pycache__/`, `.ipynb_checkpoints/`,
  `.DS_Store`, `.env`, `data/raw/`, and `models/`.
- Bandit scans package code under `src/` with its default test set. There
  is no custom Bandit profile and no `#nosec` usage in the current source.
- pip-audit checks the locked environment for known vulnerabilities.
- Gitleaks scans commit history. CI checks out the full history
  (`fetch-depth: 0`) so the action is not limited to the latest commit.
  The local command redacts secret values in its output. The repo has no
  `.gitleaks.toml`.
- The Actions workflow token scope is `contents: read`. The Gitleaks action
  receives `GITHUB_TOKEN`, and `GITLEAKS_ENABLE_COMMENTS` is `false`.
- Git commits are SSH-signed from the developer machine (`gpg.format` is
  `ssh`, `commit.gpgsign` is enabled, signing key is an SSH public key in
  the user Git config). That config is not in the repository. CI does not
  check commit signatures.

**Planned.** Nothing further is configured. Do not describe a scanner,
branch-protection rule, or signature check in CI until it is actually
added to the repo.

## 8. CI architecture

**Today.** One workflow, `.github/workflows/ci.yml`.

- Name: `Python CI and Security`.
- Triggers: `pull_request` to `main`, and `push` to `main`.
- Permissions: `contents: read`.
- Job: `quality-and-security` on `ubuntu-latest`.
- Steps: checkout (`actions/checkout@v6`, full history), install uv
  (`astral-sh/setup-uv@v6`), `uv python install 3.14`, `uv sync --locked`,
  Ruff, Bandit, pip-audit, pytest, then `gitleaks/gitleaks-action@v3`.

The workflow is continuous integration only. It does not build a release
artifact and it does not deploy.

Changes reach `main` through a pull request from a feature branch. A push
straight to `main` also runs the same job. Required reviews and required
status checks are not defined in this repository, so this document does not
claim that GitHub blocks a merge when the job fails.
