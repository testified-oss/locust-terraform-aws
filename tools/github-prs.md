# GitHub — PRs (stub)

When **working an open issue** (`tools/github-issues.md`, **`HEARTBEAT.md` Section G**), open a **draft** PR from the topic branch when there is at least one commit (typically from a **second worktree**; same mirror layout as [`git-worktrees.md`](git-worktrees.md)). If **`gh pr list --state open`** is non-zero while open issues exist, you are on the **idle** path (**`HEARTBEAT.md` Section E.2**) — **do not** create another PR this run.

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
