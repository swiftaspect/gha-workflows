# gha-workflows

Reusable GitHub Actions workflows shared across a set of related repos. Each workflow is invoked from a consumer repo via `workflow_call`.

## Workflows

| Workflow | Purpose |
|----------|---------|
| `ci.yml` | Lint + typecheck + test (via `make check`), commit-message lint on pull requests, optional pre-release-dep gate (PR to main only), optional branch pre-release publish (via `make publish`, for the shared library) |
| `publish-tag.yml` | On tag push: optional container publish, optional npm publish, optional release-asset upload |
| `release.yml` | release-please-driven release PR creation and auto-merge |
| `dependabot-automerge.yml` | Merge a dependabot PR once every check on its head commit passes, if the update is patch or minor |

Consumer shims pick which inputs to flip. A typical TypeScript server repo's `.github/workflows/ci.yml` looks like:

```yaml
name: Testing

on:
  push:
    branches-ignore:
      - 'release-please*'
  pull_request:
    branches:
      - main

jobs:
  ci:
    uses: swiftaspect/gha-workflows/.github/workflows/ci.yml@v<version>
    with:
      check-prerelease-deps: true
      compose-up-for-tests: true
    secrets: inherit
```

The shared library additionally sets `publish-on-branch: true`. The pre-release publishes when a pull request carries the `preview` label and the checks pass, so `make sync-branch-deps` in a sibling repository can resolve a matching branch build. Add the label when another repository needs to pin the branch; without it nothing publishes.

**Branch pre-releases go to the registry only — never a GitHub Release.** The publish step inherits the workflow's `permissions: contents: read, packages: write`, so it is structurally incapable of creating a GitHub Release or pushing a git tag (both require `contents: write`). Branch builds land solely as registry artifacts: an npm dist-tag and/or a container tag. GitHub Releases and git tags are produced *only* by the real-release path (`release.yml` / `publish-tag.yml`, which run with `contents: write`). A repo's `make publish` must uphold this — it may push to package/container registries but must not run `gh release …` or `git tag`/`git push`. The workflow exposes `GITHUB_REF_NAME`, `GITHUB_RUN_NUMBER`, and `GITHUB_TOKEN` to it, and the Makefile owns the language-specific translation.

On a pull request, the workflow lints the commit messages in the pull request before running anything else. Conventional Commits is a git convention rather than a language one, so the check runs here rather than delegating to the repository. A repository with no `commitlint.config.js` is skipped with a notice. The checkout uses `fetch-depth: 0` because the default shallow clone contains none of the commits being checked.

## Versioning

Pin consumers to a specific tag:

```yaml
uses: swiftaspect/gha-workflows/.github/workflows/ci.yml@v0.1.0
```

Tags follow conventional-commits-driven release-please bumps. Consumers update
their pin explicitly when adopting a new version.

A `v1` tag also exists, moved onto every `v1.x.y` release by
`self-major-tag.yml`. It is not the recommended pin. It is there for one caller
shape that cannot use an exact one: a shim that lives inside a project template
as a jinja source, where the pin is unreachable to dependabot, because
dependabot matches manifests by filename. Pin `@v1` only if that describes you,
and understand that a change here then reaches you on your next run with no
review step in between.

## Inputs reference

### `ci.yml`

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `publish-on-branch` | bool | `false` | After `make check` passes, on a pull request labelled `preview`, run `make publish` so the repo's Makefile emits a branch-tagged pre-release artifact (npm dist-tag, container tag, …). Language-agnostic — the Makefile owns the translation. |
| `check-prerelease-deps` | bool | `false` | PR gate (only runs on PRs targeting `main`): rejects pre-release / dist-tag `@swiftaspect/*` sibling deps and branch-tagged `ghcr.io/swiftaspect/*` compose images. The manifest half is delegated to the repo's `make check-prerelease-deps` target (language-owned; skipped with a notice if absent); the compose image-tag check runs in-workflow (already language-agnostic). |
| `compose-up-for-tests` | bool | `false` | Run `make start` before `make check` and `make stop` after. For repos whose tests need the compose dependency stack. |
| `app-token-repositories` | string | `''` | Comma-separated repositories under the caller's owner that the check run needs to read. Example: a private sibling whose release assets `make check` downloads. When set, the minted app installation token replaces `GITHUB_TOKEN` for the `make start` and `make check` steps. The app must then cover everything those steps reach, package reads included. |

