# python-github-actions

A collection of reusable GitHub Actions workflows for Python projects.

## Workflows

### `deps` — Install Python Dependencies

Installs Python dependencies into a virtual environment and uploads it as an artifact for use by downstream jobs.

**Inputs:**

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `pip_args` | Yes | — | The extras key to install (e.g. `dev`, `test`, `docs`) |
| `python_version` | No | `3.11` | Python version to use |

**Usage:**

```yaml
jobs:
  install-deps:
    uses: jhunufernandes/python-github-actions/.github/workflows/deps.yml@main
    with:
      pip_args: dev
```

---

### `tests` — Run Tests

Downloads the pre-built virtual environment artifact and runs the Python test suite using `unittest discover`.

**Inputs:**

| Input | Required | Description |
|-------|----------|-------------|
| `pip_args` | Yes | The extras key used when the deps artifact was created |

**Usage:**

```yaml
jobs:
  run-tests:
    needs: install-deps
    uses: jhunufernandes/python-github-actions/.github/workflows/tests.yml@main
    with:
      pip_args: dev
```

---

### `docs` — Deploy Documentation

Downloads the pre-built virtual environment artifact and deploys MkDocs documentation to GitHub Pages.

**Inputs:**

| Input | Required | Description |
|-------|----------|-------------|
| `pip_args` | Yes | The extras key used when the deps artifact was created |

Expects a MkDocs configuration file at `docs/mkdocs.yml`.

**Usage:**

```yaml
jobs:
  deploy-docs:
    needs: install-deps
    uses: jhunufernandes/python-github-actions/.github/workflows/docs.yml@main
    with:
      pip_args: docs
```

---

### `release` — Create a GitHub Release

Reads the version from `pyproject.toml`, creates a Git tag, and publishes a GitHub release.

**Inputs:** None

**Usage:**

```yaml
jobs:
  create-release:
    uses: jhunufernandes/python-github-actions/.github/workflows/release.yml@main
```

---

### `auto` — Automate Issue Workflow

When triggered by an issue event, this workflow:

1. Creates a branch named `issue/<number>-<title>`
2. Assigns the issue to the repository owner
3. Opens a draft pull request targeting the specified base branch

**Inputs:**

| Input | Required | Description |
|-------|----------|-------------|
| `pr_destiny` | Yes | The base branch for the draft pull request (e.g. `main`) |

**Usage:**

```yaml
on:
  issues:
    types: [opened]

jobs:
  automate:
    uses: jhunufernandes/python-github-actions/.github/workflows/auto.yml@main
    with:
      pr_destiny: main
```

## Example Pipeline

Below is a complete example that chains `deps`, `tests`, `docs`, and `release` together:

```yaml
name: CI/CD

on:
  push:
    branches: [main]
  issues:
    types: [opened]

jobs:
  install-deps:
    uses: jhunufernandes/python-github-actions/.github/workflows/deps.yml@main
    with:
      pip_args: dev

  run-tests:
    needs: install-deps
    uses: jhunufernandes/python-github-actions/.github/workflows/tests.yml@main
    with:
      pip_args: dev

  deploy-docs:
    needs: install-deps
    uses: jhunufernandes/python-github-actions/.github/workflows/docs.yml@main
    with:
      pip_args: docs

  create-release:
    needs: run-tests
    uses: jhunufernandes/python-github-actions/.github/workflows/release.yml@main

  automate-issue:
    if: github.event_name == 'issues'
    uses: jhunufernandes/python-github-actions/.github/workflows/auto.yml@main
    with:
      pr_destiny: main
```

## License

This project is provided as-is. See [LICENSE](LICENSE) for details.
