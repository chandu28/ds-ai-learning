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
| Programming | Python |
| Data Analysis | NumPy, Pandas |
| Visualization | Matplotlib, Seaborn, Plotly |
| Machine Learning | Scikit-learn |
| Notebooks | Jupyter |
| Development | Cursor, Git, GitHub |
| Quality | Ruff, Pytest |
| Environment | uv |

## Repository Structure

```text
ds-ai-learning/
├── configs/
├── data/
│   ├── raw/
│   └── processed/
├── models/
├── notebooks/
├── src/
│   └── ds_ai_learning/
├── tests/
├── visualizations/
├── AGENTS.md
├── PROJECT_ARCHITECTURE.md
├── pyproject.toml
└── README.md
```

## Development Setup

Requirements: macOS, Python 3.14, and uv.

```bash
git clone https://github.com/chandu28/ds-ai-learning.git
cd ds-ai-learning
uv sync
```

Run tests:

```bash
uv run pytest
```

Run code quality checks:

```bash
uv run ruff check .
```

## Project Roadmap

This repository will grow through practical, end-to-end projects covering:

- Data analysis and exploratory data analysis.
- Statistical analysis and experimentation.
- Machine learning and model evaluation.
- Applied AI and LLM engineering.
- Production-oriented data and ML pipelines.

## Engineering Practices

- Feature branches and Pull Requests.
- Signed Git commits.
- Automated testing and linting.
- Reproducible experiments.
- Modular, maintainable Python code.

---

**Status:** Learning and project development in progress.