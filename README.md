# github-actions-workflows

Reusable building blocks for GitHub Actions: two **composite actions** and one **reusable
workflow**. They are used by the training repository
[`github-actions-training`](https://github.com/nsinha-livevox/github-actions-training), which
explains how each one is called, line by line.

## What is here

| Path | Kind | What it does |
|---|---|---|
| `.github/actions/write-summary/action.yml` | Composite action | Writes a Markdown block to the run's Summary page |
| `.github/actions/setup-python-env/action.yml` | Composite action | Installs Python with the pip cache, then the requirements |
| `.github/workflows/reusable-python-test.yml` | Reusable workflow | Runs `pytest` on one or more Python versions and keeps a test report |

## How to use them

You always call a piece with `uses:` and a **version** after the `@`. Use the tag (`@v1`), not
`@main`, so a change here cannot break your pipeline unexpectedly.

### `write-summary` (a step)

```yaml
- uses: nsinha-livevox/github-actions-workflows/.github/actions/write-summary@v1
  with:
    title: Build
    message: Version 1.0.7 is ready
```

| Input | Required | Description |
|---|---|---|
| `title` | yes | Heading of the block |
| `message` | yes | Text under the heading (Markdown allowed) |

### `setup-python-env` (a step)

```yaml
- uses: nsinha-livevox/github-actions-workflows/.github/actions/setup-python-env@v1
  with:
    python-version: '3.12'
```

| Input | Default | Description |
|---|---|---|
| `python-version` | `3.12` | Python version to install |
| `requirements` | `requirements.txt` | Requirements file to install (also the cache key) |

| Output | Description |
|---|---|
| `python-version` | The exact Python version that was installed |

### `reusable-python-test.yml` (a whole job)

```yaml
jobs:
  test:
    uses: nsinha-livevox/github-actions-workflows/.github/workflows/reusable-python-test.yml@v1
    with:
      python-versions: '["3.11", "3.12"]'
```

| Input | Default | Description |
|---|---|---|
| `python-versions` | `["3.12"]` | JSON list of versions: one test job per version (a matrix) |
| `test-path` | `tests` | Folder or file that holds the tests |
| `requirements` | `requirements.txt` | Requirements file to install |

The workflow checks out the **calling** repository, installs the requirements, runs `pytest` and
uploads `test-report-python-<version>` as an artifact (also when a test fails). It needs `pytest`
in the caller's requirements file.

## Composite action or reusable workflow?

| | Composite action | Reusable workflow |
|---|---|---|
| Shares | A group of steps | A whole job (or several) |
| Called in | `steps:` | `jobs:` |
| Runner | The caller's job provides it | Defines its own `runs-on` |
| File | `action.yml` | Any name, `on: workflow_call` |

## Versions

The files reference each other by tag (`reusable-python-test.yml` uses `write-summary@v1`), so all
three pieces move together under one tag. To release a change:

```bash
git tag -f v1            # move the major tag to the latest commit (or create v2 for a breaking change)
git push -f origin v1
```

Callers that use `@v1` pick the change up on their next run. Breaking changes (renamed or removed
inputs) deserve a new major tag.

## Checking your changes

[`actionlint`](https://github.com/rhysd/actionlint) catches mistakes in workflow files before you
push (`brew install actionlint`, then `actionlint`).

## Public repository

This repository is public, so any repository can call these pieces without extra access settings,
and GitHub-hosted runner minutes are free. Keep it free of anything internal: no company names,
hosts, account numbers or secrets.
