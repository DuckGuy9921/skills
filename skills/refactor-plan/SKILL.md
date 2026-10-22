---
name: refactor-plan
description: Use when the user invokes /refactor-plan or asks to plan a safe incremental refactor, restructure, or migration of existing code, such as splitting a module, renaming an API, or replacing a library, with tests first and reversible steps.
---

# Refactor Plan

Produce a safe, incremental refactor plan that characterizes current behavior with tests before any code changes, breaks the work into small reversible steps, and states blast radius, ordering, and rollback.

## Steps

1. **Pin down the target.** Take the text after `/refactor-plan` as the refactor goal. If there is none, ask one clarifying question (what to refactor and what must stay the same), then stop. If the goal is named but vague, assume the narrowest reading that matches the code you find, and state that assumption in the output.

2. **Locate the code.** Run `git status` and `git log --oneline -5` to confirm the tree is clean and see recent churn. If it has uncommitted changes, list those files under Assumptions, because the plan's baseline is then unclear. If it is not a git repo, say so under Assumptions and skip the git baseline. Use Grep and Glob to find the target symbols, files, and call sites. Record each path with line numbers. Do not guess paths.

3. **Map the blast radius.** For each target symbol, run Grep for its name across the repo, including tests, configs, and docs. List every caller, public export, and external consumer you can see. Find test files with Glob (for example `**/*test*` and `**/*spec*`) and mark each caller as tested if a test file imports it or reaches it through a public entry point, otherwise untested.

4. **Find characterization gaps.** Read the target code (for large files, only the target symbols and their immediate callers) and identify behaviors a refactor could change: return shapes, error paths, side effects, ordering, default values, and timing. For each uncovered behavior, specify a characterization test that asserts the current observable behavior (output, errors, side effects), not the desired behavior. Name the test runner you actually find in package.json, pyproject.toml, Makefile, or similar. If none exists, make adding one the first step.

5. **Sequence small steps.** Split the refactor so each step leaves the suite green and can be committed alone. Order them: add characterization tests, add the new code path beside the old one, switch callers one group at a time, then delete the old path. Give each step one line: change, files touched, verification command. Rollback is added in step 6.

6. **Define verification and rollback.** For each step, state the exact command that proves it, taken from the repo (test, type check, or lint). Write these commands down only; do not run them. State the rollback: usually `git revert <commit>` for a single step, or a feature flag if the change ships in one deploy. Flag any step that changes data, schemas, or public APIs and cannot be undone by reverting code alone, and name what must be coordinated.

7. **Use subagents only for large surveys.** If a subagent tool is available and the call-site survey spans many independent directories, spawn one subagent per directory with `model: haiku` to list call sites. Otherwise, or for small surveys, search directly.

## Output format

```
# Refactor plan: <goal>

**Assumptions:** bullets, or "None"

**Blast radius:**
| Symbol | Callers | Tested |
|---|---|---|

**Characterization tests:** numbered list, each with file path and what it asserts

**Steps:** numbered list, each as `change | files | verify command | rollback`

**Risks:** bullets for non-reversible steps and open questions

Recommend starting with step 1.
```

## Notes

- This skill produces a plan only. Do not edit source files, and run only read-only commands (no tests, migrations, or anything that writes, deletes, or pushes). The user decides when to execute.
- Characterize current behavior even when it looks wrong. Record suspected bugs separately instead of fixing them inside the refactor.
- Never invent test commands, file paths, or callers. If a check cannot be found or run, write "unverified" next to it.
- If a step touches more than about 10 files, split it.
- Do not mix behavior changes with structural moves in one step.
- If the goal spans dozens of call sites or several modules, run the Step 3 survey and Step 4 gap analysis only for the first module, and list the remaining modules as follow-up plans.
