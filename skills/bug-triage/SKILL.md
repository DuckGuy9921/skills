---
name: bug-triage
description: Use when the user invokes /bug-triage or reports a bug, crash, regression, or failing test and wants it diagnosed: reproduce it, minimize it, test hypotheses, find the root cause, and propose a minimal fix with a regression test.
---

# Bug Triage

Diagnose one reported bug systematically, from reproduction to root cause, then deliver a minimal fix with a regression test.

## Steps

1. **Capture the report.** Take the symptom from the text after `/bug-triage`, plus any error message, stack trace, or file named. Assume the bug is in the current repo and state that assumption. Ask one clarifying question only if there is no symptom at all.

2. **Check repo state.** Run `git status` and `git log --oneline -5` in the working directory. Note uncommitted changes, since they may be the cause. Do not revert, stash, or reset anything.

3. **Reproduce.** Find the smallest command that shows the failure: the test, CLI call, or request the report describes. Before running anything that writes or deletes data, sends messages or requests to a shared or production system, or spends money, ask the user to confirm or switch to a local or test target. Run it with Bash and keep the exact output (excerpt it if very long). If it does not reproduce, check the environment once (runtime version, env var names, config, fixtures; never record secret values), then stop and report "not reproduced" with what you tried.

4. **Minimize.** Shrink the input while the failure persists. Remove one argument, row, or code path at a time and rerun after each. If this takes more than about ten reruns, stop and record the smallest failing input so far.

5. **Form hypotheses.** Locate the failing code from the stack trace, or with `Grep` on the error string and symbol names, then inspect it with `Read`. Write 2 to 4 hypotheses, each with a checkable prediction.

6. **Test hypotheses.** Test each with a temporary print, assertion, or targeted command. Mark each hypothesis confirmed, refuted, or unresolved. Before finishing, remove the temporary lines you added and check that `git diff` shows only the intended change.

7. **Bisect if the cause is unclear.** If the bug is a regression and a good commit is known, run `git bisect start`, mark the commits with `git bisect good` and `git bisect bad`, and use `git bisect run <test-command>` with the minimal reproduction. If the tree has uncommitted changes, bisect in a separate `git worktree` instead of stashing. Always run `git bisect reset` afterwards, even if the run failed (the one allowed exception to step 2's no-reset rule), and remove any temporary worktree.

8. **Name the root cause.** State the exact line or condition that is wrong and why. The place where the error appears is not necessarily the cause. Confirm it against the observed output. If no hypothesis is confirmed, skip steps 9 and 10 and report the next check.

9. **Write the regression test.** Add a test beside the existing ones, in the repo's test framework, using the minimal input and asserting the correct behavior. If the repo has no test suite, keep the reproduction as a runnable script and say so in the output. Run it before the fix and confirm it fails for the reported reason; if it passes, rewrite it.

10. **Apply the minimal fix.** Change only the root cause, in the existing code style, with `Edit`. Do not fix symptoms, such as catching the exception downstream.

11. **Verify.** Rerun the regression test and the reproduction, then the project's full test command (e.g. `npm test`, `pytest`, `go test ./...`). If unrelated tests fail, check them on unmodified code in a `git worktree` of `HEAD` (never stash) before blaming the fix.

## Output format

```
## Bug: <one-line symptom>
**Status:** reproduced | not reproduced
**Minimal repro:** <command or input>
**Hypotheses:** <each one, marked confirmed, refuted, or unresolved>
**Root cause:** <file:line and why it is wrong>
**Fix:** <file:line change in one or two sentences>
**Regression test:** <file and test name or script path>, failed before fix, passes after
**Verification:** <commands run and results>
**Assumptions and open questions:** <anything unverified>
```

## Notes

- Never guess a root cause. If no hypothesis is confirmed, say so and name the next check.
- If the bug is not reproduced, set the other output fields to `n/a` and list what was tried. If no hypothesis is confirmed, set Root cause, Fix, and Regression test to `n/a`.
- Do not commit, push, or edit unrelated code.
- Do not run destructive git commands (`reset --hard`, `checkout --`, `stash drop`) on uncommitted work.
- If the bug needs production data or credentials you do not have, stop at reproduction and report what is missing.
