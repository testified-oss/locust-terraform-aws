# GitHub — PRs (stub)

Future: open improvement PRs from a dedicated branch in a worktree (same mirror layout as [`git-worktrees.md`](git-worktrees.md)).

## Rules (do not duplicate)

- Branch names: see [`conventions.md`](conventions.md).
- **Every** commit message and PR title: [`conventions.md`](conventions.md) only.

## Commands (reference)

```bash
gh pr list --repo "OWNER/REPO" --state open
gh pr view <number> --repo "OWNER/REPO"
```

## Policy

- No merge without explicit human approval.
- No force-push from automation unless a human updates this file to allow it.
