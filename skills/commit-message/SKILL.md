---
name: commit-message
description: Use when the user invokes /commit-message or asks to write, draft, or suggest a commit message for staged changes, or says "commit message for this", "what should I commit", or "split these changes into commits".
---

# Commit Message

Draft a clear commit message for the staged changes, matching the commit style already used in this repository.

## Steps

1. **Check the staged set.** Run `git diff --cached --stat` and then `git diff --cached` from the repository root. If nothing is staged, run `git status --short`, report that nothing is staged, and stop. Do not stage files yourself. If the staged diff is very large (for example, more than about 1,500 changed lines), work from the `--stat` output and the hunks for the most significant files, and say in the `Assumed:` line which files you did not read in full. If the diff contains apparent secrets, credentials, or private keys (such as API keys, tokens, or `-----BEGIN PRIVATE KEY-----` blocks), stop, name the file, and warn the user instead of drafting a message.

2. **Read the repo conventions.** Run `git log -n 30 --format='%s'` and note whether subjects use a Conventional Commits prefix (`feat:`, `fix:`), a scope, a ticket ID, imperative mood ("Add" rather than "Added"), and typical length. Run `git config --get commit.template` and read that file if it exists. Match the dominant pattern. If the repository has no commits yet, `git log` fails; use a plain imperative subject and say so in the `Assumed:` line.

3. **Decide whether the changes form one commit.** Group the staged files by concern. If they mix unrelated work, such as a bug fix with a dependency bump, or a refactor with a feature, list each group with its files. Suggest unstaging with `git restore --staged .` and restaging each group with `git add <paths>`. Never suggest `git reset --hard`, `git checkout -- <path>`, or `git restore` without `--staged`, because those discard working-tree changes. Do not run these commands.

4. **Understand each change.** Read the hunks, not only the file names. Where the diff alone does not explain a change, read the staged version with `git show :<path>` and the previous version with `git show HEAD:<path>`. Do not read the working-tree file, because it can hold unstaged edits that are not part of this commit. Describe what changed. Describe why only when the diff or code shows it; otherwise leave the why out rather than guessing.

5. **Write the subject.** One line, imperative mood, at most 72 characters, no trailing period. Follow the style from step 2. It must describe the whole staged change, or the single group if you split.

6. **Write the body.** Leave one blank line after the subject. Wrap lines at 72 characters. Use short bullets when there are several distinct changes. Call out breaking changes or required migration steps if the diff shows them, such as a changed public signature or a removed config key. Omit the body for a trivial one-line change.

7. **Add trailers only when required.** If the user's CLAUDE.md, memory, or session instructions require trailer lines such as `Co-Authored-By` or `Claude-Session`, append them after a blank line. Add no other trailers.

8. **Stop before committing.** Show the message. Do not run `git commit` unless the user asks for it.

## Output format

Reply with no preamble, in this order:

1. One fenced `text` block per commit, each holding the exact message to pass to git. If there is one commit, show one block. If you suggested a split, show one block per group, in the order of the split list in item 2.
2. Only if you suggested a split, a `## Suggested split` heading followed by a numbered list. Each item reads `<subject> (<files>)`.
3. Only if you made an assumption, one line starting with `Assumed:`.

## Notes

- Never run `git add`, `git restore`, `git reset`, or `git commit` on your own. This skill drafts; the user decides.
- Do not fall back to unstaged changes silently. If nothing is staged, say so and stop.
- If `git log` shows no clear convention, use a plain imperative subject with no prefix and state that in the `Assumed:` line.
- Do not paste the full diff or each file's contents into the answer. Keep the answer to the message and any split.
- Do not describe changes you did not see in the diff. Verify every claim in the body against the hunks.
