# python-github-actions

A collection of reusable GitHub Actions workflows for Python projects.

## Workflows

### `deps` — Install Python Dependencies

Installs Python dependencies into a virtual environment and uploads it as an artifact for use by downstream jobs.

**Inputs:**

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `extras` | Yes | — | The extras key to install (e.g. `dev`, `test`, `docs`) |
| `python_version` | No | `3.11` | Python version to use |

**Usage:**

```yaml
jobs:
  build-venv:
    uses: jhunufernandes/python-github-actions/.github/workflows/deps.yml@main
    with:
      extras: dev
```

---

### `tests` — Run Tests

Downloads the pre-built virtual environment artifact and runs the Python test suite using `unittest discover`.

**Inputs:**

| Input | Required | Description |
|-------|----------|-------------|
| `extras` | Yes | The extras key used when the deps artifact was created |

**Usage:**

```yaml
jobs:
  run-tests:
    needs: build-venv
    uses: jhunufernandes/python-github-actions/.github/workflows/tests.yml@main
    with:
      extras: dev
```

---

### `docs-deploy` — Deploy Documentation

Downloads the pre-built virtual environment artifact and deploys MkDocs documentation to GitHub Pages.

**Inputs:**

| Input | Required | Description |
|-------|----------|-------------|
| `extras` | Yes | The extras key used when the deps artifact was created |

Expects a MkDocs configuration file at `docs/mkdocs.yml`.

**Usage:**

```yaml
jobs:
  deploy-pages:
    needs: build-venv
    uses: jhunufernandes/python-github-actions/.github/workflows/docs.yml@main
    with:
      extras: docs
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

### `issue-automation` — Automate Issue Workflow

When triggered by an issue event, this workflow:

1. Creates a branch named `issue/<number>-<title>`
2. Assigns the issue to the repository owner
3. Opens a draft pull request targeting the specified base branch

**Inputs:**

| Input | Required | Description |
|-------|----------|-------------|
| `pr_base` | Yes | The base branch for the draft pull request (e.g. `main`) |

**Usage:**

```yaml
on:
  issues:
    types: [opened]

jobs:
  automate:
    uses: jhunufernandes/python-github-actions/.github/workflows/auto.yml@main
    with:
      pr_base: main
```

---

### `lint` — Lint and Type-check

Installs the project and runs `ruff check` and `ty check`.

**Inputs:**

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `python_version` | No | `3.11` | Python version to use |
| `install_args` | No | `-e . --group dev` | Arguments passed to `pip install` — extras like `.[dev]` or PEP 735 dependency groups like `-e . --group dev` |

**Usage:**

```yaml
jobs:
  lint:
    uses: jhunufernandes/python-github-actions/.github/workflows/lint.yml@main
    with:
      python_version: "3.14"
```

---

### `docker-publish` — Build and Push a Docker Image to GHCR

Builds a multi-arch image and pushes it to GHCR as `:latest` plus a short-SHA tag, logging in with the workflow's `GITHUB_TOKEN`.

**Inputs:**

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `image_name` | No | Lowercased `github.repository` | Image name without the `ghcr.io/` prefix |
| `platforms` | No | `linux/amd64,linux/arm64` | Target platforms |

The calling workflow must grant `packages: write` (and `contents: read`).

**Usage:**

```yaml
permissions:
  contents: read
  packages: write

jobs:
  build-image:
    uses: jhunufernandes/python-github-actions/.github/workflows/docker.yml@main
```

---

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
  build-venv:
    uses: jhunufernandes/python-github-actions/.github/workflows/deps.yml@main
    with:
      extras: dev

  run-tests:
    needs: build-venv
    uses: jhunufernandes/python-github-actions/.github/workflows/tests.yml@main
    with:
      extras: dev

  deploy-pages:
    needs: build-venv
    uses: jhunufernandes/python-github-actions/.github/workflows/docs.yml@main
    with:
      extras: docs

  create-release:
    needs: run-tests
    uses: jhunufernandes/python-github-actions/.github/workflows/release.yml@main

  automate-issue:
    if: github.event_name == 'issues'
    uses: jhunufernandes/python-github-actions/.github/workflows/auto.yml@main
    with:
      pr_base: main
```

## License

This project is provided as-is. See [LICENSE](LICENSE) for details.
