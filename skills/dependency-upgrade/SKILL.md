---
name: dependency-upgrade
description: Use when the user invokes /dependency-upgrade or asks to upgrade, bump, or update packages safely, such as "upgrade deps", "bump major versions", "check changelogs before updating", "update the lockfile", or "roll back the upgrade".
---

# Dependency Upgrade

Upgrade dependencies one unit at a time, checking changelogs and tests after each, so any breakage traces to one package and can be rolled back.

## Steps

1. **Read the scope.** Treat the text after `/dependency-upgrade` as the scope: a package name (that package only), `major` (major-version candidates only), or empty (all patch and minor updates). Find the manifest (`package.json`, `pyproject.toml`, `requirements*.txt`, `go.mod`, `Cargo.toml`). If none exists, ask one question and stop. If several exist, ask which to upgrade unless the scope names a package found in only one.

2. **Identify the tool.** Match the lockfile: `package-lock.json` means npm, `pnpm-lock.yaml` pnpm, `yarn.lock` yarn, `uv.lock` or `poetry.lock` Python, `go.sum` Go, `Cargo.lock` Cargo. Use only that tool. If `packageManager` in `package.json` or a CI workflow names a different version than the installed one, stop and ask.

3. **Record the baseline.** Run `git status --short`. If dependency files have uncommitted changes, stop and tell the user. Save `git rev-parse HEAD` for the report. Run the project's test, lint, typecheck, and build scripts (from `package.json`, a Makefile, or CI) and note which already fail. Record a missing check as `not configured`; do not invent one.

4. **List and classify candidates.** Run the outdated command for the tool (`npm outdated`, `pip list --outdated`, `go list -m -u all`, or `cargo outdated` if that subcommand is installed). Label each entry patch, minor, or major by semver. Run the audit (`npm audit` or the equivalent) and mark flagged packages.

5. **Read changelogs.** For each candidate, find release notes: the repository from `npm view <pkg> repository.url` (or the equivalent), then the notes for the target version via `gh release view <tag> -R <owner>/<repo>` if `gh` is available, or the package's `CHANGELOG.md`. Look for "BREAKING", "removed", "renamed", "deprecated", and raised minimum runtime versions, then search the repo for each affected API. If subagents are available, run one per package in parallel with `model: haiku`, each returning only the breaking changes that touch this repo. Otherwise, work through the packages in order.

6. **Order the upgrades.** A unit is one package, or a group that must move together (such as `@babel/*`). Apply patch and minor units first, with audit-flagged units earliest. Each major unit goes alone, last.

7. **Upgrade one unit at a time.** Use the tool's explicit-version command, such as `npm install axios@1.7.2`, `uv add "requests==2.32.3"`, or `go get example.com/mod@v1.2.3`, so the manifest and lockfile update together. Never hand-edit a lockfile.

8. **Check after each unit.** Run the step 3 checks. Only failures that are new compared with the baseline count as upgrade fallout. For each one, read the error and the matching changelog entry before changing anything else.

9. **Fix fallout narrowly.** If the changelog requires a small code change, change only the affected call sites, and never edit test expectations unless the changelog documents the behavior change. Otherwise revert the unit by reinstalling its previous version with the step 7 command, and mark it skipped.

10. **Inspect the lockfile diff.** Run `git diff --stat` and read the lockfile changes. Only the targeted packages and their transitive dependencies should move. Pin any unrelated change back to its original version with the step 7 command.

11. **Verify from clean.** Delete `node_modules` or the virtualenv only if it is gitignored and regenerable. Reinstall with the frozen command (`npm ci`, `pnpm install --frozen-lockfile`, `uv sync --locked`, `cargo build --locked`), then run all baseline checks.

12. **Roll back if still broken.** If new failures remain after skipped units are reverted, restore only the files this run changed (they were clean at baseline, per step 3), such as `git checkout -- package.json package-lock.json`. Never run `git reset --hard` or `git checkout .`, which would discard other work. Then rerun the checks and report.

## Output format

- **Scope:** the scope and any assumption made.
- **Baseline:** checks that passed and failed before any change, plus the HEAD saved in step 3.
- **Upgraded:** a table with columns `Package`, `From`, `To`, `Type`, `Result` (passed, reverted, or skipped).
- **Breaking changes handled:** one bullet per change, with its changelog entry and the file and line changed.
- **Final checks:** the clean-install result, then test, lint, typecheck, and build outcomes with counts.
- **Needs a decision:** skipped majors, with the reason and the release-notes link.

## Notes

- Do not commit or push unless the user asks; leave changes in the working tree.
- Do not upgrade the language runtime as a side effect. If a package needs a newer runtime, skip it and report the requirement.
- If a changelog cannot be found, say so and mark the package unverified. Quote only what you actually read.
