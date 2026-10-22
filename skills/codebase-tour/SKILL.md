---
name: codebase-tour
description: Use when the user invokes /codebase-tour or asks for an onboarding map of an unfamiliar repo, such as "where do I start", "explain this codebase", "how is this repo laid out", or "how do I build and test it".
---

# Codebase Tour

Produce a verified onboarding map of a repository: layout, entry points, build, test and run commands, architecture, conventions, hotspots, and where a newcomer should start.

## Steps

1. **Confirm the repo root.** Run `git rev-parse --show-toplevel` in the working directory. If that fails, use the working directory when it holds source files, and state the assumption in the output. If the user named a path, scope the whole tour to that path and say so.

2. **Map the layout.** Run `git ls-files | cut -d/ -f1-2 | sort | uniq -c | sort -rn | head -30`. Skip vendored or generated trees (`node_modules`, `vendor`, `dist`, `build`, `.venv`).

3. **Find entry points.** Read whichever manifests exist: `package.json`, `pyproject.toml`, `setup.py`, `Cargo.toml`, `go.mod`, `pom.xml`, `build.gradle`, `Makefile`, `Dockerfile`, `docker-compose.yml`. Record `main`, `bin` and `scripts` fields, `[project.scripts]`, `[[bin]]` tables, and `func main` locations found with `grep -rln "func main" --include=*.go`. Read the first 60 lines of each entry file.

4. **Extract commands.** Collect build, test, lint and run commands from manifests, `Makefile` targets, `.github/workflows/*.yml` and the README. Label each command with the file it came from. For a tool you are unsure about, check it exists with `command -v` and read its `--help`. Do not run install, build or test commands.

5. **Read the docs.** Read `README*`, `CONTRIBUTING*`, any `docs/` index, and `CLAUDE.md` or `AGENTS.md` if present. Mark each doc claim you cannot confirm in code as unverified.

6. **Trace the architecture.** Start from one entry point and follow calls or imports through three to five files with Read and `grep -rn`. Name each layer (routing, services, storage) and its directory. For a repo over about 2,000 tracked files, you may use one `model: haiku` subagent per top-level directory, each capped at 150 words.

7. **Capture conventions.** Read two or three representative source files and one test file. Note naming, error handling, config loading, test location and framework, and formatter or linter config (`.editorconfig`, `ruff.toml`, `.eslintrc*`, `.prettierrc`, `rustfmt.toml`). Report only what those files show.

8. **Find hotspots.** Run `git log --since="6 months ago" --name-only --pretty=format: | sort | uniq -c | sort -rn | head -15` for churn. Run `git ls-files -z | xargs -0 wc -l | sort -rn | head -15` for size. Run `git grep -c -E "TODO|FIXME|HACK" | sort -t: -k2 -rn | head -10` for debt markers. Name the signal behind each hotspot. If the repo has no git history, use size and debt markers only and say so.

9. **Choose a starting path.** Pick three to five files to read in order, each with a one-line reason, and one small first task, such as a test to read or a function to trace. Every path must exist in the repo.

## Output format

Return these sections in order:

- Title `# Codebase tour` plus the repo directory name, then a one- or two-sentence summary.
- `## Layout`: a bulleted list of the top directories with their purpose.
- `## Entry points`: bulleted `path:line` references with what runs there.
- `## Commands`: a table with columns `Purpose`, `Command`, `Source`.
- `## Architecture`: one short paragraph, then an ordered list of layers with paths.
- `## Conventions`: short bullets.
- `## Hotspots`: bullets of `path` with the signal that flagged it.
- `## Where to start`: an ordered list of three to five files and one first task.
- `## Unverified`: doc claims code did not confirm, or "None".

## Notes

- Every fact needs a file read or command output. Otherwise, list it under Unverified.
- The tour is read-only. Do not edit files, run installs, or start servers. Checking `--version` or `--help` is fine.
- Do not write a report file.
- Do not narrate each directory. Keep the layout to eight to ten entries.
- Ask one question only if the working directory holds no code and no path was given; otherwise assume it and state that.
