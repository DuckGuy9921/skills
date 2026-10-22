---
name: changelog-writer
description: Use when the user invokes /changelog-writer or asks for release notes, a changelog, or a summary of what changed between git tags, grouped Keep-a-Changelog style with breaking changes flagged.
---

# Changelog Writer

Generate user-facing release notes from the git history between two refs, grouped into Keep a Changelog sections with breaking changes flagged at the top.

## Steps

1. **Resolve the range.** Take `FROM..TO` from the text after `/changelog-writer`. If only one ref is given, use it as TO. If nothing is given, set TO to `HEAD`. Then find FROM: if TO is a tag (HEAD counts when it is tagged), FROM is the next tag after it in `git tag --merged TO --sort=-creatordate` (the first line is TO itself); otherwise FROM is the latest tag reachable from TO (`git describe --tags --abbrev=0 TO`). Confirm each ref with `git rev-parse --verify <ref>`. If a ref does not resolve, ask the user once for the correct ref and stop. If no earlier tag exists, or the repo has no tags, use the root commit (`git rev-list --max-parents=0 HEAD`) as FROM and state that assumption in the output.

2. **List the commits.** Run `git log --no-merges --format='%h%x09%s%x09%b%x1e' FROM..TO` to get hash, subject and body for each commit. Pull PR numbers from `(#123)` suffixes in subjects. If the range has more than 300 commits, ask the user once whether to narrow it before continuing.

3. **Inspect unclear commits.** For a commit whose subject is vague (`wip`, `fix stuff`, `update`) or that touches a public surface, run `git show --stat <hash>` and read the relevant diff before classifying it. If more than about 40 commits need this and subagents are available, split them across subagents on a cheaper model, each returning one line per commit.

4. **Detect breaking changes.** Mark a commit breaking only if one of these holds: its body has a `BREAKING CHANGE:` footer, its subject has `!` before the colon (such as `feat!:` or `fix(api)!:`), or its diff removes or renames a public export, CLI flag, config key, environment variable, API route or file format. Confirm the last case with `git show <hash> -- <path>` and look for removed lines on the public surface. Do not mark a change breaking on a guess.

5. **Classify.** Map `feat` to Added, `fix` to Fixed, `perf` and `refactor` with behaviour changes to Changed, deprecations to Deprecated, removals to Removed, and security fixes to Security. Drop `chore`, `ci`, `test`, `style`, `build`, behaviour-neutral `refactor` and docs-only commits unless they change user-visible behaviour. Count the dropped commits for the output. A breaking commit goes only under Breaking changes, never also under its type section.

6. **Rewrite for users.** Turn each kept commit into one line that describes its effect for a user. Start with a verb, remove hashes and internal module names, and merge commits that describe the same change. Append `(#123)` when a PR number exists.

7. **Set the version and date.** Use the version the user gives, otherwise the tag on TO as written, or `Unreleased` when TO is not a tag. Take the date from `git log -1 --format=%cs TO`. Do not choose a semver bump unless asked. If asked, suggest major for breaking changes, minor for Added entries, and patch otherwise, labelled as a suggestion.

8. **Flag what you cannot classify.** List any commit you could not place confidently under `Needs review:` rather than guessing a section.

## Output format

First line: `Range: FROM..TO, N non-merge commits, M omitted as internal.`

Then one markdown block:

```
## [VERSION] - YYYY-MM-DD

### Breaking changes
- **BREAKING:** entry (#PR)

### Added
- entry

### Changed
- entry

### Deprecated
### Removed
### Fixed
### Security
```

Omit any heading with no entries, and omit Breaking changes when there are none. End with `Needs review:` followed by each unclassified commit as `short-hash subject`, only if there are any.

## Notes

- Print the changelog in chat. Do not edit `CHANGELOG.md` or create files unless the user asks. If they do, insert the block directly below the file's `# Changelog` heading; if there is no such heading, print the block and say so instead of inserting.
- Use only facts from the git history. Never invent features, versions, dates or PR numbers.
- Write for users, not developers: no refactor names, file paths or hashes in entries.
- If the range contains no non-merge commits, say so and stop without printing an empty block.
- Do not publish, tag or push anything as part of this command.
