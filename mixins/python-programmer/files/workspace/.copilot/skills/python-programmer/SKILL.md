---
name: python-programmer
description: 'Use alongside software-craftsman when writing or reviewing Python code. Adds Python-specific enforcement of linting/formatting, type hints, and automated testing, plus Python best practices (PEP 8, virtual environments, dependency management, error handling, security).'
---

# python-programmer

Use this skill together with `software-craftsman` whenever writing new Python code or reviewing existing Python code or a diff. `software-craftsman` supplies the general workflow, output format, and language-agnostic craftsmanship/security/anti-pattern rules — this skill adds the Python-specific rules on top of it. Apply these rules pragmatically — favor simple, readable, well-tested, statically-checkable code.

## Workflow (Python-specific additions)

1. Inspect `pyproject.toml` (and any `setup.cfg`/`tox.ini`) to discover the project's configured linter, formatter, type checker, and test runner. Use whatever is already configured — do not introduce a competing tool.
2. Apply the Linting & Formatting, Type Hints, and Testing rules below, plus the general Python Best Practices, alongside `software-craftsman`'s Definition of Good Software Craftsmanship.
3. **In `write` mode**, additionally: add/update type hints and tests, and run the discovered (or default) lint/format/type-check/test commands before self-checking against the Definition of Done.
4. **In `review` mode**, additionally flag: missing/incomplete type hints, missing tests, and lint/format/type-check violations as Blockers in `software-craftsman`'s Output Format.

## Linting & Formatting (mandatory)

- **Discover the project's configured tools first.** Check `pyproject.toml` for `[tool.ruff]`, `[tool.black]`, `[tool.isort]`, `[tool.flake8]`/`.flake8`, `[tool.pylint]`, etc. Use whichever linter and formatter are already configured there.
- **If no linter/formatter is configured in `pyproject.toml`, default to `ruff`** (`ruff check .` for linting, `ruff format .` for formatting) — it is fast and covers both roles.
- Run the linter and formatter on every file you create or modify before considering the work done. Fix all reported issues in changed code; do not silently suppress rules (`# noqa`, `# type: ignore`) without a comment explaining why.
- Do not reformat or relint files outside the scope of the change.

## Type Hints (mandatory)

- **All functions and methods must have full type hints** — parameter types and return type (including `-> None` where applicable) — for both new and modified code.
- **Exception: test functions/methods** (e.g., anything under `tests/`, or named `test_*`/`*_test`, or using a test framework's fixtures) are not required to have type hints, since test parameters are often fixtures with dynamic types.
- Use precise types from `typing`/`collections.abc` (e.g., `Sequence[str]`, `Mapping[str, int]`) over bare `list`/`dict` when the contract matters; use built-in generics (`list[int]`, `dict[str, int]`) per PEP 585 instead of `List`/`Dict` on Python 3.9+.
- Prefer `X | None` (PEP 604) over `Optional[X]` on Python 3.10+; match whatever convention the codebase already uses.
- Run the project's type checker (e.g., `mypy`, `pyright` — check `pyproject.toml`/`mypy.ini`/`pyrightconfig.json` for configuration) on changed files and resolve reported errors.

## Testing (mandatory)

- **Discover the project's configured test manager first.** Check `pyproject.toml` for `[tool.pytest.ini_options]`, a `[tool.hatch.envs.test]`/`nox`/`tox` setup, or a `test` dependency group, and use that runner.
- **If no test manager is configured in `pyproject.toml`, default to `pytest`.**
- Every new or changed behavior must be covered by automated tests. Tests assert observable behavior/contracts, not internal implementation details, so they survive refactors.
- Mirror the project's existing test layout (e.g., `tests/` directory structure paralleling `src/`) and naming convention (`test_*.py`).
- Run the full relevant test command (not just a spot check) before reporting completion, and ensure it passes.

## Python Best Practices

- **Follow PEP 8** for naming and layout: `snake_case` for functions/variables/modules, `PascalCase` for classes, `UPPER_SNAKE_CASE` for constants. Let the formatter handle whitespace/line-length.
- **Isolate dependencies.** Use a virtual environment (`venv`, `uv`, `poetry`, or whatever the project already uses) — never install packages into the system interpreter. Declare all dependencies in `pyproject.toml` (or `requirements.txt`) with sensible version constraints.
- **Use context managers (`with`) for resource management** — files, network connections, locks, DB sessions — instead of manual `open()`/`close()` pairs.
- **Raise and catch specific exceptions.** Avoid bare `except:` or broad `except Exception:` without re-raising or logging with context; define custom exception classes for domain-specific error conditions instead of overloading built-ins.
- **Avoid mutable default arguments** (`def f(x: list[int] = []):`) — use `None` and initialize inside the function body.
- **Prefer composition and simple data containers.** Use `dataclasses` (or `pydantic` models if already in use) instead of hand-rolled `__init__`/getter/setter boilerplate.
- **Use f-strings** for string formatting over `%`-formatting or `.format()`, unless the codebase's logging convention requires lazy `%`-style formatting (e.g., stdlib `logging`).
- **Guard the module entry point** with `if __name__ == "__main__":` for any script that can also be imported.
- **Write docstrings for public modules, classes, and functions** (PEP 257) describing purpose, parameters, and return values — reserve inline comments for non-obvious *why*, not *what*.

## Security (Python-specific additions — see `software-craftsman` for the full generic checklist)

**Verify every item below in addition to the generic checklist:**

- [ ] All database queries with user input use parameterised queries (e.g., DB-API `?`/`%s` placeholders or an ORM) — never raw string concatenation/f-strings/`.format()`.
- [ ] No secrets, API keys, or connection strings are hardcoded. Configuration flows through environment variables (`os.environ`), a secrets manager, or a config file excluded from version control.
- [ ] All external inputs (HTTP, CLI, file, message) are validated/parsed through a schema or validator (e.g., `pydantic`, `argparse` types) before use.
- [ ] Never use `eval()`, `exec()`, or `pickle.load()`/`yaml.load()` (unsafe loader) on untrusted input.

## Anti-Patterns — Python-specific additions (see `software-craftsman` for the generic list)

- **DO NOT** introduce a new lint/format/test/type-check tool when the project already has one configured in `pyproject.toml`.
- **DO NOT** ship a function or method without type hints (except test functions).
- **DO NOT** ship new or changed behavior without accompanying automated tests.

## Definition of Done (Python-specific additions — see `software-craftsman` for the generic checklist)

Before reporting completion, verify all of the following in addition to the generic checklist:

- [ ] The project's linter and formatter (from `pyproject.toml`, or `ruff` by default) both pass on changed files.
- [ ] All new/modified functions and methods have complete type hints, except test functions.
- [ ] The project's type checker (if configured) passes on changed files.
- [ ] New or changed behavior has automated test coverage using the project's test manager (or `pytest` by default), and the test suite passes.
- [ ] Public modules, classes, and functions have docstrings.
