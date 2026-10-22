---
name: user-story-writer
description: Use when the user invokes /user-story-writer or asks to turn a feature idea into user stories with Given/When/Then acceptance criteria, edge cases, an out-of-scope list, and sizing hints.
---

# User Story Writer

Turn a feature idea into testable user stories with acceptance criteria, edge cases, out-of-scope items, and sizing hints.

## Steps

1. **Get the idea.** Use the text after `/user-story-writer` as the feature idea. If there is no text, ask the user one clarifying question (what feature they want to specify) and stop until they answer. Do not ask more than one question. If the text names several unrelated features, write stories for the first one only and list the others as out of scope.

2. **Ground in the repo.** If the working directory is a git repo, run `git status` to see uncommitted work. Then search for the idea's key nouns (routes, components, models, field names) with Grep and Glob, or any equivalent search tool. Read the matching files and any changed files that match the idea. If nothing matches, or the directory is not a code repo, record that and continue. Record what already exists so stories extend it instead of duplicating it. Never describe current behavior you have not read.

3. **Identify actors.** List each distinct user role the idea names or implies. If none is named, assume one primary user and state that assumption in the output.

4. **Write the stories.** One story per user goal, in the form `As a <actor>, I want <capability>, so that <benefit>.` If a story needs two independent behaviors, split it. Write at most 8 stories; list any extra as out of scope.

5. **Write acceptance criteria.** Give 2 to 5 Given/When/Then criteria per story. Each `Then` must name one observable result a test can assert: a status code, displayed text, a stored value, or a changed record. Replace vague words such as "fast", "easy", or "user-friendly" with an observable outcome. Use a number only if the user or the repo supplies it; otherwise list the number under Assumptions as one to confirm.

6. **Add edge cases.** For each story, check these categories and keep only those that apply: empty input, invalid input, permission denied, repeated or concurrent action, network or service failure, and limits (max length, max count). Give each kept case its expected result.

7. **Set out of scope.** List capabilities the idea could imply but this work excludes, each with a one-line reason.

8. **Size each story.** Use XS (one file, copy or config change), S (one component plus tests), M (several files in one layer), L (crosses frontend and backend or adds a data model), or XL (too big). Mark every L and XL story as needing a split and give the split as the smaller stories that replace it. Each split story must be S or smaller, and split stories count toward the 8-story limit; list any stories beyond that limit as out of scope.

9. **Recheck testability.** Reread every criterion. If any `Then` has no concrete expected result, rewrite it before output.

## Output format

```
# <Feature name>

**Summary:** <one sentence>
**Assumptions:** <bullets, or "None">
**Repo context:** <files found and what they do, or "No existing code found.">

## Story 1: <short title>

As a <actor>, I want <capability>, so that <benefit>.

**Acceptance criteria**
- Given <state>, when <action>, then <observable result>.

**Edge cases**
- <case>: <expected result>

**Size:** <XS|S|M|L|XL> (<reason>; for L or XL, add "Split into:" with one line per smaller story, each in the story sentence form and sized S or smaller)

(repeat for each story)

## Out of scope
- <item>: <reason>
```

## Notes

- Write the output in chat. Do not create a file unless the user asks for one.
- Do not invent APIs, file names, field names, or existing behavior. If the repo does not show it, say it is unknown.
- Do not write implementation code or propose a technical design beyond the sizing hint.
- If the idea spans more than three separate areas of the codebase and subagents are available, spawn one subagent per area with `model: haiku` to gather repo context in parallel. Subagents only read and report file paths and what they do; they must not edit files. Merge the results yourself. Otherwise search the areas one after another.
- Keep edge cases tied to a story. Do not add generic checklists that do not fit the feature.
