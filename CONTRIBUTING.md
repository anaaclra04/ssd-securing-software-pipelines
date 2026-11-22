# Contributing

Thanks for contributing! This guide keeps the repo consistent and easy to review. Please read it before opening your first branch or pull request.

## Table of Contents

1. [Workflow Overview](#workflow-overview)
2. [Branches](#branches)
3. [Commits](#commits)
4. [Pull Requests](#pull-requests)
5. [Python Style](#python-style)
6. [YAML Style](#yaml-style)
7. [Secrets and Sensitive Data](#secrets-and-sensitive-data)
8. [Getting Help](#getting-help)

---

## Workflow Overview

1. Pick up (or create) an issue and assign yourself.
2. Pull the latest `main` and create a branch from it.
3. Make small, focused commits using Conventional Commits.
4. Push your branch and open a pull request linked to the issue.
5. Get at least **two approving reviews** and passing checks.
6. Squash-merge into `main`, then delete the branch.

```bash
git checkout main
git pull origin main
git checkout -b feat/12-parse-workflow-jobs
# ...work, commit...
git push -u origin feat/12-parse-workflow-jobs
```

**Never push directly to `main`.** All changes go through a pull request.

---

## Branches

### Naming format

```
<type>/<issue-number>-<short-description>
```

- Use lowercase letters, numbers, and hyphens only.
- Keep the description short (2–5 words).
- Include the issue number when there is one. If there isn't, leave it out: `docs/update-readme`.

### Branch types

Branch types match the commit types below.

| Type       | Use for                                   | Example                          |
|------------|-------------------------------------------|----------------------------------|
| `feat`     | A new feature                             | `feat/12-parse-workflow-jobs`    |
| `fix`      | A bug fix                                 | `fix/27-null-step-name`          |
| `docs`     | Documentation only                        | `docs/setup-instructions`        |
| `refactor` | Code change with no behavior change       | `refactor/31-split-parser`       |
| `test`     | Adding or fixing tests                    | `test/40-policy-edge-cases`      |
| `chore`    | Maintenance, dependencies, tooling        | `chore/bump-pyyaml`              |
| `ci`       | CI/CD configuration                       | `ci/add-lint-job`                |
| `perf`     | Performance improvements                  | `perf/cache-parsed-configs`      |
| `style`    | Formatting only (no logic changes)        | `style/apply-ruff-format`        |

### Branch hygiene

- One branch = one issue or one logical change.
- Keep branches short-lived. Rebase on or merge `main` often to avoid big conflicts.
- Delete your branch after it is merged.

---

## Commits

We follow the [Conventional Commits 1.0.0](https://www.conventionalcommits.org/en/v1.0.0/) specification.

### Format

```
<type>(<optional scope>): <short summary>

<optional body>

<optional footer(s)>
```

### Rules

- **type** is one of the branch types listed above.
- **scope** (optional) is the area of the codebase, in lowercase, e.g. `parser`, `policy`, `monitor`, `docs`, `ci`.
- **summary**:
  - Imperative mood: "add", not "added" or "adds".
  - Lowercase first letter, no period at the end.
  - 72 characters or fewer.
- **body** (optional) explains *what* and *why*, not *how*. Wrap lines at 72 characters.
- **footer** (optional) references issues or notes breaking changes.

### Breaking changes

Add `!` after the type/scope **and** a `BREAKING CHANGE:` footer:

```
feat(policy)!: rename allowed_paths to fs_allow

BREAKING CHANGE: policy files must use `fs_allow` instead of `allowed_paths`.
```

### Examples

```
feat(parser): extract job dependencies from workflow files
```

```
fix(monitor): handle steps with no name field

Steps without a `name` key caused a KeyError when building the
event log. Fall back to the step index instead.

Closes #27
```

```
docs: add local setup instructions to README
```

```
test(policy): add cases for deny-by-default network rules
```

### Good habits

- Commit early and often, but each commit should leave the code in a working state.
- Don't mix unrelated changes in one commit (e.g. a bug fix and a reformat).
- Use `Closes #<n>` or `Refs #<n>` in the footer to link issues.

---

## Pull Requests

### Title

The PR title must follow the same Conventional Commits format, since it becomes the squash-merge commit message:

```
feat(parser): extract job dependencies from workflow files
```

### Description

Every PR should include:

- **What** changed and **why**.
- **How to test** it (commands, inputs, expected output).
- **Linked issue** (`Closes #12`).
- Screenshots or sample output, when helpful.

### Before requesting review

- [ ] Branch is up to date with `main`.
- [ ] Code follows the Python and YAML style rules below.
- [ ] Linters and formatters pass locally.
- [ ] Tests are added or updated, and all tests pass.
- [ ] Docstrings and docs are updated for any changed behavior.
- [ ] No secrets, tokens, or personal files are committed.

### Reviews

- At least **two approvals** is required before merging.
- Reviewers should respond within **24 hours** (per our team contract).
- Be specific and kind in feedback. Suggest, don't demand. Use prefixes if helpful: `nit:`, `question:`, `blocking:`.
- The PR author resolves conversations and merges after approval.
- Use **Squash and merge** to keep `main` history clean.

---

## Python Style

We follow the standard Python guidelines:

- [PEP 8](https://peps.python.org/pep-0008/) for code style
- [PEP 257](https://peps.python.org/pep-0257/) for docstrings
- [PEP 484](https://peps.python.org/pep-0484/) for type hints

### Tooling

| Tool                                      | Purpose                    | Command              |
|-------------------------------------------|----------------------------|----------------------|
| [Ruff](https://docs.astral.sh/ruff/)      | Linting and import sorting | `ruff check .`       |
| Ruff formatter                            | Formatting (Black-compatible) | `ruff format .`   |
| [mypy](https://mypy.readthedocs.io/)      | Static type checking       | `mypy .`             |
| [pytest](https://docs.pytest.org/)        | Testing                    | `pytest`             |

Run all of them before pushing.

### Conventions

- **Line length:** 88 characters (Ruff/Black default).
- **Indentation:** 4 spaces, never tabs.
- **Naming:**
  - `snake_case` for functions, variables, and modules
  - `PascalCase` for classes
  - `UPPER_SNAKE_CASE` for constants
  - `_leading_underscore` for internal/private names
- **Imports:** grouped as standard library, third-party, then local, with a blank line between groups. No wildcard imports (`from x import *`).
- **Type hints:** required on all public functions and methods.
- **Strings:** prefer f-strings for formatting.

### Docstrings

Use **Google-style** docstrings with triple double quotes. Every public module, class, and function needs one.

```python
"""Parse GitHub Actions workflow files into structured phases."""

from pathlib import Path

import yaml

MAX_JOBS = 256


class WorkflowParser:
    """Reads a workflow file and extracts its jobs and steps.

    Attributes:
        path: Location of the workflow YAML file.
    """

    def __init__(self, path: Path) -> None:
        self.path = path

    def load_jobs(self) -> dict[str, dict]:
        """Load the jobs defined in the workflow.

        Returns:
            A mapping of job IDs to their raw job definitions.

        Raises:
            FileNotFoundError: If the workflow file does not exist.
            ValueError: If the file has more than ``MAX_JOBS`` jobs.
        """
        with self.path.open(encoding="utf-8") as f:
            data = yaml.safe_load(f) or {}

        jobs = data.get("jobs", {})
        if len(jobs) > MAX_JOBS:
            raise ValueError(f"Too many jobs: {len(jobs)}")
        return jobs
```

- One-line docstrings fit on one line: `"""Return the job count."""`
- Multi-line docstrings: a summary line, a blank line, then details.
- Use `Args:`, `Returns:`, `Raises:`, `Yields:`, and `Attributes:` sections as needed.

### Comments

- Start comments with `# ` (hash + one space).
- Write full sentences with a capital letter.
- Explain **why**, not what the code already says.
- Inline comments: at least **two spaces** before the `#`, and use them sparingly.

```python
# Fall back to the step index because `name` is optional in the spec.
step_id = step.get("name", f"step-{index}")

timeout = 30  # Seconds; matches the runner's default.
```

- Use `# TODO(username): description` for follow-up work, and open an issue for anything non-trivial.
- Don't leave commented-out code. Git history keeps it.

### Safety notes

- Always use `yaml.safe_load()`, never `yaml.load()` without a safe loader.
- Avoid `eval()`, `exec()`, and `shell=True` in `subprocess` calls.

---

## YAML Style

YAML files (workflows, configs, policies) are linted with [yamllint](https://yamllint.readthedocs.io/).

```bash
yamllint .
```

### Conventions

- **Extension:** use `.yml` consistently (or `.yaml`, but pick one per directory and stick to it).
- **Indentation:** 2 spaces, never tabs.
- **Document start:** begin files with `---`.
- **Line length:** 80 characters where practical.
- **Keys:** `snake_case` for our own config keys. Keep whatever casing an external tool requires (e.g. GitHub Actions uses `runs-on`).
- **Booleans:** use `true` / `false` only (not `yes`, `no`, `on`, `off`).
- **Strings:** leave them unquoted unless needed. When quoting, prefer double quotes. Quote values that could be misread (e.g. `"1.10"`, `"*"`, `"off"`).
- **Lists:** indent list items under their parent key.
- **No trailing whitespace**, and end every file with a single newline.

### Comments

- Start comments with `# ` (hash + one space).
- Put a header comment at the top of each file explaining its purpose.
- Full-line comments go on their own line, indented to match the code they describe.
- Inline comments need at least **two spaces** before the `#`.

```yaml
---
# Lints and tests the project on every push and pull request.
name: CI

on:
  push:
    branches: [main]
  pull_request:

permissions:
  contents: read  # Least privilege: read-only by default.

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # Pin the Python version so results are reproducible.
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"

      - name: Install dependencies
        run: pip install -r requirements.txt

      - name: Run tests
        run: pytest
```

### Recommended `.yamllint` config

Place this at the repo root:

```yaml
---
extends: default

rules:
  line-length:
    max: 80
    level: warning
  truthy:
    allowed-values: ["true", "false"]
    check-keys: false  # Allows GitHub Actions' `on:` key.
  comments:
    min-spaces-from-content: 2
  indentation:
    spaces: 2
    indent-sequences: true
```

---

## Secrets and Sensitive Data

- **Never commit** API keys, tokens, passwords, `.env` files, or credentials.
- Use GitHub Secrets for anything CI needs.
- If you accidentally commit a secret, tell the team on Discord right away and rotate it. Removing it in a later commit is not enough.
- Keep `.gitignore` up to date for local files (virtual environments, IDE settings, build output).

---

## Getting Help

- Ask questions in the relevant Discord channel. Keep technical discussions out of DMs so the whole team can see them.
- Raise blockers early, either on Discord or at the weekly meeting.
- If something in this guide is unclear or out of date, open a `docs/` PR to fix it.
