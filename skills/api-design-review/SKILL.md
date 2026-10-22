---
name: api-design-review
description: Use when the user invokes /api-design-review or asks to review a REST, RPC, or library API design, OpenAPI spec, proto file, or public interface for naming, consistency, versioning, errors, pagination, idempotency, or backward compatibility.
---

# API Design Review

Review a REST, RPC, or library API design and report concrete problems, with a verdict.

## Steps

1. **Locate the design.** Use the text after `/api-design-review` as the target: a file path, a directory, or a branch name. For a directory, use `find` for `*openapi*.{yaml,yml,json}`, `*.proto`, and route files. For a branch, run `git diff --name-only <base>...<branch>` without checking the branch out, using the base from step 2. With no target, run `git diff --name-only $(git merge-base HEAD <base>)` and keep the API-facing files. If nothing is found, ask one clarifying question and stop.

2. **Establish the baseline.** Base is `main`, or the default branch if `main` does not exist. Get each changed file's previous version with `git show <merge-base>:<file>`, where merge-base is `git merge-base HEAD <base>`. If the file is absent at the merge base, or the target is not in git, state that the API is new or unbaselined and skip the breaking-change classification in step 8.

3. **Check naming.** List every resource, path, field, parameter, method, and error code. Flag mixed casing in one surface (camelCase next to snake_case), verbs in REST paths such as `/getUser`, singular and plural mixed for the same resource, and one concept given two names. Use `grep -rn` to find every variant.

4. **Check consistency.** Compare endpoints of the same kind: ID format, list and single-item envelope shape, date and time format, boolean and enum conventions, and HTTP method for the same action.
5. **Check errors.** Confirm each error has a stable machine-readable code and a human message, and that status codes match meaning (400 vs 422, 404, 409). Flag 5xx responses returned for client-caused errors, and any error a client would have to parse from free text.

6. **Check collections.** Every list endpoint needs a default and maximum page size, an opaque cursor or a documented offset, and a stated sort order. Flag unbounded lists, offset pagination that skips or duplicates items under concurrent inserts, and cursors that expose internal IDs.

7. **Check idempotency.** For each POST, PUT, PATCH, DELETE, and RPC mutation, decide whether a retry is safe. Check for idempotency keys on creates, that PUT replaces the whole resource, that a repeated DELETE returns a documented result, and that the spec states whether PATCH merges or replaces.

8. **Check versioning and compatibility.** Against the baseline from step 2, classify each change. Breaking: removed or renamed field or endpoint, changed type, newly required input, changed default, narrowed output, changed error code or status. Additive: new optional field, new endpoint, new enum value only where clients are documented to ignore unknown values. Check where the version lives (path, header, or package) and whether breaking changes bump it.

9. **Verify each finding.** Cite file and line for every finding (`grep -n` or open the file). Drop any finding you cannot point to in the source.

10. **Rank.** Order findings: breaking changes first, then errors and idempotency, then pagination, then naming and consistency.

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

Severity: high, medium, or low. Area: naming, consistency, errors, pagination, idempotency, or versioning. Verdict `block` only for a breaking change to a released API or a defect where a safe retry causes duplicate writes or data loss. Use `approve with changes` for any other finding, including a breaking change to an unreleased API, and `approve` for none.

## Assumptions
- Baseline used, files reviewed, and anything not checked.
```

## Notes

- Review only. Do not edit files or post PR comments unless asked.
- Do not invent endpoints, fields, or behavior. Note any conclusion that depends on unread code under Assumptions.
- Do not report style preferences. Report only issues that cause client bugs, ambiguity, breakage, or inconsistency with the rest of the same API.
- A consistency finding must cite the majority pattern it deviates from.
- For a library, treat exported functions and types as the endpoints.
- Over about 30 endpoints: review the changed ones plus a sample, and say so under Assumptions.
