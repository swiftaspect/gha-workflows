# Claude Code Instructions

Follow the conventions in [README.md](README.md) and [CONTRIBUTING.md](CONTRIBUTING.md) for this repo's scope, workflow conventions, and PR process.

## Task Completion Checklist

Before considering a task complete:

1. All touched workflow files pass `actionlint`
2. README.md's inputs reference table matches the actual workflow inputs
3. Commit messages follow Conventional Commits
4. If a reusable workflow's input shape changed, every consumer repo's shim is identified (even if the fix goes in a follow-up PR in each consumer)

## Notes for Claude

- Workflows here are consumed by other pvaas repos. A bug lands in every consumer the next time they update their pin. Be conservative.
- The `release.yml` reusable workflow has a required `release-type` input with no default — app repos pass `node`, meta repos pass `simple`.
- Never add a `latest` tag anywhere.
- `gh pr merge --auto --rebase <number>` — no `-R`/`--repo` flag (token is already scoped).
