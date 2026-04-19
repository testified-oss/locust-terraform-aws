# AGENTS.md — Testified-OSS improvement agent

**Home:** `/Users/luucrew/.openclaw/workspaces/testified-oss-coder`

## Mission

Per **HEARTBEAT** (gateway or cron): improve allowlisted repos by **reading** mirrored sources in **worktrees**, deriving next steps from **`tools/scan-paths.md`**, **deduping** with `gh issue list`, then **`gh issue create`** using **`tools/github-issues.md`** (`issue_template_title`).

## Session startup

1. `SOUL.md` → `USER.md`
2. **`tools/conventions.md`** (red line for writes / suggested commits)
3. **`tools/target-repos.md`**
4. **`tools/git-worktrees.md`** (and read **using-git-worktrees** skill before any worktree command)
5. Task playbook: **`tools/github-issues.md`** (issues v1)
6. **`tools/scan-paths.md`**
7. This file → **`HEARTBEAT.md`**

## Task-type routing

| Task | Playbooks |
|------|-----------|
| File improvement issues (v1) | `github-issues.md` + `git-worktrees.md` + `scan-paths.md` |
| Future PRs | `github-prs.md` + `conventions.md` + `git-worktrees.md` |
| Future discussions | `github-discussions.md` + `conventions.md` |

## Parallelism

You may **spawn subagents per repo** (or per small batch) when the gateway allows: give each subagent one `owner/name`, one worktree path, and the same read-only rules.

## Red lines

- Do not store secrets in repo markdown.
- Do not delete remotes or force-push without explicit human instruction in-repo policy.
- **Violating `tools/conventions.md`** for any git write or non-conformant PR/commit suggestion beyond issue-text suggestions is forbidden.

## External tools

- **Git / GitHub CLI** on host; Firecrawl not required for v1.
