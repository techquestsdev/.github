# techquestsdev/.github

Organization-wide GitHub configuration, community health files, canonical issue labels, and reusable GitHub Actions workflows for [Tech Quests](https://github.com/techquestsdev) repositories.

Note: `profile/README.md` renders on the public `https://github.com/techquestsdev` organization profile page. This root `README.md` documents how individual repositories inherit or call the shared configuration in this repository.

## 1. Automatically Inherited Community Health Files

Any repository under `techquestsdev` automatically inherits the files below **only when that repository does not define its own local copy** in root, `docs/`, or `.github/`:

| Path in `techquestsdev/.github` | Scope | Notes |
| --- | --- | --- |
| `CODE_OF_CONDUCT.md` | Org-wide | Contributor Covenant 2.1 |
| `CONTRIBUTING.md` | Org-wide | Conventional Commits and PR guidelines |
| `SECURITY.md` | Org-wide | Private vulnerability reporting policy |
| `FUNDING.yml` | Org-wide | Sponsor links |
| `pull_request_template.md` | Org-wide | Generic PR checklist (works for Go, Rust, Tauri, and web repos) |
| `ISSUE_TEMPLATE/bug_report.yml` | Org-wide | Structured bug report form (`bug`, `needs-triage`) |
| `ISSUE_TEMPLATE/feature_request.yml` | Org-wide | Structured feature request form (`enhancement`, `needs-triage`) |
| `ISSUE_TEMPLATE/config.yml` | Org-wide | Disables blank issues and links to Discussions and docs |
| `.github/DISCUSSION_TEMPLATE/q-a.yml` | Org-wide | Q&A discussion category form |
| `.github/DISCUSSION_TEMPLATE/ideas.yml` | Org-wide | Ideas discussion category form |

### Migrating an existing repo to inherit templates

GitHub replaces the entire default `ISSUE_TEMPLATE` directory if a repository has *any* file inside its own `.github/ISSUE_TEMPLATE/`. To inherit the shared templates in a `techquestsdev` repository, delete the local duplicates:

```bash
git rm -rf --ignore-unmatch .github/ISSUE_TEMPLATE .github/DISCUSSION_TEMPLATE .github/pull_request_template.md pull_request_template.md CONTRIBUTING.md CODE_OF_CONDUCT.md SECURITY.md
```

## 2. Canonical Labels (`sync-labels.yml`)

GitHub does not inherit issue labels from `.github` automatically. Canonical labels are defined in `.github/labels.json` (`bug`, `enhancement`, `documentation`, `duplicate`, `good first issue`, `help wanted`, `invalid`, `question`, `wontfix`, `needs-triage`, `dependencies`, `security`, `ci`, `github-actions`, `go`, `rust`).

### Option A: Sync from here via `workflow_dispatch`

Run the **Sync Labels** workflow in the Actions tab of `techquestsdev/.github`:
- Set `target` to `techquestsdev/<repo>` (or `all` to sync every non-archived repo in the org).
- Configure a `LABELS_TOKEN` secret in `techquestsdev/.github` with `repo` (or fine-grained `Issues: Read and write` + `Metadata: Read`) permissions across the org when targeting other repositories.

### Option B: Call from an individual repository

Add `.github/workflows/labels.yml` in the target repo (uses the repo's built-in `GITHUB_TOKEN`, no PAT needed):

```yaml
name: Sync Labels

on:
  workflow_dispatch:
  schedule:
    - cron: "0 6 * * 1"

permissions:
  contents: read
  issues: write

jobs:
  labels:
    uses: techquestsdev/.github/.github/workflows/sync-labels.yml@main
```

## 3. Reusable Workflows

All reusable workflows live under `.github/workflows/` and are versioned on `main`. Public repositories outside `techquestsdev` (for example staging repos before org transfer) can also call them directly via `techquestsdev/.github/.github/workflows/<name>.yml@main`.

### `go-ci.yml` (Go test, lint, Trivy security scan, 3-OS build)

Standard Go CI caller (`.github/workflows/ci.yml`):

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read
  security-events: write

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  ci:
    uses: techquestsdev/.github/.github/workflows/go-ci.yml@main
    secrets: inherit
```

Optional inputs for repos with build tags or generated-file drift checks (for example `gdash`):

```yaml
  ci:
    uses: techquestsdev/.github/.github/workflows/go-ci.yml@main
    with:
      build-tags: "nodbus"
      golangci-args: "--config=.golangci.yml --build-tags=nodbus --timeout=5m"
      generate-check-paths: "docs/public/schema.json"
    secrets: inherit
```

### `go-release.yml` (Conventional-commit auto-semver + GoReleaser)

Computes the next semantic version across all commits since the latest `v*.*.*` tag (`feat:` -> minor, `fix:`/`perf:` -> patch, `type!:` or `BREAKING CHANGE:` -> major, `[skip release]` or `chore:`/`docs:`/`ci:`-only -> skipped), pushes the annotated tag, runs `go test -race`, and executes GoReleaser v2.

Caller `.github/workflows/release.yml`:

```yaml
name: Release

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: write

concurrency:
  group: release-${{ github.ref }}
  cancel-in-progress: false

jobs:
  release:
    uses: techquestsdev/.github/.github/workflows/go-release.yml@main
    secrets:
      HOMEBREW_TOKEN: ${{ secrets.HOMEBREW_TOKEN }}
```

Required repository secret when publishing formulas or casks to `techquestsdev/homebrew-tap`: `HOMEBREW_TOKEN`.

### `semver-tag.yml` (Language-neutral conventional-commit tagger)

For non-Go repositories (Rust, Tauri, C, Node) that have their own build/release matrix after tagging:

```yaml
name: Tag Release

on:
  push:
    branches: [main]

permissions:
  contents: write

jobs:
  tag:
    uses: techquestsdev/.github/.github/workflows/semver-tag.yml@main
    secrets:
      RELEASE_TOKEN: ${{ secrets.RELEASE_TOKEN }}
```

Note on GitHub Actions loop prevention: if downstream release builds live in a separate workflow triggered by `on: push: tags:`, pass a `RELEASE_TOKEN` PAT or GitHub App token (tags pushed with the default `GITHUB_TOKEN` do not trigger new workflow runs). Otherwise, call your build job in the same workflow with `needs: tag` and `if: needs.tag.outputs.skipped != 'true'`.

### `dependency-review.yml` (PR vulnerability and license gate)

Caller `.github/workflows/dependency-review.yml`:

```yaml
name: Dependency Review

on:
  pull_request:
    branches: [main]

permissions:
  contents: read
  pull-requests: write

jobs:
  dependency-review:
    uses: techquestsdev/.github/.github/workflows/dependency-review.yml@main
```

### `docs-pages.yml` (pnpm static docs build and GitHub Pages deploy)

Caller `.github/workflows/docs.yml`:

```yaml
name: Docs

on:
  push:
    branches: [main]
    paths:
      - "docs/**"
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: pages
  cancel-in-progress: false

jobs:
  docs:
    uses: techquestsdev/.github/.github/workflows/docs-pages.yml@main
```
