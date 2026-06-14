# claude-plugin-actions

Reusable GitHub Actions workflows for Claude Code plugin marketplaces. Consumers pin a `@v0` (or tighter) ref in their marketplace repos to run version hygiene checks without copying scripts.

## Repository Structure

```
.github/
  workflows/
    check-plugin-versions.yml   # Reusable workflow (consumed via workflow_call)
    ci.yml                      # CI: shellcheck + smoke test of the bundled script
    release.yml                 # Release: semver tagging + moving-tag fast-forward
  dependabot.yml                # Weekly action-version bump PRs
scripts/
  check-plugin-versions.sh     # Check logic — the consumer contract
CHANGELOG.md                   # Points to GitHub Releases; versioning policy
```

## What the Check Does

`scripts/check-plugin-versions.sh <base-ref>` validates Claude Code plugin marketplace hygiene:

1. **No `version` on marketplace entries** — `plugin.json` is the sole source of truth; marketplace-side `version` fields are silently ignored by the runtime and fail this check.
2. **`plugin.json` version bumped** — any changed file under `plugins/<name>/` requires a version bump in `plugins/<name>/.claude-plugin/plugin.json`.
3. **`metadata.version` bumped** — `marketplace.json`'s `metadata.version` must be incremented whenever any plugin changes.

The reusable workflow self-checks-out this repo at `github.job_workflow_sha` so the YAML and bundled script are always in sync.

## Running Checks Locally

```bash
# From a marketplace repo root:
git clone https://github.com/jedwards1230/claude-plugin-actions /tmp/cpa
bash /tmp/cpa/scripts/check-plugin-versions.sh origin/main
```

Environment variables (both optional):
- `MARKETPLACE_PATH` — path to `marketplace.json` (default: `.claude-plugin/marketplace.json`)
- `PLUGINS_DIR` — plugin root directory (default: `plugins`)

Exit codes: `0` = pass, `1` = version error, `2` = script/dependency error.

## Consumer Usage

```yaml
jobs:
  check:
    uses: jedwards1230/claude-plugin-actions/.github/workflows/check-plugin-versions.yml@v0
    secrets: inherit
```

Inputs: `marketplace-path`, `plugins-dir`, `base-ref`, `post-pr-comment` (all optional — see README).

## CI Workflow

`ci.yml` runs on every push and PR to `main`:
- **shellcheck** — lints `scripts/` via `ludeeus/action-shellcheck`
- **smoke test** — builds a temporary fixture git repo and exercises the script against three cases: missing bump (expect fail), proper bump (expect pass), no plugin changes (expect pass)

## Releases

Releases are **opt-in via labels** on merged PRs:
- `semver:patch` / `semver:minor` / `semver:major` — triggers a release
- No label → no release

The release workflow (`release.yml`) computes the next version from the latest `vX.Y.Z` tag, generates a changelog from merged PRs, creates an immutable tag, then fast-forwards moving tags (`v0`, `v0.1`) so pinned consumers stay current.

Manual dispatch is supported with `bump_type` and optional `dry_run` inputs.

## Versioning Policy

| Ref | Behavior |
|-----|----------|
| `@v0` | Moving — latest stable major |
| `@v0.1` | Moving — latest patch in the minor |
| `@v0.1.0` | Immutable pinned release |
| `@main` | Bleeding edge |

## Action Version Pins

Current pinned versions (update these when Dependabot bumps them):
- `actions/checkout@v6`
- `softprops/action-gh-release@v3`
- `ludeeus/action-shellcheck@00cae500b08a931fb5698e11e79bfbd38e612a38` (SHA-pinned)

## Contributing

1. Make changes in a PR branch.
2. CI must pass (shellcheck + smoke test).
3. Add `semver:patch`, `semver:minor`, or `semver:major` label to the PR to trigger a release on merge.
4. Changes to `check-plugin-versions.sh` are the consumer contract — update the smoke test in `ci.yml` for new behaviors.
