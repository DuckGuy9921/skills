---
name: readme-writer
description: Use when the user invokes /readme-writer or asks to write, create, improve, or update a project README, including install, usage, configuration, or contributing sections that must match the real repo.
---

# README Writer

Write or improve a project's README.md so every install, usage, and configuration instruction matches the real code and has been run successfully.

## Steps

1. **Locate the target.** Run `git rev-parse --show-toplevel` to find the repo root; if that fails, use the current working directory. Use `Glob` for `README*` at the root. If a README exists, `Read` it in full before changing anything, and keep its accurate content, badges, and license text.

2. **Inventory the project.** Run `ls -la` at the root and `Read` each manifest that exists: `package.json`, `pyproject.toml`, `setup.py`, `setup.cfg`, `Cargo.toml`, `go.mod`, `Makefile`, `Dockerfile`, `docker-compose.yml`. Also read `CONTRIBUTING.md`, `LICENSE`, `.nvmrc`, `.python-version`, and `.github/workflows/*.yml` if present. The CI workflow shows the commands maintainers actually run.

3. **Collect install commands.** Take install and build commands from manifest scripts or CI steps, not from memory. Record the required runtime version from `engines`, `requires-python`, `.nvmrc`, or the toolchain file.

4. **Find real entry points.** Check `bin` in `package.json`, `[project.scripts]` in `pyproject.toml`, `__main__.py`, `main.go`, and `src/main.rs`. Grep for `argparse`, `click`, `typer`, `commander`, or `cobra` to find CLI flags. In the copy from step 6, run `--help` on each entry point to confirm the real flags and subcommands before documenting them.

5. **Document configuration from code.** Grep for environment variable reads: `process.env`, `os.environ`, `os.getenv`, `std::env::var`, `os.Getenv`. Document only variables the code reads, with defaults taken from the code. Compare against `.env.example` and list any mismatch under Assumptions.

6. **Verify every command in a copy.** Create a temporary directory with `mktemp -d`, then copy the step 1 repo root into it with `rsync -a --exclude .git --exclude node_modules <root>/ <tmp>/`. Run each install, build, test, and usage command there, so the working tree stays clean. Skip deploy, publish, release, push, and destructive commands and mark them unverified. Record the exit code and a one-line result for each. If a command needs credentials or network services, or runs longer than about five minutes, mark it unverified instead of claiming it works. If subagents are available, independent commands may run in parallel on `model: haiku`.

7. **Write the README.** Create or edit `README.md` at the repo root with these sections, in this order, omitting any the repo does not support: title and one-sentence description; Requirements; Installation; Usage (verified examples, with short real output where useful); Configuration (table of variable, default, purpose); Development (test and lint commands); Contributing (from `CONTRIBUTING.md` or the verified test command); License (from `LICENSE`). Do not add features, badges, or URLs the repo does not contain.

8. **Check the result.** Confirm every relative link and file path mentioned exists, using `Glob` or `Read`. Run `git diff --stat`, then re-read the full README once for wrong flags, stale names, and leftover placeholder text.

## Output format

Reply with exactly these sections:

**README:** absolute path to `README.md`, and whether it was created or updated.

**Verified commands:** a table with columns `Command`, `Result` (pass, fail, or unverified), and `Note`.

**Assumptions:** bullets for each guess made, such as an inferred Python version or a missing license file. Write "None" if there were none.

**Sections:** one line listing the sections written or changed.

## Notes

- Edit only `README.md`. Do not modify source, manifests, lockfiles, or other docs, and do not create `CONTRIBUTING.md` or other files unless the user asks.
- Never run deploy, publish, release, push, or destructive commands (`rm -rf`, database migrations) while verifying.
- Do not install global tools or change the global environment. Use a local venv or the project's own package manager inside the copy.
- If a command fails, show the failure in the table. Document an alternative only if you verified it. Do not hide failures.
- If the directory is not a project at all (no manifest, no code, no README), ask the user one question about which project to document, then stop.
- If the repo holds several packages, document the root unless the user names one. If the user asks for one section only, edit only that section.
- For very large repos, rely on manifests and CI files rather than reading whole source trees.
- Keep the README short. Prefer one working example over long prose.
