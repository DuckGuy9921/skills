# skills
A repository for skills and design resources

## Structure
- `skills/` - Skills we make live here, one folder per skill.

## Skills
| Skill | When to use |
|---|---|
| `/api-design-review` | the user invokes /api-design-review or asks to review a REST, RPC, or library API design, OpenAPI spec, proto file, or public interface for naming, consistency, versioning, errors, pagination, idempotency, or backward compatibility. |
| `/bug-triage` | the user invokes /bug-triage or reports a bug, crash, regression, or failing test and wants it diagnosed: reproduce it, minimize it, test hypotheses, find the root cause, and propose a minimal fix with a regression test. |
| `/changelog-writer` | the user invokes /changelog-writer or asks for release notes, a changelog, or a summary of what changed between git tags, grouped Keep-a-Changelog style with breaking changes flagged. |
| `/code-explainer` | the user invokes /code-explainer or asks to explain unfamiliar code, a file, or a function, e.g. "how does this work", "walk me through X", "trace the entry point", "what does this module do". |
| `/codebase-tour` | the user invokes /codebase-tour or asks for an onboarding map of an unfamiliar repo, such as "where do I start", "explain this codebase", "how is this repo laid out", or "how do I build and test it". |
| `/commit-message` | the user invokes /commit-message or asks to write, draft, or suggest a commit message for staged changes, or says "commit message for this", "what should I commit", or "split these changes into commits". |
| `/company-meeting` | the user invokes /company-meeting or wants a simulated company meeting where role-based agents (CEO, CTO, Product, Design, Marketing, Finance) discuss a topic and produce decisions and action items. |
| `/csv-data-cleaner` | the user invokes /csv-data-cleaner or asks to "clean this CSV", "profile this spreadsheet", or check a CSV or XLSX for nulls, duplicates, wrong types, outliers or encoding problems. Never overwrites the original. |
| `/decision-memo` | the user invokes /decision-memo or asks for a one-page decision memo or lightweight ADR, such as "write up this decision", "compare these options and recommend one", or "record why we chose X over Y". |
| `/dependency-upgrade` | the user invokes /dependency-upgrade or asks to upgrade, bump, or update packages safely, such as "upgrade deps", "bump major versions", "check changelogs before updating", "update the lockfile", or "roll back the upgrade". |
| `/dockerfile-review` | the user invokes /dockerfile-review or asks to review a Dockerfile, docker-compose file, or .dockerignore for layer caching, multi-stage builds, image size, non-root users, pinned tags, secrets, or healthchecks. |
| `/email-drafter` | the user invokes /email-drafter or asks to draft, write, or reply to a professional email, such as "draft an email to my manager", "reply to this thread", "write a follow-up", or "help me decline politely". Produces a draft only, never sends. |
| `/meeting-notes` | the user invokes /meeting-notes or pastes a raw meeting transcript, recording dump, or messy notes and wants a summary, decisions, action items with owners and dates, and open questions. |
| `/perf-investigation` | the user invokes /perf-investigation or asks why a command, test, endpoint, function, query or build is slow, wants a performance baseline, a profile of a bottleneck, or proof that a speedup works with before and after numbers. |
| `/postmortem` | the user invokes /postmortem or asks for a blameless incident postmortem, outage review, or incident writeup with timeline, impact, root cause, 5 whys, and action items with owners. |
| `/pr-description` | the user invokes /pr-description or asks to write, draft, or rewrite a PR title and description, e.g. "write the PR description" or "draft a PR title". Built from the actual git diff and commits. Does not create the PR. |
| `/readme-writer` | the user invokes /readme-writer or asks to write, create, improve, or update a project README, including install, usage, configuration, or contributing sections that must match the real repo. |
| `/refactor-plan` | the user invokes /refactor-plan or asks to plan a safe incremental refactor, restructure, or migration of existing code, such as splitting a module, renaming an API, or replacing a library, with tests first and reversible steps. |
| `/regex-builder` | the user invokes /regex-builder or asks to build, write, or fix a regular expression from example strings, e.g. "regex that matches these emails", "regex for dates", "why does my regex match this", or lists of matches and non-matches. |
| `/release-checklist` | the user invokes /release-checklist or asks to check if a release is ready, bump the version, verify tests or CI, review migrations, draft a changelog, tag a release, or plan rollout, rollback, and post-release monitoring. |
| `/security-checklist` | the user invokes /security-checklist or asks for a defensive security review of code changes, a diff, branch, or PR, covering injection, authn/authz, secrets, input validation, SSRF, deserialization, and dependency risk. |
| `/shell-script-review` | the user invokes /shell-script-review or asks to review, harden, or fix a bash or sh script: set -euo pipefail, quoting, shellcheck warnings, traps, portability, dangerous rm or globs, idempotency. |
| `/sql-helper` | the user invokes /sql-helper or asks to write, explain, debug, or optimize SQL, including joins, CTEs, NULL handling, indexes, or reading EXPLAIN plans for Postgres, MySQL, SQLite, or another dialect. |
| `/study-plan` | the user invokes /study-plan or asks for a learning plan, study schedule, or roadmap for a topic, skill, or exam, given a goal, current level, weekly time budget, and optional deadline. |
| `/test-writer` | the user invokes /test-writer or asks to write unit or integration tests for existing code, add test coverage, or cover edge cases and error paths for a file, function, class, or module. |
| `/user-story-writer` | the user invokes /user-story-writer or asks to turn a feature idea into user stories with Given/When/Then acceptance criteria, edge cases, an out-of-scope list, and sizing hints. |
