---
name: release
description: Commit, branch, version and release in pnpm workspaces built on the cpdevtools git-flow template (a .publish/versions.yml, 0.0.0-MAIN placeholder versions). Use when asked to commit, create a branch, prepare a release, bump a version, publish a pre-release, check CI, overlay unpublished sibling packages with dev-link, or explain why a release PR or workflow appeared. Covers the release model, branch and tag names, pre-releases from feature branches, dev-link overlays, the pre-commit hook, and the rules for committing and pushing.
---

# cpdevtools git-flow releases

Reference: <https://github.com/cpdevtools/git-flow-template> (its README). The user owns
`cpdevtools`: when a fix belongs in the toolchain it may land upstream, so check with `gh` whether
it did rather than assume.

## 0. Applies when

The repo has `.publish/versions.yml`.

## The model

- Manifests carry the placeholder `0.0.0-MAIN`; the real version is in `.publish/versions.yml`
  (`0.0.0-MAIN: 0.1.0`). Never put a real version in a `package.json`.
- **Any push to a branch not under `release/**`** runs `create-release-pr` (opens or updates a
  release PR) and `test`. Merging the release PR runs `build-pack-publish`.
- `main` publishes stable releases. A slashed branch (`feature/<name>`) is a development line that
  publishes only pre-releases (`{next}.0.0-feature.<name>.alpha.N`). Merge `main` into it, not the
  other way, until it graduates.
- Tags: `{package}/v{version}`, `MAIN/v{version}`, `v{version}`.
- `pnpm gitflow version` sets a version interactively and commits; non-interactively, edit
  `.publish/versions.yml` and commit that.
- Release artifact types include libraries (`ng-lib` packs `dist`) and `executable` (e.g. a
  PyInstaller exe built on `windows-latest`); the config is `release-artifacts.ts`.
- Unpublished sibling packages: publish a pre-release first, depend on the published range, then
  overlay the local checkout with `devutil dev-link` (`.publish/dev-local.yml`). The overlay can't
  bootstrap a package that isn't on the registry.

## Rules

- **Commit only when asked.** Use the attribution line the session gives.
- **Pushing is the user's.** A push starts CI and release PRs: never push without an explicit go
  for that push. If something appears on a remote unexpectedly, ask; the user likely pushed it.
- **Branch with `--no-track`:** `git switch -c <name> --no-track origin/main`. A branch started
  from a remote-tracking branch gets it as upstream, and a plain `git push` then goes to `main`.
- After committing, say how far ahead of its remote the branch is ("ahead by N").
- Never commit the user's own uncommitted edits that weren't part of the task; list them instead.
- Before committing: `pnpm check` (dependency versions agree), plus the package's
  `typecheck`/`lint`/`test`. `pnpm fix` aligns versions, removes local tags and formats.
- The husky pre-commit hook blocks `file:/…` paths in manifests or the lockfile: a dev-link leaked.
  Fix the cause; never bypass the hook.

**Mistakes to avoid:** a real version in `package.json`; merging a feature line into `main` to
"get a release"; `git switch -c x origin/main` without `--no-track`; `--no-verify`.

## Check CI

```bash
gh run list --limit 10
gh run view <id> --log-failed
gh pr list
```

Every PR, issue or run mentioned to the user is a clickable GitHub link, never a bare `#7`.
