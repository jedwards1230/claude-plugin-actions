# claude-plugin-actions

![Version](https://img.shields.io/github/v/release/jedwards1230/claude-plugin-actions?style=flat-square&color=blue)
![License](https://img.shields.io/github/license/jedwards1230/claude-plugin-actions?style=flat-square)

Reusable GitHub Actions workflows for [Claude Code plugin marketplaces](https://code.claude.com/docs/en/plugin-marketplaces).

## Workflows

### `check-plugin-versions.yml`

Validates plugin marketplace hygiene on every push and pull request:

- `plugin.json` `version` is bumped whenever files under `plugins/<name>/` change
- `marketplace.json` `metadata.version` is bumped when the catalog changes
- No `version` field on marketplace plugin entries (silently ignored per the [plugins reference](https://code.claude.com/docs/en/plugins-reference#version-resolution-and-release-channels); `plugin.json` is the source of truth)
- Posts a PR comment summarizing added / removed / updated plugins

The check logic lives in [`scripts/check-plugin-versions.sh`](scripts/check-plugin-versions.sh). The reusable workflow self-checks-out this repo at the same ref the caller used, so the workflow YAML and bundled script never drift.

#### Inputs

| Input | Description | Default |
|-------|-------------|---------|
| `marketplace-path` | Path to `marketplace.json` relative to repo root | `.claude-plugin/marketplace.json` |
| `plugins-dir` | Directory containing plugin subdirectories (matches `metadata.pluginRoot`) | `plugins` |
| `base-ref` | Git ref to compare against | PR base SHA, or `HEAD~1` on push |
| `post-pr-comment` | Post a PR comment summarizing plugin changes | `true` |

#### Consumer usage

Drop this in `.github/workflows/check-plugin-versions.yml` in your marketplace repo:

```yaml
name: Plugin Version Check

on:
  push:
    branches: [main]
    paths: ['plugins/**', '.claude-plugin/marketplace.json']
  pull_request:
    types: [opened, synchronize]
    paths: ['plugins/**', '.claude-plugin/marketplace.json']

permissions:
  contents: read
  pull-requests: write

jobs:
  check:
    uses: jedwards1230/claude-plugin-actions/.github/workflows/check-plugin-versions.yml@v0
    secrets: inherit
```

That's it — no local script copy needed.

#### Local pre-push validation

To run the same checks locally before pushing:

```bash
git clone https://github.com/jedwards1230/claude-plugin-actions /tmp/claude-plugin-actions
bash /tmp/claude-plugin-actions/scripts/check-plugin-versions.sh origin/main
```

## Version pinning

| Approach | Ref | Update behavior |
|----------|-----|-----------------|
| Latest stable | `@v0` | Moving major tag, gets new features and fixes |
| Pinned minor | `@v0.1` | Gets patch fixes only |
| Pinned patch | `@v0.1.0` | Exact version, no auto-updates |
| Latest unstable | `@main` | Bleeding edge |

## License

MIT — see [LICENSE](LICENSE).