### `publish-tag.yml`

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `publish-container` | bool | `false` | Run `make publish-container` — pushes primary + alt tags to the registry (never `latest`). |
| `publish-npm` | bool | `false` | Run `make publish-package` — publishes the npm tarball to GitHub Packages. |
| `upload-release-assets` | bool | `false` | Upload the `make build-package` tarball to the GitHub Release via `gh release upload`. |

### `release.yml`

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `release-type` | string | **required** | release-please release type. `node` for app repos with a `package.json`; `simple` for meta repos with no version to bump. No default — callers must specify. |

### `dependabot-automerge.yml`

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `merge-method` | string | `rebase` | `rebase` or `merge`. `squash` errors rather than defaulting away, because it collapses the Conventional Commit messages release tooling reads from the default branch. |
| `allowed-update-types` | string | `version-update:semver-patch version-update:semver-minor` | Space-separated dependabot `update-type` values. Every dependency in a PR must declare one of them, so a dependency carrying no `update-type` at all can never match. |

Its shim is not shaped like the others, because a merge decision can only be
made once the tests have finished:

```yaml
name: Dependabot Auto-Merge

on:
  workflow_run:
    workflows: ['Testing']    # the name of the workflow that runs your tests
    types: [completed]

permissions:
  checks: read
  contents: write
  pull-requests: write

jobs:
  automerge:
    if: >-
      github.event.workflow_run.conclusion == 'success'
      && github.event.workflow_run.actor.login == 'dependabot[bot]'
      && startsWith(github.event.workflow_run.head_branch, 'dependabot/')
    uses: swiftaspect/gha-workflows/.github/workflows/dependabot-automerge.yml@v<version>
    secrets: inherit
```

`pull_request` cannot be used. Dependabot's own `pull_request` events receive a
read-only `GITHUB_TOKEN` that cannot merge, and they fire when the PR opens,
before the tests have run. `workflow_run` is the only event that arrives after
a test run finishes and carries a token that can act on the result. It also
means a PR cannot change the rules that judge it, since `workflow_run`
workflows always execute the copy on the default branch.

`permissions` is required, and all three scopes are. A called workflow can only
narrow what its caller was given, so where `default_workflow_permissions` is
`read` nothing works without the block. `checks: read` is the one that is easy
to miss: an explicit `permissions` block sets every scope it does not name to
`none`, and the merge gate reads the check runs on the head commit, so leaving
it out fails with `Resource not accessible by integration (HTTP 403)`.

The `if` is a cost filter, not the security boundary. A test workflow that
triggers on `push` runs for every branch, including branches with no PR, and
the guard skips those before a runner is allocated.

Before merging, the workflow checks:

| Check | Notes |
|-------|-------|
| PR author is `dependabot[bot]` | read from the API, not the event actor |
| Head commit author is `dependabot[bot]` and its signature is verified | catches a commit pushed onto the branch by anyone else |
| Head branch is in this repository | a fork can never qualify |
| Head commit still equals the tested commit | re-checked through `--match-head-commit` at merge time |
| Base is the default branch, and the PR is not a draft | |
| Every check run on the head SHA has concluded, and none failed | all of them, not only the workflow that triggered this one; `skipped` and `neutral` count as passes |
| Every dependency declares an allowed `update-type` | read from the metadata block in the commit message |

A pull request that fails one of those checks is left open, and the job passes.
Nothing is wrong. A human decides from there.

The merge is different. Dependabot rebases its own branches and the tests run
again. A merge that loses that race is a notice, and the next run merges. Every
other merge failure fails the job, including a branch protection that forbids
the merge. That case never resolves itself, so a repository configured that way
should not run this workflow.

## Secrets and variables expected

Consumers must have installed the SAGHAH GitHub App and must expose these org-level values:

- `vars.SAGHAH_APP_ID`
- `vars.SAGHAH_CLIENT_ID`
- `secrets.SAGHAH_PRIVATE_KEY`

These back the release-please `gh pr merge --auto --rebase` flow.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

Apache-2.0 — see [LICENSE](LICENSE).
