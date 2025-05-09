---
title: Python Development Setup
date: 2024-09-07
---

Recently, I started to center my python development around [`uv`](https://docs.astral.sh/uv) and other tooling  developed by [astral.sh](https://astral.sh/). It allows me to manage python environments, linting and formatting.

## Managing Python Packages using `uv`

`uv` is an extremely fast Python package and recently also became a full on package manager.

You can install it using `curl`.

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

You can then create a new a new Python project, or use an existing one by running the `uv init` command:

```bash
uv init <project-name>
# or if already in project
uv init
```

This will create the following structure:

```bash
.
├── .python-version  # project's default python version
├── README.md
├── hello.py
└── pyproject.toml   # metadata about the project
```

Run `uv add` to add dependencies. This will create an virtual environment (`.venv`), if you didn't already have one and it then installs the dependencies. The installed dependency will be present in the `pyproject.toml`.

```bash
uv add <dependency>
```

Your project will have the following structure:

```bash
.
├── .venv               # project's virtual environment
├── .python-version
├── README.md
├── hello.py
├── pyproject.toml 
└── uv.lock             # lockfile containing exact info about projects dependencies
```

## Development

I like to have several things in my python development setup. A good linter and formatter. Some kind of tool that allows me to test my code.

And finally pre-commit hooks that run some checks (including formatting, linting and tests) before my commits.

### Linting & formatting using `ruff`

I used to use black, flake8 and isort for my linting and code formatting. However since `ruff` was released I haven't looked back.

```bash
uv add --dev ruff
```

Using ruff after black, isort and flake8 is delightful. `ruff` is a super fast linter (fixing issues and organizing imports) and code formatter. 

```bash
# ruff linting
uv run ruff check --fix --select I

# ruff formatting
uv run ruff format
```

### `pre-commit` hooks

pre-commit hooks run before your commits are made. These hooks can perform a variety of checks.

```bash
uv add --dev pre-commit
```

With the following `.pre-commit.yaml`.

```yaml
repos:
-   repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.6.0
    hooks:
    -   id: trailing-whitespace
    -   id: end-of-file-fixer
    -   id: check-toml
    -   id: check-json
    -   id: check-yaml
    -   id: check-xml
    -   id: check-ast
    -   id: check-docstring-first
    -   id: check-executables-have-shebangs
    -   id: check-added-large-files
    -   id: check-case-conflict
    -   id: check-merge-conflict
    -   id: check-symlinks
    -   id: detect-private-key
    -   id: no-commit-to-branch
- repo: https://github.com/astral-sh/ruff-pre-commit
  rev: v0.6.4
  hooks:
    - id: ruff
      args: [ --fix, --select, I ]
    - id: ruff-format
```

### Testing using `pytest`

I don't always write tests, but if I do, I'll most likely use pytest, and a pytest coverage plugin.

```bash
uv add --dev pytest pytest-cov
```

To run those test (with test coverage) against my python files I run the following.

```bash
uv run pytest <test-dir> --cov <source-dir>
```

## CI with GitHub actions

I also like to run linting, formatting and tests in CI. Below is a GitHub Action at `.github/workflows/ci.yaml`. 

```yaml
name: CI
on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
        # Checkout repo
      - uses: actions/checkout@v4

        # Setup uv
      - name: Install uv
        uses: astral-sh/setup-uv@v2

        # Setup python and install dependencies
      - name: Install dependencies
        run: uv sync --all-extras --dev

        # Run linting and formatting
      - name: Run ruff
        run: |
            uv run ruff check --fix --select I --output-format=github .
            uv run ruff format

        # Run pytest for tests
      - name: Run tests
        run: uv run python -m pytest
```

## Links 

- https://docs.astral.sh/uv/
- https://docs.astral.sh/ruff/

