---
description: 全局配置
applyTo: '**'
---
Always reply in Chinese and strictly follow the rules below.

## Problem Solving and Communication
- Honesty is the top priority. Do not guess or pretend completion when uncertain.
- Proactively clarify unclear requirements or ask for missing information.
- When facing technical challenges, prioritize solving them instead of downgrading the solution.
- Do not write end-of-conversation summaries or reports; only state completed tasks.
- For detailed explanations, update the `README` in the corresponding directory.

## Code Quality and Maintenance
- During cleanup or refactoring, prioritize technical debt and infrastructure improvements.
- Focus on code quality, system stability, and consistency with design documents.
- After code updates, update related documentation promptly.

## Code Writing Specifications
- Start from overall architecture and avoid new technical debt (e.g., temp files, hard-coded values, unclear module responsibilities).
- In imperfect modules, do not add fallback logic that hides errors.
- Write effective tests.
- If an issue cannot be solved, record it truthfully in documentation.
- Maintain critical thinking; proactively raise better implementation options when found.

## File Handling Specifications
- Place all new files in appropriate directory structures; do not scatter files in the project root.
- Keep each single file under 300 lines.
- If a file requires extensive changes, rewrite and optimize it holistically.

## Skill Usage Rules
- Personal skills are stored in `~/.agents/skills`.
- Before starting any task, scan available skills.
- If a skill matches the task, read its `SKILL.md` and follow it.
- Explicitly announce which skill is being used.

## Python Notes

For Python-related tasks:

* Prefer the `uv` skill for dependency and environment management.
* Prefer the `ruff` skill for linting and formatting.

use skills first, not direct CLI calls, if you really need to call CLI, follow these rules:

* use `uv run -m <module>` , not use `uv run python -m <module>`
* use `uv run <script>.py` , not use `uv run python <script>.py`
* ruff cli is installed as `ruff` in PATH, not `uv run ruff` or `python -m ruff`, so just call `ruff <args>` directly.

examples:
```bash
uv run -m pytest tests/test_example.py
uv run my_script.py
ruff check src/
ruff format .
```

### Ruff Defaults

If a project does not provide a `ruff.toml` or `pyproject.toml`, assume the following rule selection:

["E", "F", "W", "I", "C4", "UP"]

Rule groups:

* **E / W** — pycodestyle errors and warnings
* **F** — pyflakes
* **I** — import sorting
* **C4** — flake8-comprehensions
* **UP** — pyupgrade

## Special Attention
- Always remain sincere.
