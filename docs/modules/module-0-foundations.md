# Module 0 — Engineering Foundations

## Purpose
Establish the development discipline every later module depends on: version control workflow,
automated linting, CI enforcement, secrets management, and environment separation — enforced by
tooling, not memory. No product code is written until this exists.

## Architecture
- **Git**: trunk-based, `main` protected, PRs required (see CONTRIBUTING.md)
- **Pre-commit**: local gate — Ruff (lint + format), Gitleaks (secret scan), file hygiene checks
- **CI**: GitHub Actions — re-runs the same checks remotely on every push/PR to `main`
- **Secrets**: Ansible Vault, password stored outside the repo, encrypted file committed
- **Environments**: `.env.example` committed, `.env` gitignored, `docker-compose.override.yml` for local overrides

## Installation
1. Clone the repo.
2. `pip install pre-commit && pre-commit install`
3. Follow `ansible/vault/README.md` to set up your local vault password.
4. `cp .env.example .env` and fill in local values.

## Configuration
- Ruff rules: `pyproject.toml` (`[tool.ruff]`)
- CI pipeline: `.github/workflows/ci.yml`
- Pre-commit hooks: `.pre-commit-config.yaml`

## Usage
- Normal flow: branch → commit (hooks run automatically) → push → PR → CI runs → merge.
- To manually run all checks without committing: `pre-commit run --all-files`

## Testing (how Module 0 itself is verified — see full test plan below)
1. Intentionally commit a badly-formatted file → pre-commit blocks it locally.
2. Intentionally stage a fake API key string → Gitleaks blocks the commit.
3. Push a branch with a lint error, bypassing pre-commit (`--no-verify`) → confirm CI catches it and fails the check.
4. Run `ansible-vault edit ansible/vault/secrets.yml` with wrong password → confirm it refuses to decrypt.
5. Run `cat ansible/vault/secrets.yml` → confirm output is ciphertext, not plaintext.

## Troubleshooting
- **Pre-commit hook not running**: confirm `pre-commit install` was run in this repo (it's per-clone, not global).
- **Gitleaks false positive**: add a `.gitleaksignore` entry — never disable the hook globally.
- **CI passes locally but fails remotely**: usually a Python version mismatch — check `.github/workflows/ci.yml` matches your local `python --version`.

## Future Improvements (explicitly deferred, not forgotten)
- Add Dependabot for automated dependency updates
- Add commit signing (GPG) once collaborating with others
- Add `semantic-release` for automated changelog/versioning once the project has real releases
- Extend CI with a Docker build step once Module 1 introduces the Compose stack
