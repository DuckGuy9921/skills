---
name: dockerfile-review
description: Use when the user invokes /dockerfile-review or asks to review a Dockerfile, docker-compose file, or .dockerignore for layer caching, multi-stage builds, image size, non-root users, pinned tags, secrets, or healthchecks.
---

# Dockerfile Review

Review the repository's Docker build and compose files against common image-quality rules, then report prioritized findings with concrete fixes.

## Steps

1. **Locate the files.** From the repo root, run `git ls-files | grep -iE '(^|/)(Dockerfile|Containerfile)(\.[^/]*)?$|compose\.ya?ml$|docker-compose.*\.ya?ml$|\.dockerignore$'`. If the directory is not a git repository, use `find . -type f -not -path './.git/*'` and apply the same patterns. Include untracked files only when they are not ignored. If the user named a path, use that instead. If nothing matches, report that and stop. Do not invent files.

2. **Read every target file in full.** Use Read on each Dockerfile, compose file, and `.dockerignore`. For each Dockerfile, list every `FROM` line and its `AS` stage name. If there are more than five files and subagents are available, read them in parallel with subagents using `model: haiku`; otherwise read them sequentially.

3. **Check layer caching.** Dependency manifests (package.json, requirements.txt, go.mod, Cargo.toml, pom.xml) must be copied and installed before application source is copied. Flag a `COPY . .` that comes before the dependency install. Flag `apt-get update` in a separate `RUN` from `apt-get install`, and flag `apt-get install` without `--no-install-recommends`.

4. **Check multi-stage use.** Flag compilers, dev dependencies, test tooling, or package caches that reach the final stage. Compiled outputs should be copied with `COPY --from=<stage>` into a slim runtime stage.

5. **Check base image and size.** Flag full OS bases where a `-slim` or distroless variant would work. Flag `ADD` where `COPY` suffices. Flag package-manager caches left in the layer, such as missing `rm -rf /var/lib/apt/lists/*` or `pip install` without `--no-cache-dir`. Do not recommend Alpine for Python or Node images with native dependencies without warning about musl compatibility.

6. **Check pinned tags.** Every `FROM` and every compose `image:` should use a specific version tag or a digest. Flag `:latest` and untagged images. Flag major-only tags as lower severity.

7. **Check the runtime user.** Confirm the final stage sets `USER` to a non-root user. Statically, accept a numeric UID or a user name the Dockerfile creates (`useradd`, `adduser`, `addgroup`); do not assume a name exists in the base image without evidence. Flag a missing `USER` and a `USER root` in the final stage. In compose, flag `user: root` and `privileged: true`.

8. **Check secrets.** Run `grep -nEi 'password|secret|token|api[_-]?key|aws_' <target files from step 1>` and inspect each hit. Flag `ARG` or `ENV` values that hold secrets, because they persist in image history. Flag `COPY .env` or credential files copied into a layer. Recommend `RUN --mount=type=secret` instead.

9. **Check healthchecks.** For long-running services, confirm a `HEALTHCHECK` instruction or compose `healthcheck:` exists. Flag checks that only test that a process exists when the service exposes an endpoint. Skip one-shot jobs.

10. **Check .dockerignore.** Confirm it excludes `.git`, `node_modules`, build output, `.env*`, and local logs. If no `.dockerignore` exists and the build uses `COPY . .`, flag it.

11. **Verify each finding.** Keep a finding only if you can cite its file, line number, and exact text. If `hadolint` is on PATH, run `hadolint <Dockerfile>` and include its output labeled as tool output. Do not build images or start containers.

## Output format

First line: `<N> files reviewed: <H> high, <M> medium, <L> low.`

Then a table with columns `Severity`, `File:line`, `Issue`, `Fix`, sorted high to low.

- High: hardcoded secrets, root user in the final stage, `:latest` base in a production image.
- Medium: cache-unfriendly ordering, build tools in the final image, missing healthcheck, missing `.dockerignore`.
- Low: missing `--no-install-recommends`, missing cache cleanup, untagged or major-only tags.

Last, a `Passed` list with one line per check that found nothing. If there are no findings, say so in the first line and still list the checks.

## Notes

- This is a review. Do not edit files unless the user asks for fixes.
- Do not pull images, build, or run containers. The review is static.
- Ignore `root` in a build-only stage that is discarded before the final image.
- For a development-only compose override, cap severity at medium and say it is for development.
- Do not present hadolint output as your own analysis.
