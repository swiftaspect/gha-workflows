# Contributing to gha-workflows

## Scope of changes

This repo holds **reusable workflows** consumed by other repos. Changes here affect every consumer the next time they bump their pin. Take that seriously:

- Breaking changes (removing inputs, changing required-ness, changing semantics) get a `feat!:` or `BREAKING CHANGE:` commit and a new major version from release-please.
- Additive changes (new optional input with a default) are `feat:` and get a minor bump.
- Bug fixes (typo in a step, incorrect `gh pr merge` invocation, etc.) are `fix:` and get a patch bump.

## Local validation

```sh
# Lint the workflow files (same tool the self-CI workflow runs)
docker run --rm -v "$(pwd)":/repo:Z -w /repo rhysd/actionlint:latest .github/workflows/*.yml
```

## Workflow conventions

- Every reusable workflow uses `on: workflow_call` only — never `push`, `pull_request`, or `schedule`.
- Every input is documented with a `description` explaining when to use it, what it does, and any caveats. No silent behavior.
- No floating refs anywhere — all actions pinned by tag (`actions/checkout@v5`, not `@main`).
- No `latest` tag references in any workflow.
- `gh pr merge` takes `--repo` after the subcommand, never `-R` before it. It reads the repository from a git remote, not from `GH_TOKEN`, so a job without an `actions/checkout` step has to pass it.

## Commit and PR workflow

- Conventional Commits on every commit.
- No squash merges — merge commits or rebase only.
- PR title summarizes the change; for single-commit PRs it matches the commit subject.
- Scopes are optional.

## Adding a new reusable workflow

1. Pick a descriptive name — the filename becomes part of every consumer's `uses:` reference, so it must be stable.
2. Add `on: workflow_call` with inputs documented.
3. Add an entry to README.md's inputs reference table.
4. Add a usage snippet to README.md if the workflow's invocation shape differs meaningfully from existing ones.
5. Run `actionlint` locally.
6. Open a PR.

## Dependency updates

Dependabot opens the pull requests for the actions these workflows pin, and
`self-dependabot-automerge.yml` runs the repository's own
`dependabot-automerge.yml` against them: a pull request merges without review
when every check on its head commit has finished without failing and dependabot
classified the bump as patch or minor. A major action version always waits for
a person, as does anything dependabot could not classify.

Those bumps are `fix` and cut a release. An action pinned inside a reusable
workflow is part of what a consumer gets by pinning a tag, so a bump that cut
no release would leave every consumer on the old pin.

## Release

Managed by release-please. Merging the auto-generated release PR cuts a tag,
and `self-major-tag.yml` moves `v1` onto it.

Consumers bump their pins on their own schedule. The exception is a shim that
lives inside a project template, which pins `@v1` because dependabot cannot
reach a pin inside a jinja source. Anything merged here reaches those repos on
their next run, with no review step in between.
