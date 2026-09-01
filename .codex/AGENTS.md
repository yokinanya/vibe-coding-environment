# Global Agent Rules

## Language

Default to Chinese in user-facing replies unless the user explicitly requests another language.

## Response Style

Do not propose follow-up tasks or enhancement at the end of your final answer.

## Debug-First Policy (No Silent Fallbacks)

- Do **not** introduce new boundary rules / guardrails / blockers / caps (e.g. max-turns), fallback behaviors, or silent degradation **just to make it run**.
- Do **not** add mock/simulation fake success paths (e.g. returning `(mock) ok`, templated outputs that bypass real execution, or swallowing errors).
- Do **not** write defensive or fallback code; it does not solve the root problem and only increases debugging cost.
- Prefer **full exposure**: let failures surface clearly (explicit errors, exceptions, logs, failing tests) so bugs are visible and can be fixed at the root cause.
- If a boundary rule or fallback is truly necessary (security/safety/privacy, or the user explicitly requests it), it must be:
  - explicit (never silent),
  - documented,
  - easy to disable,
  - and agreed by the user beforehand.

## Engineering Quality Baseline

- Follow SOLID, DRY, separation of concerns, and YAGNI.
- Use clear naming and pragmatic abstractions; add concise comments only for critical or non-obvious logic.
- Remove dead code and obsolete compatibility paths when changing behavior, unless compatibility is explicitly required by the user.
- Consider time/space complexity and optimize heavy IO or memory usage when relevant.
- Handle edge cases explicitly; do not hide failures.

## Code Metrics (Hard Limits)

- **Function length**: 50 lines (excluding blanks). Exceeded  extract helper immediately.
- **File size**: 350 lines. Exceeded  split by responsibility.
- **Nesting depth**: 3 levels. Use early returns / guard clauses to flatten.
- **Parameters**: 3 positional. More  use a config/options object.
- **Cyclomatic complexity**: 10 per function. More  decompose branching logic.
- **No magic numbers**: extract to named constants (`MAX_RETRIES = 3`, not bare `3`).

## Decoupling & Immutability

- **Dependency injection**: business logic never `new`s or hard-imports concrete implementations; inject via parameters or interfaces.
- **Immutable-first**: prefer `readonly`, `frozen=True`, `const`, immutable data structures. Never mutate function parameters or global state; return new values.

## Security Baseline

- Never hardcode secrets, API keys, or credentials in source code; use environment variables or secret managers.
- Use parameterized queries for all database access; never concatenate user input into SQL/commands.
- Validate and sanitize all external input (user input, API responses, file content) at system boundaries.
- **Conversation keys  code leaks**: When the user shares an API key in conversation (e.g. configuring a provider, debugging a connection), this is normal workflow  do NOT emit "secret leaked" warnings. Only alert when a key is written into a source code file. Frontend display is already masked; no need to remind repeatedly.

## Git Commit Signing (GPG)

- If the repository or global Git config has GPG signing enabled (`commit.gpgsign = true` or `user.signingKey` is set), **never** bypass it with `--no-gpg-sign`, `--no-verify`, or by temporarily disabling the config.
- If `git commit` fails because the GPG agent is waiting for a YubiKey touch (timeout / `gpg: signing failed: No pinentry` / `gpg: signing failed: Inappropriate ioctl for device`):
  1. **Prompt the user** to touch their YubiKey or unlock the smart card.
  2. After the user signals readiness, retry the commit **with the same signing config** — do not strip the signature flag.
  3. If the user explicitly decides to skip signing for this commit, they must make that choice themselves; the agent must not make it silently.

## Testing and Validation

- Keep code testable and verify with automated checks whenever feasible.
- When running backend unit tests, enforce a hard timeout of 60 seconds to avoid stuck tasks.
- Prefer static checks, formatting, and reproducible verification over ad-hoc manual confidence.

## Skills

Skills are stored in `~/.codex/skills/` (personal) and optionally `.codex/skills/` (project-shared).

Before starting a task:

- Scan available skills.
- If a skill matches, read its `SKILL.md` and follow it.
- Announce which skill(s) are being used.

### Ruff Defaults

If a project does not provide a `ruff.toml` or `pyproject.toml`, assume the following rule selection:

["E", "F", "W", "I", "C4", "UP"]

Rule groups:

* **E / W** — pycodestyle errors and warnings
* **F** — pyflakes
* **I** — import sorting
* **C4** — flake8-comprehensions
* **UP** — pyupgrade
