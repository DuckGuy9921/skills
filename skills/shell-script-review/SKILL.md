---
name: shell-script-review
description: Use when the user invokes /shell-script-review or asks to review, harden, or fix a bash or sh script: set -euo pipefail, quoting, shellcheck warnings, traps, portability, dangerous rm or globs, idempotency.
---

# Shell Script Review

Review one or more shell scripts for correctness and safety, apply minimal hardening fixes, and report what changed.

## Steps

1. **Locate the scripts.** Use the text after `/shell-script-review` as a path or glob. If it is empty, run `git diff --name-only $(git merge-base HEAD origin/main)` and keep the `.sh` files and files with a `#!` shell shebang. If that command errors (for example, no `origin/main`) or finds nothing, run `git ls-files '*.sh'`. If still nothing, ask one question naming the file you need, then stop.

2. **Read each script in full.** Use Read on the whole file. Note the shebang: `bash` (including `/usr/bin/env bash`), or `sh` (including `/bin/sh`). Scripts declared as `sh` must not use bash-only features such as `[[ ]]`, arrays, `${var,,}`, `pipefail`, or `source`.

3. **Run static checks.** Run `bash -n FILE` for syntax. Run `shellcheck -x FILE` if `command -v shellcheck` succeeds. For `sh` scripts, also run `checkbashisms FILE` if it is installed. If a tool is missing, write that in the report and do the matching checks by hand.

4. **Check strict mode.** Bash scripts should have `set -euo pipefail` directly after the shebang. Do not add it blindly to a script that deliberately checks `$?` or tolerates failures. Flag that case and explain the trade-off instead.

5. **Check quoting and expansion.** Confirm every `$var`, `"$@"`, and `$(...)` is quoted. Replace `for f in $(ls ...)` with a glob loop. Use `read -r`. Use `"$@"`, not `$*`.

6. **Check deletions, moves, and globs.** For every `rm`, `mv`, `chmod -R`, `chown -R`, `find -delete`, and `> file` redirect, confirm the path cannot expand to empty or to `/`. Require `${VAR:?message}` for variables used in destructive paths. In bash loops over globs, check for `shopt -s nullglob` where an empty match would change behavior.

7. **Check temp files and traps.** Temp files must come from `mktemp`, not fixed names in `/tmp`. Cleanup belongs in `trap '...' EXIT`. Check that the trap preserves the script's exit status and does not mask errors.

8. **Check portability.** Compare the constructs used against the shebang and the platforms the repo targets (look at CI config or README). Flag GNU-only options such as `sed -i`, `readlink -f`, `date -d`, and `grep -P` only when the script runs where they are unavailable.

9. **Check idempotency.** Re-running must be safe. Use `mkdir -p`, `ln -sfn`, and `cp -f` where appropriate. Guard `>>` appends to config files with a `grep -q` check. Do not fail on state that already exists.

10. **Apply minimal fixes.** Edit only the problems you found, using Edit. Keep the script's behavior and style. Do not rewrite working code. Do not add `# shellcheck disable` without a comment giving the reason.

11. **Verify.** Re-run `bash -n` and `shellcheck -x` on each edited file. Never execute a script that deletes, installs, or contacts a network service to test it. For safe checks, copy the script into a temporary directory (`mktemp -d`) and run it there with destructive commands stubbed.

12. **Parallelise large batches.** If there are more than five scripts and subagents are available, review them with subagents using `model: haiku`, one script per subagent, each returning findings only. Otherwise review them one at a time.

## Output format

```
## Shell script review

**Scripts reviewed:** <paths>
**Verdict:** <ready | minor fixes applied | needs changes>

### Findings
| File | Line | Severity | Issue | Fix |
|------|------|----------|-------|-----|

### Changes applied
- <file>: <one line per change>

### Not changed
- <file>: <issue left open and why>

### Checks run
- bash -n: <result>
- shellcheck: <result, or "not installed">
```

## Notes

- Report only findings you verified in the file. Never invent shellcheck codes or line numbers.
- Severity is `high` for destructive paths, unquoted expansion in commands, and missing strict mode. Use `medium` for portability and traps. Use `low` for style.
- Do not change behavior, command output format, or public arguments while hardening. The one intended exception is adding strict mode under step 4, which can make a script exit earlier; mention that in the report.
- Do not install tools without the user's permission. Report the missing tool instead.
- Skip generated files, vendored code, and files under `.git`.
