# Contributing / Working Agreement (EAOS)

Even as a solo project, this repo follows real team discipline. That's the point of Module 0.

## Branching
- `main` is protected — nothing is pushed to it directly.
- Every change happens on a short-lived branch: `feat/module-2-identity`, `fix/jwt-expiry-bug`.
- Merge to `main` via Pull Request, even solo — this forces you to review your own diff with fresh eyes and confirms CI passes before merge.

## Commit convention (Conventional Commits)
Format: `type(scope): short description`

Types: `feat`, `fix`, `docs`, `chore`, `refactor`, `test`, `ci`

Examples:
- `feat(identity): add JWT-based login endpoint`
- `fix(ai-service): handle Ollama timeout gracefully`
- `docs(module-2): add ERD and installation steps`

## Before every commit
Pre-commit hooks run automatically (`pre-commit install` once per clone) and will:
- lint and format with Ruff
- block large files, merge conflict markers, trailing whitespace
- scan for secrets with Gitleaks

If a hook fails, fix the issue — do not bypass with `--no-verify` except in a genuine emergency, and never for the secret scanner.

## Pull Requests
Every PR description answers:
1. What does this change?
2. Why was it needed?
3. How was it tested?

CI must pass (lint, format check, tests, secret scan) before merge.
