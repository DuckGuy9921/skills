---
name: test-writer
description: Use when the user invokes /test-writer or asks to write unit or integration tests for existing code, add test coverage, or cover edge cases and error paths for a file, function, class, or module.
---

# Test Writer

Write unit or integration tests for existing code by matching the repo's own test framework and conventions, then run them until they pass.

## Steps

1. **Identify the target.** Use the text after `/test-writer` as the target: a file path, function, class, or module name. If it is empty, use the changed files from `git diff --name-only HEAD` plus untracked files from `git ls-files --others --exclude-standard`. If that is also empty, ask one clarifying question and stop. If a named path or symbol does not exist, report that and stop.

2. **Read the code under test.** Read each target file in full (in sections if very large). List every exported function or method with its inputs, return values, thrown errors, and branches. Note external dependencies (network, database, filesystem, clock, environment variables) that will need mocking.

3. **Discover the test framework.** Check `package.json` (`scripts.test` and devDependencies such as jest, vitest, mocha), `pyproject.toml` or `pytest.ini` (pytest), `go.mod` (`go test`), `Cargo.toml` (`cargo test`), and `Gemfile` (rspec or minitest). Search for existing tests with `**/*.test.*`, `**/*.spec.*`, `**/test_*.py`, `**/*_test.go`, and `**/tests/**` (using a file-search tool if available, otherwise `find`). Read one or two of them and copy their file naming, location, import style, and assertion style. If the repo has no existing tests, use the framework's default layout and record that under Assumptions.

4. **Check the baseline.** Run the existing suite with the discovered command (for example `npm test`, `pytest -q`, or `go test ./...`) before adding anything. Record any failures that already exist. Do not fix them unless they block your new tests. If the suite needs live services, a database, or credentials, ask the user before running it.

5. **Plan the cases.** For each unit, cover three groups. Happy path: typical valid input and its expected output. Edges: empty, zero, null or None, a single element, maximum size, unicode, and off-by-one boundaries. Errors: invalid input, thrown exceptions, rejected promises, and error return values. Use realistic domain values, not `foo` or `bar`. Write integration tests only when the code crosses a module or I/O boundary and the repo already has integration tests.

6. **Write the test file.** Put it where the repo's existing tests live, using the same name pattern. If that file already exists, add cases to it rather than overwriting it. Mock only true external dependencies with the mocking tool the repo already uses. Each test checks one behavior, with a descriptive name.

7. **Run the new tests.** Run only the new file first (for example `npx vitest run <path>`, `pytest <path> -q`, or `go test ./pkg -run <Name>`). If a test fails because its expectation was wrong, correct the expectation to the intended behavior, judged from names, docs, and callers, not from the current output. If it fails because the source has a bug, keep the assertion for the intended behavior and record it as a suspected bug.

8. **Run the full suite and linter.** Run the complete test command from step 4 to confirm nothing else broke. If the repo configures a linter or formatter (such as `npm run lint`, `ruff check`, or `gofmt -l`), run it on the new file.

## Output format

```
## Tests added
- <path to new test file> (<N> tests)

## Coverage
- <unit>: happy path (<case>), edges (<cases>), errors (<cases>)

## Results
- Command: <test command>
- Full suite: <passed>/<total> passing
- Pre-existing failures: <none, or list>

## Suspected bugs
- <test name>: <one-line description>, or "None"

## Assumptions
- <framework or command assumptions, or "None">
```

## Notes

- Never change source code to make a test pass. Never delete, skip, or weaken an existing test, and never loosen an assertion to match buggy output.
- Do not add test dependencies or config files unless the user asks. If the repo has no test framework, use the language's standard runner (`node:test` for Node, `pytest` for Python, `go test` for Go), and state that as an assumption.
- Test public behavior, not private implementation details. Skip trivial getters and generated code.
- Do not let tests hit the real network or real services. Mock them, and keep runs fast.
- If the target spans more than about three files and subagents are available, you may spawn one subagent per file with `model: haiku` so they run in parallel. Each subagent writes only its own test file. Do the framework discovery once and pass its results to each subagent.
