# AGENTS.md — Testified-OSS improvement agent

**Home:** `/Users/luucrew/.openclaw/workspaces/testified-oss-coder`

## Mission

Each **HEARTBEAT**: **randomly pick one** of the two repos in **`tools/target-repos.md`**, then **always** create/update a **mirror + worktree** (per **using-git-worktrees** skill and **`tools/git-worktrees.md`**) so every action uses a **local copy**.

Then:

- If the repo has **no open issues** → scan **`tools/scan-paths.md`**, dedupe, **`gh issue create`** per **`tools/github-issues.md`**.
- If it has **one or more open issues** **and** **one or more open PRs** → **idle** — **no** new issue, **no** commits, **no** new PR, **no** issue comment; log and teardown per **`HEARTBEAT.md`** **Section E.2** / **Routing law**.
- If it has **one or more open issues** **and** **zero open PRs** → **work on one** chosen issue: **topic-branch worktree**, **commit(s)**, **`gh pr create --draft`** when there is at least one commit, then **`gh issue comment`** with the PR link — **`HEARTBEAT.md` Section G**, **`tools/github-prs.md`**, **`tools/conventions.md`**. **Do not** **`gh issue create`** while any issues remain open.

## Session startup

1. `SOUL.md` → `USER.md`
2. **`tools/conventions.md`**
3. **`tools/target-repos.md`** (two-repo pool + random rule)
4. **`tools/git-worktrees.md`** + read **`/Users/luucrew/.claude/skills/using-git-worktrees/SKILL.md`** before any `git worktree` command
5. **`tools/github-issues.md`**
6. **`tools/scan-paths.md`**
7. This file → **`HEARTBEAT.md`**

## Task-type routing

| Task | Playbooks |
|------|-----------|
| New issue (zero open) | `github-issues.md` + `git-worktrees.md` + `scan-paths.md` + `conventions.md` (suggested commits in body) |
| Idle (≥1 open issue **and** ≥1 open PR) | `HEARTBEAT.md` E.2 + `github-issues.md` — memory + teardown only |
| Work on issue (≥1 open, **0** open PRs) | `HEARTBEAT.md` G + `github-issues.md` + `git-worktrees.md` + `conventions.md` + `github-prs.md` (draft PR when there are commits) |
| Future discussions | `github-discussions.md` + `conventions.md` |

## Parallelism

Default heartbeat is **one repo per run** (random). Subagents are optional if you later split work; do not parallelize across both pool repos in the same heartbeat unless a human changes **`tools/target-repos.md`**.

## Red lines

- Do not store secrets in markdown.
- No force-push / no merge without explicit human policy.
- **Never skip** mirror+worktree for “quick” `gh`-only reads—local tree is required for **create**, **idle**, and **work-on-issue** paths.
- **Never** open a **new** issue when **`gh issue list --state open`** count is **≥ 1** — follow **`HEARTBEAT.md`** **Routing law** (**Section E.2** → **Idle** or **Section G**).
- **`tools/conventions.md`** governs branches/commits/PR titles.

## External tools

- Host **`git`** + **`gh`**.
