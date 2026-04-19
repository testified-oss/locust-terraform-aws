# TOOLS.md — index (Testified-OSS improvement agent)

**Workspace:** `/Users/luucrew/.openclaw/workspaces/testified-oss-coder`

## Mission (one line)

Improve **allowlisted** `testified-oss` repos by scanning checked-out sources (mirrors + worktrees), deduping open issues, and filing **`gh issue create`** issues using the org playbook—**never** inventing work that already has an open issue.

## Startup order (every session / HEARTBEAT)

1. `SOUL.md`
2. `USER.md`
3. **`tools/conventions.md`** — commits, branches, PR rules (red line for any git write)
4. **`tools/target-repos.md`** — which `owner/name` repos exist this cycle
5. **`tools/git-worktrees.md`** — paths, mirror/worktree commands, **using-git-worktrees** skill
6. Task playbook for this run (v1 issues): **`tools/github-issues.md`**
7. **`tools/scan-paths.md`** — relative paths to read inside each worktree
8. `AGENTS.md`
9. `HEARTBEAT.md`

## Playbooks (modular)

| Playbook | Role |
|----------|------|
| [`tools/conventions.md`](tools/conventions.md) | Conventional commits, branch/PR naming, org rules |
| [`tools/target-repos.md`](tools/target-repos.md) | Allowlist of repos to improve |
| [`tools/git-worktrees.md`](tools/git-worktrees.md) | Mirror layout, worktree add/remove, skill link |
| [`tools/github-issues.md`](tools/github-issues.md) | Template title, dedupe, **`gh issue create`** flags |
| [`tools/github-prs.md`](tools/github-prs.md) | Stub for future PR automation (links conventions) |
| [`tools/github-discussions.md`](tools/github-discussions.md) | Stub for discussions / API |
| [`tools/scan-paths.md`](tools/scan-paths.md) | Files to read per repo after checkout |

## Operator registry

| Key | Where |
|-----|--------|
| Cron job definition | `/Users/luucrew/.openclaw/cron/jobs.json` |
| Agent id | `testified-oss-coder` in `/Users/luucrew/.openclaw/openclaw.json` |
