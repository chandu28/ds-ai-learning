# DS and AI Learning Project

## Goals
Build practical projects for Data Scientist, AI Engineer and Applied ML roles.

## Technology
- Python 3.14 for current data science projects. `.python-version` pins `3.14`.
- Use uv for dependency management. Keep `pyproject.toml` and `uv.lock` together.
- Use Cursor as the editor. Do not add Cursor project settings unless asked.
- Use Jupyter for exploration and experimentation.
- Use reusable modules inside `src/ds_ai_learning/`.
- Use pytest for automated tests.
- Use Ruff for linting. The CI command is `uv run ruff check .`.
- Use Bandit for security scanning of `src/` (`uv run bandit -r src/`).
- Use pip-audit for dependency vulnerabilities (`uv run pip-audit --skip-editable`).
- Use Gitleaks for secret detection. Local command: `gitleaks git --redact`. CI uses `gitleaks/gitleaks-action@v3`. Gitleaks is not a uv dependency.
- Use GitHub Actions for CI. The only workflow is `.github/workflows/ci.yml`. It does not deploy.
- Sign Git commits with the existing SSH signing setup. Do not change Git configuration.
- Land changes through a feature branch and a pull request into `main`.

## Coding standards
- Write clean, readable, modular Python.
- Add type hints and useful docstrings.
- Avoid duplicating code in notebooks.
- Use reproducible random seeds.
- Never commit secrets, virtual environments, raw datasets, or saved models.
- Explain important implementation decisions.

## Secure coding
These rules are mandatory. Only the tools configured in `pyproject.toml` and `.github/workflows/ci.yml` count as project security tooling.

- Do not hardcode secrets, tokens, passwords, keys, or connection strings in source, notebooks, tests, docs, or workflow files.
- Do not silence Bandit with `#nosec` unless the same change explains why the finding is safe.
- Keep Bandit's scan on `src/`. Do not point it at a narrower path to hide a file that should be scanned.
- Do not add a `.gitleaks.toml` allowlist, a Bandit skip, or a pip-audit ignore unless the user asks for that exception and the reason is written down in the change.
- Prefer the standard library and existing dependencies. Do not add a package to pass data through an `eval` or a shell string.
- Do not log secret values. The local Gitleaks command uses `--redact` so findings do not print the secret.
- If a secret was committed, say so and treat rotation as required. Deleting it in a later commit does not remove it from history, and CI scans the full history (`fetch-depth: 0`).
- Do not weaken `.github/workflows/ci.yml`: keep `permissions: contents: read`, and do not set `GITLEAKS_ENABLE_COMMENTS` to true unless asked.
- Do not claim a security control that is not configured. SSH signature verification and merge blocking are local or GitHub settings outside this repo. CI does not verify commit signatures, and this repo does not define branch protection.

## Secret handling
- `.gitignore` already excludes `.env`, `.venv/`, `data/raw/`, and `models/`. Keep secrets in an untracked `.env` or in the environment, not in the repo.
- Do not commit `.env`, key files, `allowed_signers`, or private keys. SSH signing uses the user Git config and `~/.ssh/`; leave that config alone.
- Before a commit that touches notebooks, configs, workflows, or data paths, run `gitleaks git --redact` when the Gitleaks CLI is installed.
- Do not paste live credentials into chat, commit messages, or documentation.
- Raw datasets stay in `data/raw/`. Saved model artifacts stay in `models/`. Neither directory is committed.

## Testing requirements
- After any code change, run `uv run pytest`.
- Before finishing a change that touches Python, dependencies, notebooks with code, or the workflow, run the same checks CI runs:

```bash
uv run ruff check .
uv run bandit -r src/
uv run pip-audit --skip-editable
uv run pytest
gitleaks git --redact
```

- A new behavior in `src/ds_ai_learning/` needs a test under `tests/`.
- ML model validation is separate from these gates. Follow section 4 of `PROJECT_ARCHITECTURE.md`: deterministic pytest for pipeline code, and a held-out evaluation with a baseline reported from the notebook.
- Do not skip CI steps or mark a check as optional in the workflow unless the user asks.

## Documentation maintenance
- When layout, commands, dependencies, ignore rules, or CI steps change, update `README.md`, `PROJECT_ARCHITECTURE.md`, and `AGENTS.md` in the same change.
- Read each file before editing it. Keep existing goals, standards, and the today-versus-planned split.
- Describe only tools and integrations that are configured. Bandit and pip-audit are dev dependencies. Gitleaks runs as a local CLI and as a GitHub Action. There is no deploy workflow, no Bandit config file, and no `.gitleaks.toml`.
- Label current behavior as **Today** and unimplemented layout as **Planned** in `PROJECT_ARCHITECTURE.md`.

## Working approach
- Explain the plan before making significant changes.
- Do not overwrite files without checking their contents.
- Run relevant tests after code changes.
- Prefer production-quality, maintainable solutions.
- Do not modify application source, tests, dependencies, or Git configuration unless the task asks for that.
