# TOOLS.md — index (Testified-OSS improvement agent)

**Workspace:** `/Users/luucrew/.openclaw/workspaces/testified-oss-coder`

## Mission (one line)

Each HEARTBEAT: **randomly choose one** repo from the pool; set **`REPO`**, run **E / E.2 probes** (`gh …` **must** include **`-L 1`** on issue/PR list so `jq 'length'` is only **0** or **1**; **`--repo "$REPO"`**) **before** mirror; then **mirror + worktree**; then **create** / **idle** / **work** per **`HEARTBEAT.md`** **Routing law** and **Section F preflight** before any `gh issue create`.

## Startup order (every session / HEARTBEAT)

1. `SOUL.md`
2. `USER.md`
3. **`tools/conventions.md`**
4. **`tools/target-repos.md`** — two-repo pool + `shuf` random pick
5. **`tools/git-worktrees.md`** — **mandatory** local copy for both paths
6. **`tools/github-issues.md`** — branch on open issues + open PRs; create vs idle vs work-on-issue
7. **`tools/scan-paths.md`**
8. `AGENTS.md`
9. `HEARTBEAT.md`

## Playbooks (modular)

| Playbook | Role |
|----------|------|
| [`tools/conventions.md`](tools/conventions.md) | Commits, branches, PR rules |
| [`tools/target-repos.md`](tools/target-repos.md) | Two repos + **random** selection recipe |
| [`tools/git-worktrees.md`](tools/git-worktrees.md) | Mirror/worktree; **both** scenarios |
| [`tools/github-issues.md`](tools/github-issues.md) | Open-issue + open-PR check; create vs idle vs work-on-issue; `gh` flags |
| [`tools/github-prs.md`](tools/github-prs.md) | Draft PR stub when working issues |
| [`tools/github-discussions.md`](tools/github-discussions.md) | Discussions stub |
| [`tools/scan-paths.md`](tools/scan-paths.md) | Files to read in the worktree |

## Operator registry

| Key | Where |
|-----|--------|
| Cron | `/Users/luucrew/.openclaw/cron/jobs.json` |
| Agent | `testified-oss-coder` in `/Users/luucrew/.openclaw/openclaw.json` |
