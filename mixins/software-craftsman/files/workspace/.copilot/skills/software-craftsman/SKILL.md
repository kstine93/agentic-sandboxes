---
name: software-craftsman
description: 'Use when writing or reviewing code. Applies SOLID, DRY, KISS, YAGNI, and Gang of Four patterns pragmatically. Covers clean code, security (OWASP Top 10), testability, and code review feedback.'
---

# software-craftsman

Use this skill whenever writing new code or reviewing existing code or a diff. Apply the principles pragmatically — favor simple, readable, well-tested code over dogmatic pattern application.

## Workflow

1. Determine the mode: `write` (producing new or modified code) or `review` (evaluating existing code or a diff).
2. Read the surrounding code and project conventions before acting — match existing patterns and dependencies rather than introducing new ones.
3. Apply the Definition of Good Software Craftsmanship and the Security checklist below to the change.
4. Check testability: flag or avoid tight coupling and hidden dependencies that would block unit testing.
5. **In `write` mode:** implement the change, add or update tests for new or changed behavior, then self-check against the Definition of Done.
6. **In `review` mode:** produce ranked feedback (blockers, then improvements, then suggestions) using the Output Format below.
7. Before finishing, verify the output against the Definition of Good Software Craftsmanship and Anti-Patterns.

## Output Format

**When reviewing code**, structure the response as:

1. **Blockers** — must fix before merge (security, correctness, broken contracts).
2. **Improvements** — should fix (maintainability, clarity, duplication).
3. **Suggestions** — optional (style preferences, future-proofing).

For each item: cite the file and line, state the issue, and show the suggested change as a code block.

**When writing code**, implement the change directly, then summarize: what changed, what tests were added or updated, and any open risks or follow-ups — using the Definition of Done as the checklist.

## Definition of Good Software Craftsmanship

- **Do not repeat code** unless the copies represent genuinely independent concepts that could change separately.
  - **EXAMPLE:** `{LOW, MEDIUM, HIGH}` used separately for alerts and for volume is acceptable repetition — they represent different things. Two functions that both format a datetime the same way are not — extract a utility.

- **Apply SOLID and YAGNI pragmatically, not as dogma.** Use design patterns and abstractions only when there are multiple concrete cases or callers to justify them.
  - **EXAMPLE:** Introducing an abstract factory is only warranted when there are multiple concrete types that must be substituted; applying it to a single implementation violates YAGNI.

- **Optimise code for the reader.** Every function, class, and variable should do exactly one thing and be named to make that thing obvious without reading the body.
  - **EXAMPLE:** A function named `process()` forces the reader to trace the entire body to understand intent. Renaming it `calculate_turnaround_delay()` makes intent obvious and reduces the need for inline comments.
  - **EXAMPLE:** Complex logic with nested iteration or more than 2-3 levels of if-else should be broken into multiple functions with clear names that explain each step, rather than left as one long block.
  - **EXAMPLE:** Directory and file structure should mirror the code's dependency graph — shared utilities live in a higher-level location, and files at the deepest level of nesting are typically not imported anywhere else.

- **Write comments that explain why, not what.** Comments should capture information the code cannot express on its own — intent, trade-offs, or external constraints — not restate what the next line already shows.
  - **EXAMPLE:** `# retry once: upstream API has a known transient failure rate` is useful. `# loop over items` above a `for` loop is not.

- **Cover new or changed behavior with automated tests.** Tests should assert observable behavior and contracts, not internal implementation details, so they survive refactors.
  - **EXAMPLE:** A test for `calculate_turnaround_delay()` should assert the returned delay for given inputs, not assert that a specific private helper was called internally.

- **Use the minimum code needed, without sacrificing readability.**
  - **EXAMPLE:** If we have a function that takes in a list of items and returns a list of the same items with some transformation, we should use a list comprehension rather than a for loop with an append statement. However, if the transformation is complex and requires multiple steps, it may be more readable to use a for loop with clear variable names for each step of the transformation.
  - **EXAMPLE:** If we have a deeply nested if-else statement, we should consider refactoring it into multiple functions with clear names that explain the purpose of each step, rather than trying to condense it into a single function with multiple levels of nesting.

- **Manage coupling deliberately.** Minimise dependencies between components, but do not over-abstract.
  - **EXAMPLE:** If we have a function that needs to access a database, it should not directly import the database client and make queries itself. Instead, it should call a separate function or class that handles all database interactions. However, if the function only needs to make one or two simple queries, it may be more efficient to allow it to interact with the database directly rather than creating an additional layer of abstraction.
  - **EXAMPLE:** If we need a small utility function to transform a data object, we should avoid importing a large library that has this function as one of its many features, and instead write the small utility function ourselves. However, if we need a wide range of functionality that is provided by a well-maintained library, it may be more efficient to use that library rather than implementing ourselves.
  - **EXAMPLE:** If we need to write code to interface with a 3rd-party API, we should try to create this code in a way that is completely separate from the rest of our codebase -- in this way, we can easily reuse this interface in other components without worrying about importing unneeded code. However, this CAN result in overly-abstract interfaces. If it seems like this decoupled code is getting too large relative to the rest of the codebase, this can mean that we are over-abstracting and should allow some coupling between the API interface code and the rest of the codebase.

## Security

**Verify every item below before approving or writing code that handles external input:**

- [ ] All database queries with user input use parameterised queries or prepared statements — never string concatenation or interpolation.
- [ ] No secrets, API keys, or connection strings are hardcoded. Configuration flows through the project's designated settings mechanism (environment variables, secrets manager, config file).
- [ ] All external inputs (HTTP, CLI, file, message) are validated through the project's input validation mechanism (schema, DTO, validator) before use.
- [ ] Authorisation is checked at every entry point via the project's designated permissions helper. Never trust the request body for user identity.
- [ ] Logged values do not include secrets, tokens, or PII.
- [ ] File paths derived from user input are validated against a known root (no path traversal).

## Anti-Patterns — Do Not Do These

- **DO NOT** add abstractions for a single caller. Wait for the second use site (Rule of Three).
- **DO NOT** rename, reformat, or refactor code outside the scope of the requested change. Only make changes that are directly requested or clearly necessary.
- **DO NOT** add exception-handling blocks for failure modes that cannot occur given the inputs. Validate at system boundaries only.
- **DO NOT** add metadata (e.g., comments, type information, annotations) to code you did not modify.
- **DO NOT** introduce a new dependency before checking the project's dependency manifest for an existing one that solves the problem.
- **DO NOT** catch broad/base exception types without re-raising or logging with full context.

## Definition of Done

Before reporting completion, verify all of the following:

- [ ] The change is the minimum needed to satisfy the request (no drive-by refactors).
- [ ] No duplicated logic was introduced. Where duplication remains, it represents independent concepts.
- [ ] All new public functions and classes have names that explain intent without reading the body.
- [ ] New or changed behavior has automated test coverage, or a stated reason why testing is not feasible.
- [ ] No secrets, credentials, or user-controlled data flow into a string-formatted query, command, or path.
- [ ] The project's lint, format, and type-check commands all pass for the changed files.
