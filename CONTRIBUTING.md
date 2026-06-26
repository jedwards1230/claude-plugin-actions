# Contributing to claude-plugin-actions

This repo ships a reusable GitHub Actions workflow and companion bash script for Claude Code plugin marketplace version hygiene. All changes go through the workflow below.

## Prerequisites

- [shellcheck](https://www.shellcheck.net/) (v0.9+) — to lint shell scripts locally

## Build, test & lint

```bash
# Lint the bundled script (mirrors the Shellcheck CI job)
shellcheck scripts/check-plugin-versions.sh
```

The full smoke test (three fixture-based cases) runs in CI only; see the `smoke-test` job in `.github/workflows/ci.yml` for the script logic.

## Documentation

Keep documentation current as part of the change, not as a follow-up — update the README and any affected docs in the same PR.

## Before you open a PR

- Make sure all CI checks pass locally first — run shellcheck before pushing.

## Branching & commits

- Branch off `main`; never commit directly to `main`.
- Use [Conventional Commits](https://www.conventionalcommits.org/) prefixes (`feat:`, `fix:`, `docs:`, `chore:`, `refactor:`, `test:`, …).
- Sign your commits where possible (`git commit -S`).
- Keep each PR focused; delete dead code rather than commenting it out.

## Pull requests

- Open the PR against `main`.
- Every PR runs CI. Resolve **all** review threads before the PR is merged.
- An automated code review runs on each PR; address and resolve its threads like any other review.
- A PR can be merged once CI is green and all review threads are resolved.

## Releases

Releases are triggered by a `semver:patch`, `semver:minor`, or `semver:major` label on the merged PR. The self-contained workflow (`.github/workflows/release.yml`) increments the latest `vX.Y.Z` tag, creates an immutable version tag, fast-forwards moving tags (`v0`, `v0.1`) so pinned consumers stay current, and publishes a GitHub Release with a changelog. Manual dispatch is available with explicit `bump_type` and `dry_run` inputs.
