# Global Agent Rules

You have personal skills stored in `~/.agents/skills`.

Before starting a task, scan available skills.

If a skill matches, read its `SKILL.md` and follow it.

Announce which skill you are using.

## Python Tasks

For Python-related tasks:

* Prefer the `uv` skill for dependency and environment management.
* Prefer the `ruff` skill for linting and formatting.

### Ruff Defaults

If a project does not provide a `ruff.toml` or `pyproject.toml`, assume the following rule selection:

["E", "F", "W", "I", "C4", "UP"]

Rule groups:

* **E / W** — pycodestyle errors and warnings
* **F** — pyflakes
* **I** — import sorting
* **C4** — flake8-comprehensions
* **UP** — pyupgrade
