---
name: shell-script-review
description: Use when the user invokes /shell-script-review or asks to review, harden, or fix a bash or sh script: set -euo pipefail, quoting, shellcheck warnings, traps, portability, dangerous rm or globs, idempotency.
---

# Shell Script Review

Review one or more shell scripts for correctness and safety, apply minimal hardening fixes, and report what changed.

## Steps

1. **Locate the scripts.** Use the text after `/shell-script-review` as a path or glob. If it is empty, run `git diff --name-only "$(git merge-base HEAD origin/main)"` and keep `.sh` files and files whose first line is a shell shebang. If that command fails (for example, no `origin/main`) or finds nothing, use `git ls-files '*.sh'` and state in the report that the scope is the whole repository. If that matches more than five scripts, report findings only and ask before editing any. If still nothing, ask one question naming the file you need, then stop.

2. **Read each script in full.** Use Read on the whole file. Note the shebang: `bash` (including `/usr/bin/env bash`) or `sh`. A `sh` script must not use bash-only features such as `[[ ]]`, arrays, `${var,,}`, `pipefail`, or `source`.

3. **Run static checks.** Run `bash -n FILE`. Run `shellcheck -x FILE` if `command -v shellcheck` succeeds. For `sh` scripts, run `checkbashisms FILE` if installed. If a tool is missing, say so in the report and do the matching checks by hand.

4. **Check strict mode.** Bash scripts should have `set -euo pipefail` directly after the shebang. Do not add it to a script that deliberately checks `$?` or tolerates failures; flag that case and explain the trade-off instead.

5. **Check quoting and expansion.** Confirm every `$var`, `"$@"`, and `$(...)` is quoted. Replace `for f in $(ls ...)` with a glob loop, use `read -r`, and use `"$@"`, not `$*`.

6. **Check deletions, moves, and globs.** For every `rm`, `mv`, `chmod -R`, `chown -R`, `find -delete`, and `> file` redirect, confirm the path cannot expand to empty or to `/`. Require `${VAR:?message}` for variables in destructive paths. Add `shopt -s nullglob` where an empty glob would change behavior.

7. **Check temp files and traps.** Temp files must come from `mktemp`, not fixed names in `/tmp`. Cleanup belongs in `trap '...' EXIT`, which must keep the script's exit status and must not hide errors.

8. **Check portability.** Compare constructs against the shebang and the platforms the repo targets (CI config or README). Flag GNU-only options such as `sed -i`, `readlink -f`, `date -d`, and `grep -P` only where they would fail.

9. **Check idempotency.** Re-running must be safe. Use `mkdir -p` and `ln -sfn`. Use `cp -f` only for files the script owns, never over user data. Guard `>>` appends to config files with a `grep -q` check. Do not fail on state that already exists.

10. **Apply minimal fixes.** Edit only the problems found, using Edit. Keep behavior, output format, and style. Do not rewrite working code. Do not add `# shellcheck disable` without a comment giving the reason. Subagents from step 12 only report; the main agent makes every edit. Skip this step if step 1 said to ask first.

11. **Verify.** Re-run `bash -n` and `shellcheck -x` on each edited file. Never execute a script that deletes, installs, or contacts a network service. To test a safe script, run a copy from `mktemp -d` with `PATH` pointing at stub commands for `rm`, `mv`, `curl`, and similar tools.

12. **Parallelise large batches.** If there are more than five scripts and subagents are available, give each script to its own subagent using `model: haiku`. Each subagent returns findings only and edits nothing. Otherwise review them one at a time.

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
- Severity is `high` for destructive paths, unquoted expansion in commands, and missing strict mode in a bash script that does not deliberately tolerate failures. Use `medium` for portability and traps, `low` for style.
- Do not change behavior while hardening. The one intended exception is strict mode under step 4, which can make a script exit earlier or fail on unset variables; say so in the report.
- Do not install tools without the user's permission. Report the missing tool instead.
- Skip generated files, vendored code, and files under `.git`.
