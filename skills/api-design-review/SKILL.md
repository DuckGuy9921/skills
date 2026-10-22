---
name: api-design-review
description: Use when the user invokes /api-design-review or asks to review a REST, RPC, or library API design, OpenAPI spec, proto file, or public interface for naming, consistency, versioning, errors, pagination, idempotency, or backward compatibility.
---

# API Design Review

Review a REST, RPC, or library API design and report concrete problems, with a verdict.

## Steps

1. **Locate the design.** Use the text after `/api-design-review` as the target: a file path, a directory, or a branch name. For a file, open it. For a directory, use `find` for `*openapi*.{yaml,yml,json}`, `*.proto`, and route files. For a branch, diff it against the base with `git diff --name-only <base>...<branch>`. With no target, run `git diff --name-only $(git merge-base HEAD <base>)` and keep the API-facing files. If nothing is found, ask one clarifying question and stop.

2. **Establish the baseline.** Get the previous version of each changed API file with `git show <base>:<file>`, where base is the merge base with `main`. Compatibility is judged against that version. If there is none, state that the API is new and skip the breaking-change check in step 8.

3. **Check naming.** List every resource, path, field, parameter, method, and error code. Flag mixed casing in one surface (camelCase next to snake_case), verbs in REST paths such as `/getUser`, singular and plural mixed for the same resource, and one concept given two names. Use `grep -rn` to find every variant.

4. **Check consistency.** Compare endpoints of the same kind against each other: ID format, list and single-item envelope shape, date and time format, boolean and enum conventions, and HTTP method for the same action. The majority pattern is the reference.

5. **Check errors.** Confirm each error has a stable machine-readable code and a human message, and that status codes match meaning (400 vs 422, 404, 409). Flag 500 responses for client mistakes and any error a client would have to parse from free text.

6. **Check collections.** Every list endpoint needs a default and maximum page size, an opaque cursor or a documented offset, and a stated sort order. Flag unbounded lists and cursors that break under concurrent inserts or expose internal IDs.

7. **Check idempotency.** For each POST, PUT, PATCH, DELETE, and RPC mutation, decide whether a retry is safe. Check for idempotency keys on creates, PUT as full replacement, a clear result when deleting an already-deleted resource, and PATCH merge-versus-replace semantics stated in the spec.

8. **Check versioning and compatibility.** Against the baseline from step 2, classify each change. Breaking: removed or renamed field or endpoint, changed type, newly required input, changed default, narrowed output, changed error code or status. Additive: new optional field, new endpoint, new enum value only where clients are documented to ignore unknown values. Check where the version lives (path, header, or package) and whether breaking changes bump it.

9. **Verify each finding.** Cite the file and line for every finding with `grep -n` or Read. Drop any finding you cannot point to in the source.

10. **Rank.** Order findings: breaking changes to released APIs first, then errors and idempotency, then pagination, then naming and consistency.

## Output format

```
# API design review: <target>

Verdict: <approve | approve with changes | block>. <one sentence on why>

## Breaking changes
- <file:line>: <change>. Impact: <which clients break>. Fix: <suggestion>
(Or: "None against baseline <ref>.")

## Findings
| Severity | Area | Location | Issue | Suggested fix |
|---|---|---|---|---|

Severity: high, medium, or low. Area: naming, consistency, errors, pagination, idempotency, or versioning.

## Assumptions
- Baseline used, files reviewed, and anything not checked.
```

## Notes

- Review only. Do not edit files or post PR comments unless the user asks.
- Do not invent endpoints, fields, or behavior. If a conclusion depends on code you did not read, say so under Assumptions.
- Do not report style preferences. Report only issues that cause client bugs, ambiguity, breakage, or inconsistency with the rest of the same API.
- A consistency finding must cite the majority pattern it deviates from.
- For a library, treat exported functions and types as the endpoints.
- If the target has more than about 30 endpoints, review the changed ones plus a sample of the rest and say so in Assumptions.
