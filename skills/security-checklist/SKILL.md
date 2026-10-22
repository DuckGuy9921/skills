---
name: security-checklist
description: Use when the user invokes /security-checklist or asks for a defensive security review of code changes, a diff, branch, or PR, covering injection, authn/authz, secrets, input validation, SSRF, deserialization, and dependency risk.
---

# Security Checklist

Review code changes defensively against common vulnerability classes and report each finding with a severity and a concrete fix.

## Steps

1. **Scope the change.** Use the path, branch, or PR the user names. Otherwise find the base branch with `git symbolic-ref refs/remotes/origin/HEAD` (fallback `main`, then `master`), run `git diff --name-only <base>...HEAD`, and add uncommitted files from `git status --short`. If there is no git repo and no target, ask one question and stop.

2. **Read changed files.** Use Read on each changed source file in full, not just the hunks. Skip lockfiles and generated or binary files, except for the dependency check in step 9.

3. **Check injection.** Use Grep on changed files for `exec|system\(|shell=True|eval\(|innerHTML|SELECT .*\+|f"SELECT`. For each hit, trace whether untrusted input reaches it and whether parameterized queries or argument arrays are used.

4. **Check authn and authz.** For each new or changed route, handler, or RPC method, confirm an auth check runs before any data access and that object lookups are scoped to the caller's identity, not a client-supplied ID. Flag disabled or skipped middleware.

5. **Check secrets.** Use Grep with `api[_-]?key|secret|token|password|PRIVATE KEY` (case-insensitive) on added lines, and run `git diff --cached --name-only` to catch staged credential files. Ignore placeholders and test fixtures. Redact any real value to its first four characters.

6. **Check input validation.** For each new entry point, confirm type, length, and range checks exist. Confirm paths built from input are normalized and confined to an allowed directory, and that HTML output is escaped.

7. **Check SSRF.** Grep for outbound calls (`fetch\(|requests\.|urlopen|axios|http\.get`). If the URL comes from input, confirm scheme and host allowlisting, and that private and link-local ranges (127.0.0.0/8, 10.0.0.0/8, 169.254.169.254) are blocked, including after redirects.

8. **Check deserialization.** Grep for `pickle\.load|yaml\.load\(|unserialize\(|ObjectInputStream|Marshal\.load`. Flag any call that consumes untrusted data, and any `yaml.load` without a safe loader.

9. **Check dependencies.** If a manifest or lockfile changed, list added or upgraded packages from the diff. Run the ecosystem audit if installed (`npm audit --omit=dev`, `pip-audit`, `cargo audit`, `govulncheck`). If a tool is missing, record that instead of guessing at CVEs.

10. **Verify each candidate.** Keep a finding only if you can name the input source, the path through the changed code, and the sink. Drop anything you cannot trace.

## Output format

```
# Security review: <scope>

Verdict: PASS | PASS WITH NOTES | BLOCK

| # | Severity | Category | Location | Issue | Fix |
|---|----------|----------|----------|-------|-----|

## Checked, no findings
- <category>: <one-line reason>

## Not run
- <check>: <reason>
```

Severity is Critical, High, Medium, Low, or Info. Any Critical or High finding gives BLOCK. Location is `path:line`. If there are no findings, leave the table empty and say so under the verdict.

## Notes

- Defensive review only. Do not write exploits, send payloads, or contact live hosts or endpoints.
- Review the changed code and its direct callers, not the whole repo, unless the user asks.
- Mark uncertain findings "needs confirmation" and state what would confirm them.
- Propose fixes in the report. Do not edit files unless the user asks.
- Skip style issues and generic advice. Every finding needs a file and line.
- If nothing has changed, say so and stop.
