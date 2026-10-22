---
name: readme-writer
description: Use when the user invokes /readme-writer or asks to write, create, improve, or update a project README, including install, usage, configuration, or contributing sections that must match the real repo.
---

# README Writer

Write or improve a project's README.md so every install, usage, and configuration instruction matches the real code and has been run successfully.

## Steps

1. **Locate the target.** Run `git rev-parse --show-toplevel` to find the repo root, falling back to the current directory. `Glob` for `README*` at the root. If one exists, `Read` it in full and keep its accurate content, badges, and license text.

2. **Inventory the project.** Run `ls -la` at the root and `Read` each existing manifest: `package.json`, `pyproject.toml`, `setup.py`, `setup.cfg`, `Cargo.toml`, `go.mod`, `Makefile`, `Dockerfile`, `docker-compose.yml`. Also read `CONTRIBUTING.md`, `LICENSE`, `.nvmrc`, `.python-version`, and `.github/workflows/*.yml` if present. CI steps show the commands maintainers actually run.

3. **Collect install commands.** Take install and build commands from manifest scripts or CI steps, not from memory. Record the required runtime version from `engines`, `requires-python`, `.nvmrc`, or the toolchain file.

4. **Find real entry points.** Check `bin` in `package.json`, `[project.scripts]` in `pyproject.toml`, `__main__.py`, `main.go`, and `src/main.rs`. Grep for `argparse`, `click`, `typer`, `commander`, or `cobra` to find CLI flags. Record them now; confirm each with `--help` in step 6.

5. **Document configuration from code.** Grep for environment reads: `process.env`, `os.environ`, `os.getenv`, `std::env::var`, `os.Getenv`. Document only variables the code reads, with defaults from the code. Compare against `.env.example` (never copy values from a real `.env`) and list mismatches under Assumptions.

6. **Verify commands in a copy.** Run `mktemp -d` for a temporary directory outside the repo, then `rsync -a --exclude .git --exclude node_modules <root>/ <tmp>/` (or `cp -r` if rsync is missing). Run each install, build, test, usage, and `--help` command there, so the working tree stays clean, and record the exit code and a one-line result. If the repo is not the user's own code, ask before running install or build commands, since install scripts execute code. Skip deploy, publish, release, push, and destructive commands (`rm -rf`, database migrations) and mark them unverified. Also mark unverified any command that needs credentials or network services, or runs longer than about five minutes. Use the project's own package manager or a local venv; install no global tools.

7. **Write the README.** Create or edit `README.md` at the repo root with these sections, in this order, omitting any the repo does not support: title and one-sentence description; Requirements; Installation; Usage; Configuration (table of variable, default, purpose); Development (test and lint commands); Contributing (from `CONTRIBUTING.md` or the verified test command); License (from `LICENSE`). Usage examples must come from verified commands; leave out or label the rest. Do not add features, badges, or URLs the repo does not contain.

8. **Check the result.** Confirm every relative link and file path mentioned exists, using `Glob` or `Read`. Run `git status --short` and `git diff --stat` (a new README appears only in status), then re-read the README once for wrong flags, stale names, and placeholder text.

## Output format

Reply with exactly these sections:

**README:** absolute path to `README.md`, and whether it was created or updated.

**Verified commands:** a table with columns `Command`, `Result` (pass, fail, or unverified), and `Note`.

**Assumptions:** bullets for each guess made, such as an inferred Python version or a missing license file. Write "None" if there were none.

**Sections:** one line listing the sections written or changed.

## Notes

- Edit only `README.md`. Do not modify source, manifests, lockfiles, or other docs, or create `CONTRIBUTING.md` or other files unless the user asks.
- If a command fails, show the failure in the table. Document an alternative only if you verified it.
- If the directory is not a project (no manifest, code, or README), ask the user one question about which project to document, then stop.
- If the repo holds several packages, document the root unless the user names one.
- For very large repos, rely on manifests and CI files rather than whole source trees.
- Prefer one working example over long prose.
