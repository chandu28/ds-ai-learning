# Data Science & AI Engineering

A hands-on learning and project portfolio focused on Data Science, Applied Machine Learning, and AI Engineering.

## Objectives

- Strengthen Python, SQL, statistics, and machine learning fundamentals.
- Build end-to-end data science and machine learning projects.
- Develop practical AI engineering skills.
- Apply software engineering and testing best practices.
- Create production-oriented, reproducible solutions.

## Technology Stack

| Area | Tools |
|---|---|
| Programming | Python 3.14 |
| Data Analysis | NumPy, Pandas, SciPy |
| Visualization | Matplotlib, Seaborn, Plotly |
| Machine Learning | Scikit-learn |
| Notebooks | Jupyter |
| Development | Cursor, Git, GitHub |
| Dependencies | uv, `uv.lock` |
| Quality | Ruff, Pytest |
| Security | Bandit, pip-audit, Gitleaks |
| CI | GitHub Actions (`.github/workflows/ci.yml`) |

Cursor is the editor used for this repository. The repo does not contain Cursor-specific project settings.

## Repository Structure

```text
ds-ai-learning/
├── .github/workflows/ci.yml
├── configs/
├── data/
│   ├── raw/                 # gitignored
│   └── processed/
├── models/                  # gitignored
├── notebooks/
├── src/
│   └── ds_ai_learning/
├── tests/
├── visualizations/
├── AGENTS.md
├── PROJECT_ARCHITECTURE.md
├── pyproject.toml
├── uv.lock
└── README.md
```

`configs/`, `data/processed/`, and `visualizations/` are present and empty. `data/raw/` and `models/` are gitignored so raw datasets and saved model artifacts stay out of Git.

## Development Setup

Requirements: macOS, Python 3.14, uv, and the Gitleaks CLI for the local secret scan. Python 3.14 is pinned in `.python-version`. Gitleaks is installed separately (Homebrew); it is not a uv dependency. Bandit and pip-audit are installed by `uv sync` from the `dev` dependency group.

```bash
git clone https://github.com/chandu28/ds-ai-learning.git
cd ds-ai-learning
uv sync
```

CI installs from the lockfile with `uv sync --locked`. Use that command locally when you want the same install CI uses.

## Code Quality & Security

Local checks match the GitHub Actions job. Run them before opening a pull request:

```bash
uv run ruff check .
uv run bandit -r src/
uv run pip-audit --skip-editable
uv run pytest
gitleaks git --redact
```

| Check | What it covers | Where it is configured |
|---|---|---|
| Ruff | Lint. Line length 88, target `py314`. | `[tool.ruff]` in `pyproject.toml`. CI runs `uv run ruff check .`. |
| Bandit | Python security issues under `src/`. Default rules. No Bandit config file. | `dev` dependency. CI runs `uv run bandit -r src/`. |
| pip-audit | Known vulnerabilities in installed dependencies. | `dev` dependency. CI runs `uv run pip-audit --skip-editable`, which skips the local editable package because it is not published on PyPI. |
| Pytest | Tests in `tests/test_*.py`. | `[tool.pytest.ini_options]` in `pyproject.toml`. |
| Gitleaks | Secrets in Git history. | Local CLI: `gitleaks git --redact`. CI: `gitleaks/gitleaks-action@v3` after a full checkout (`fetch-depth: 0`). No `.gitleaks.toml` in the repo. |

Ruff can format code, but the CI job does not run `ruff format`.

Commits are signed with SSH through local Git config (`gpg.format ssh`, `commit.gpgsign`, and an SSH signing key). The repository does not store that key or `allowed_signers`, and CI does not verify signatures. Confirm a commit locally with `git log -1 --show-signature`.

Work happens on a feature branch and merges through a pull request into `main`.

## CI workflow

Workflow file: `.github/workflows/ci.yml` (`Python CI and Security`).

It runs on pull requests targeting `main` and on pushes to `main`. Permissions are `contents: read`. One job, `quality-and-security`, runs on `ubuntu-latest`:

1. Check out the repository with full history.
2. Install uv and Python 3.14.
3. `uv sync --locked`.
4. Ruff, Bandit, pip-audit, and pytest, in that order.
5. Gitleaks over the Git history. Pull-request comments from the action are disabled (`GITLEAKS_ENABLE_COMMENTS: false`).

There is no deployment job.

## Project Roadmap

This repository will grow through practical, end-to-end projects covering:

- Data analysis and exploratory data analysis.
- Statistical analysis and experimentation.
- Machine learning and model evaluation.
- Applied AI and LLM engineering.
- Production-oriented data and ML pipelines.

## Engineering Practices

- Feature branches and pull requests into `main`.
- SSH-signed Git commits, configured outside this repository.
- Automated lint, test, and security checks in GitHub Actions.
- Reproducible experiments.
- Modular, maintainable Python code.
- Secrets, virtual environments, raw datasets, and saved models stay out of Git (see `.gitignore`).

---

**Status:** Learning and project development in progress.
