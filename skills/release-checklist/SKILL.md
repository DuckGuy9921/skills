---
name: release-checklist
description: Use when the user invokes /release-checklist or asks to check if a release is ready, bump the version, verify tests or CI, review migrations, draft a changelog, tag a release, or plan rollout, rollback, and post-release monitoring.
---

# Release Checklist

Verify a release candidate's real state in the repo and CI, prepare the version bump, changelog, and rollout plan, and perform tagging or publishing only after explicit confirmation.

## Steps

1. **Confirm the repo state.** Run `git status --short` and `git branch --show-current`. If the working tree is dirty, list the files and stop until they are committed or stashed. Find the last release tag with `git describe --tags --abbrev=0`. List unreleased commits with `git log <last-tag>..HEAD --oneline`. If there are none, report nothing to release and stop.

2. **Locate every version declaration.** Search with `grep -rn` for `"version"` in `package.json`, `pyproject.toml`, `setup.cfg`, `Cargo.toml`, `version.txt`, and any `__version__` or `VERSION` constant. Record each file and value. If the values disagree, report them and stop before editing anything.

3. **Propose the bump.** Derive the bump from the unreleased commit subjects. Use `!` or `BREAKING CHANGE` for major, `feat` for minor, and anything else for patch, if the project follows Conventional Commits. Otherwise read the last changelog entry for the convention. State the proposed version and its reason. Do not edit files yet.

4. **Verify tests and CI.** Read the test, lint, and typecheck commands from `package.json` scripts, `Makefile`, `tox.ini`, or `.github/workflows/`. Run them locally. Then run `gh run list --branch <branch> --limit 5` and `gh run view <run-id>` for the release commit. A failing or pending run is a blocker, and you must name the failing job. If no tests exist, report that as a gap, not a pass.

5. **Review migrations.** Run `git diff --name-status <last-tag>..HEAD` restricted to `migrations/`, `alembic/versions/`, `db/migrate/`, or `prisma/migrations/`, whichever exists. For each new migration, flag destructive operations (`DROP`, renames, `NOT NULL` without default) and missing down steps as rollout risks.

6. **Draft the changelog.** Read the top of `CHANGELOG.md` to match its format. Group unreleased commit subjects under Added, Changed, Fixed, Removed, and Breaking. Show the draft in the reply and do not write it to the file yet.

7. **Plan rollout and rollback.** Use only deploy mechanisms found in the repo (scripts, Makefile targets, Helm charts, workflows). State the deploy order, whether migrations run before or after code, any feature flags (grep for flag config), the previous artifact or version to roll back to, and the signals that trigger rollback. If the repo has no deploy mechanism, say so rather than inventing one.

8. **Request confirmation.** Present the version, files to change, changelog draft, tag name, and publish target. Ask the user to confirm the exact version string. Stop here until they reply. Only after an explicit yes, edit the version files, commit with `git commit`, run `git tag -a v<version> -m "Release v<version>"`, and push with `git push origin <branch> v<version>`. Run the publish command only if the user approved it. Confirmation covers only the version it names.

9. **Verify and monitor after release.** Confirm the tag exists with `git ls-remote --tags origin v<version>`. Check the release workflow with `gh run list --limit 3`. Confirm the artifact is visible in its registry, such as `npm view <pkg> version`. Then list the health and error signals the repo or user identifies, with a watch window (default 60 minutes).

## Output format

```
## Release readiness: <version or "undetermined">
**Verdict:** READY or BLOCKED (one-line reason)

| Check | Status | Evidence |
|---|---|---|
| Repo state | pass/fail | <git output summary> |
| Version sources | pass/fail | <files and values> |
| Tests and lint | pass/fail/gap | <commands and exit codes> |
| CI on release commit | pass/fail/pending | <run id and job> |
| Migrations | pass/risk/none | <files and flags> |

## Changelog draft
<the drafted entry>

## Rollout and rollback
<deploy order, rollback target, trigger signals>

## Next step
<the one confirmation needed, or "Released v<version>" with the verification results>
```

## Notes

- Verify state with commands. Never assume a test passed, a CI run is green, or a tag exists.
- Never tag, push, publish, or create a GitHub release without explicit confirmation in this conversation.
- Do not modify files before step 8 is confirmed.
- If the version source is ambiguous, ask one clarifying question and stop.
