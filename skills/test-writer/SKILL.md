---
name: test-writer
description: Use when the user invokes /test-writer or asks to write unit or integration tests for existing code, add test coverage, or cover edge cases and error paths for a file, function, class, or module.
---

# Test Writer

Write unit or integration tests for existing code by matching the repo's own test framework and conventions, then run them until they pass.

## Steps

1. **Identify the target.** Use the text after `/test-writer` as the target: a file, directory, function, class, or module name (find symbols with Grep). If it is empty, use changed source files from `git diff --name-only HEAD` and `git ls-files --others --exclude-standard`, skipping deleted files, docs, lockfiles, and generated code. If nothing is found, ask one clarifying question and stop. If a named path or symbol does not exist, report that and stop.

2. **Read the code under test.** Read each target file in full (in sections if very large). List each exported function or method with its inputs, outputs, thrown errors, and branches, and note external dependencies (network, database, filesystem, clock, environment) that need mocking.

3. **Discover the test framework.** Check `package.json` (`scripts.test` and jest, vitest, or mocha in devDependencies), `pyproject.toml` or `pytest.ini`, `go.mod`, `Cargo.toml`, and `Gemfile`. Find existing tests with `**/*.test.*`, `**/*.spec.*`, `**/test_*.py`, `**/*_test.go`, and `**/tests/**` (use a file-search tool if available, otherwise `find`). Read one or two and copy their naming, location, imports, and assertion style. If there are none, use the framework's default layout and record it under Assumptions.

4. **Check the baseline.** Run the existing suite with the discovered command (for example `npm test`, `pytest -q`, or `go test ./...`) before adding anything. Record existing failures without fixing them unless they block your tests. If the suite needs live services, a database, or credentials, ask the user first.

5. **Plan the cases.** For each unit cover: happy path (typical valid input and expected output); edges (empty, zero, null or None, single element, maximum size, unicode, off-by-one); errors (invalid input, thrown exceptions, rejected promises, error return values). Use realistic domain values, not `foo` or `bar`. Write integration tests only if the repo already has them and the code crosses a module or I/O boundary; mock network and external services anyway.

6. **Write the test file.** Put it where the repo's existing tests live, using the same name pattern. If that file already exists, add cases to it rather than overwriting it. Mock only true external dependencies, using the mocking tool the repo already uses. Each test checks one behavior and has a descriptive name.

7. **Run the new tests.** Run only the new file, using the repo's test command with a path (for example `npx vitest run <path>`, `pytest <path> -q`, or `go test ./pkg -run <Name>`). If a test's expectation is clearly wrong, correct it to the intended behavior, judged from names, docs, and callers, never to the current output. If the source looks buggy or you cannot tell which side is wrong, keep the assertion for the intended behavior and record a suspected bug.

8. **Run the full suite and linter.** Run the complete command from step 4 and compare with the baseline to confirm nothing else broke. If the repo configures a linter or formatter (such as `ruff check`, `gofmt -l`, or `npm run lint`), run it and fix only issues in the new file.

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

- Never change source code to make a test pass. Never delete, skip, or weaken an existing test, or loosen an assertion to match buggy output.
- Do not add test dependencies or config files unless the user asks. If the repo has no framework, use the built-in runner (`node:test` for Node, `unittest` for Python, `go test` for Go) and state it as an assumption. Use `pytest` only if it is already installed.
- Test public behavior, not private details. Skip trivial getters and generated code. Never let tests hit the real network or services; mock them and keep runs fast.
- For more than about three files, you may spawn one `model: haiku` subagent per file after steps 1 to 4, passing them the framework and baseline results. Each writes only its own test file and does not run the full suite; you run step 8 once at the end.
