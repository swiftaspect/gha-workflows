# gha-workflows

Reusable GitHub Actions workflows shared by all pvaas repos. Each workflow is invoked from a consumer repo via `workflow_call`.

## Workflows

| Workflow | Purpose |
|----------|---------|
| `ci.yml` | Lint + typecheck + test (via `make check`), optional pre-release-dep gate (PR to main only), optional npm-branch-publish (for the shared library) |
| `publish-tag.yml` | On tag push: optional container publish, optional npm publish, optional release-asset upload |
| `release.yml` | release-please-driven release PR creation and auto-merge |

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

The shared library (`spiras`) additionally sets `publish-npm-on-branch: true` so branch pushes publish dist-tagged pre-release versions consumable by `make sync-branch-deps`.

## Versioning

Pin consumers to a specific tag:

```yaml
uses: swiftaspect/gha-workflows/.github/workflows/ci.yml@v0.1.0
```

Tags follow conventional-commits-driven release-please bumps. No floating `v1` tag — consumers update their pin explicitly when adopting a new version.

## Inputs reference

### `ci.yml`

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `publish-npm-on-branch` | bool | `false` | After `make check` passes, publish a pre-release npm version with the sanitized branch name as dist-tag. Used by spiras. |
| `check-prerelease-deps` | bool | `false` | PR gate (only runs on PRs targeting `main`): fail if `package.json` has pre-release or dist-tag `@swiftaspect/*` deps, or if compose files reference branch-tagged `ghcr.io/swiftaspect/*` images. |
| `compose-up-for-tests` | bool | `false` | Run `make start` before `make check` and `make stop` after. For repos whose tests need the compose dependency stack. |

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
