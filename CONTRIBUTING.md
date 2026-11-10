# Contributing

## Setup

1. Install Git, Docker Desktop, Python 3.12 and [Task](https://taskfile.dev).
2. Copy `.env.example` to `.env` and set local passwords (never commit `.env`).
3. Create a virtual environment: `python -m venv .venv`, then activate it.
4. Install the hooks: `pip install pre-commit` and `pre-commit install`.

## Workflow

- Branch from `main` using `type/short-description`, for example
  `feat/charge-idempotency`.
- Write commits following Conventional Commits: `type: short summary in the
  imperative`. Types: feat, fix, docs, chore, refactor, test, ci, build, perf,
  style, revert.
- Keep pull requests small and focused. Fill in the pull request template.

## Useful commands

Run `task` to list all available tasks.
