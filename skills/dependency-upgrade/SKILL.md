---
name: dependency-upgrade
description: Use when the user invokes /dependency-upgrade or asks to upgrade, bump, or update packages safely, such as "upgrade deps", "bump major versions", "check changelogs before updating", "update the lockfile", or "roll back the upgrade".
---

# Dependency Upgrade

Upgrade dependencies one unit at a time, checking changelogs and tests after each, so any breakage traces to one package and can be rolled back.

## Steps

1. **Read the scope.** Treat the text after `/dependency-upgrade` as the scope: a package name (only that package), `major` (only major-version candidates), or empty (all patch and minor updates). Search for a manifest (`package.json`, `pyproject.toml`, `requirements*.txt`, `go.mod`, `Cargo.toml`). If none exists, ask one question and stop. If several exist (a monorepo), ask which to upgrade unless the scope names a package found in just one.

2. **Identify the tool.** Match the lockfile: `package-lock.json` means npm, `pnpm-lock.yaml` pnpm, `yarn.lock` yarn, `uv.lock` or `poetry.lock` Python, `go.sum` Go, `Cargo.lock` Cargo. Use only that tool. Check the `packageManager` field in `package.json` and `.github/workflows` for the expected version.

3. **Record the baseline.** Run `git status --short`. If dependency files already have uncommitted changes, stop and tell the user. Save `git rev-parse HEAD` as the rollback point. Run the project's test, lint, typecheck, and build scripts (from `package.json` `scripts`, a Makefile, or CI) and note which already fail. Record a check as `not configured` if none exists; do not invent one.

4. **List and classify candidates.** Run `npm outdated`, `pip list --outdated`, `go list -m -u all`, or `cargo outdated` (requires `cargo install cargo-outdated`), as appropriate. Label each entry patch, minor, or major by comparing semver. Run `npm audit` or the equivalent and move flagged packages to the front.

5. **Read changelogs.** For each candidate, find release notes: the repository from `npm view axios repository.url`, then its release notes with `gh release view v1.7.2 -R axios/axios`, or the package's own `CHANGELOG.md`. Look for "BREAKING", "removed", "renamed", "deprecated", and raised minimum runtime versions. Search the repo for each affected API. If subagents are available, spawn one per package with `model: haiku` to do this in parallel; each returns only breaking changes that touch this repo. Otherwise, work through the packages in order.

6. **Order the upgrades.** Apply patch and minor updates first. Group only packages that must move together, such as `@babel/*`. Each major version goes alone, last.

7. **Upgrade one unit at a time.** Use the package manager with an explicit version, such as `npm install axios@1.7.2`, so the manifest and lockfile update together. Never hand-edit a lockfile.

8. **Check after each unit.** Run the step 3 checks. On failure, read the error and the matching changelog entry before changing anything else.

9. **Fix fallout narrowly.** If the changelog requires a small code change, change only the affected call sites. Otherwise revert that unit (see step 12) and mark it skipped. Never edit test expectations unless the changelog documents the behavior change.

10. **Inspect the lockfile diff.** Run `git diff --stat` and read the lockfile changes: only targeted packages and their transitive dependencies should move. Revert any unrelated version change by reinstalling that package at its original version with the package manager.

11. **Verify from clean.** Delete `node_modules` or the virtualenv only if it is gitignored and regenerable, then reinstall with the frozen command (`npm ci`, `pnpm install --frozen-lockfile`, `uv sync --locked`, `cargo build --locked`), then run all baseline checks.

12. **Roll back on failure.** Use this when step 11 still fails after skipping fallout units. Restore only the files this run changed (they were clean at baseline, per step 3), for example `git checkout -- package.json package-lock.json`, or reinstall the old version of one package. Never run `git reset --hard` or `git checkout .`, which would discard other work.

## Output format

- **Scope:** the scope and any assumption made.
- **Baseline:** checks that passed and failed before any change.
- **Upgraded:** a table with columns `Package`, `From`, `To`, `Type`, `Result` (passed, reverted, or skipped).
- **Breaking changes handled:** one bullet per change, with its changelog entry and the file and line changed.
- **Final checks:** the clean-install result, then test, lint, typecheck, and build outcomes with counts.
- **Needs a decision:** skipped majors, with the reason and the release-notes link.

## Notes

- Do not commit or push unless the user asks; leave changes in the working tree.
- Do not upgrade the language runtime as a side effect. If a package needs a newer runtime, skip it and report the requirement.
- If a changelog cannot be found, say so and mark the package unverified. Quote only what you actually read.
- Compare later failures against the baseline. Only new failures count as upgrade fallout.
