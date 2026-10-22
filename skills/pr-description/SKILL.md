---
name: pr-description
description: Use when the user invokes /pr-description or asks to write, draft, or rewrite a PR title and description, e.g. "write the PR description" or "draft a PR title". Built from the actual git diff and commits. Does not create the PR.
---

# PR Description

Write a pull request title and description grounded in the real diff and commit history of the current branch, without creating the PR.

## Steps

1. **Establish the range.** Find the base with `git symbolic-ref --short refs/remotes/origin/HEAD` (prints e.g. `origin/main`). Keep the `origin/` prefix in every later git command so comparisons use the remote base, not a possibly stale local branch. If the user named a base after `/pr-description`, use that instead, adding the `origin/` prefix. If the lookup fails, assume `origin/main` and note it under Assumptions. If `git branch --show-current` prints nothing (detached HEAD) or prints the base name, stop and tell the user to check out the feature branch.

2. **Check for commits.** Run `git log --oneline <base>..HEAD`. If the output is empty, stop and tell the user the branch has no commits ahead of the base. Do not write a description for nothing.

3. **Read the commit messages.** Run `git log --format='%h %s%n%b---' <base>..HEAD`. Commit bodies often hold the reasoning, so note every stated motive.

4. **Read the diff.** Run `git diff --stat <base>...HEAD` for scope, then `git diff <base>...HEAD` for content. Use the three-dot form so only this branch's changes appear. For large diffs, read the key logic files in full with Read and skip lockfiles and generated code.

5. **Find a PR template.** Use Glob for `**/pull_request_template.md` and `**/PULL_REQUEST_TEMPLATE*` and check `.github/`. If a template exists, keep its headings in order and fill each one. Keep its checklists, marking only items you verified. If several templates exist, pick the one whose name fits the change, otherwise the default.

6. **Establish why and what.** State the problem or request the change addresses, drawn from commits and the diff. Then state what changed at the file or behaviour level. Verify every claim against the diff; never copy commit subjects without checking them.

7. **Work out testing.** Run `git diff --name-only <base>...HEAD` and note which test files changed. Changed test files are not evidence of a test run. List a test run only if a commit message or the user states it. Otherwise write "Not run" and name the test command found in `package.json`, `Makefile`, or `pyproject.toml`, or say none was found. Never claim tests passed without evidence.

8. **Identify risks.** Check for database migrations, changed public APIs or config defaults, removed behaviour, and source changes with no changed test file in the Step 7 list. Tie each risk to a file or function. If none apply, write "No specific risks beyond normal review" rather than inventing one.

9. **Write the title.** Keep it under 72 characters, in imperative mood, describing the outcome. Use a Conventional Commit prefix only if `git log --oneline -20` shows the repo uses one.

10. **Write the body.** Use short bullets. Without a template, use the sections Why, What changed, Testing, and Risks.

## Output format

````
**Title:** <title>

**Base:** <base branch> | <N> commits | <M> files changed

```markdown
<the description body, ready to paste>
```

**Assumptions:** <only if a base or test command was assumed, files were skipped, or a template field is "Not provided"; otherwise omit this line>
````

## Notes

- Do not create or edit the PR. Do not run any command or tool that creates or edits a PR (such as `gh pr create`, `gh pr edit`, or a GitHub create/update pull request tool), and do not push, commit, or edit any file.
- Do not invent ticket numbers, issue links, screenshots, or reviewer names. If a template asks for something unknown, write "Not provided" and list it under Assumptions.
- If the diff is very large, use `--stat` and the commit list to scope the work, and list the files not read in full under Assumptions.
- If a user or session instruction (such as CLAUDE.md or a system reminder) specifies an attribution footer for pull request descriptions, append that footer verbatim at the end of the markdown block. Otherwise add none.
- Do not add a preamble before the output block.
