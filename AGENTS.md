# AGENTS.md — Testified-OSS improvement agent

**Home:** `/Users/luucrew/.openclaw/workspaces/testified-oss-coder`

## Mission

Each **HEARTBEAT**: **randomly pick one** of the two repos in **`tools/target-repos.md`**, set **`REPO="OWNER/REPO"`**, then run **`HEARTBEAT.md` Sections E then E.2** (existence probes with **`-L 1`** — **always both**, **before** mirror) so **`has_open_issues`** / **`has_open_prs`** are **0** or **1** and logged. Then **always** create/update a **mirror + worktree** (**Section D**; **using-git-worktrees** + **`tools/git-worktrees.md`**).

Then (same order as **`HEARTBEAT.md` Routing law**):

- **`has_open_issues=0`** and **`has_open_prs=0`** → after D: scan **`tools/scan-paths.md`**, dedupe, **`HEARTBEAT.md` Section F preflight**, then **`gh issue create`** per **`tools/github-issues.md`** (always **`--repo "$REPO"`**).
- **`has_open_prs≠0`** (whether or not **`has_open_issues=0`**) → after D: **`HEARTBEAT.md` Section J** — **no** `gh issue create`, **no** `gh pr create` (second PR), **no** merge. **Do** inspect **CI + PR/review comments** on **one** open PR; **commits + URL push** only on that PR’s **head branch** to fix failing checks or clear review/CI feedback; then **`gh pr comment`** if you changed anything (or log **no-op**). Playbook: **`tools/github-prs.md`**.

- **`has_open_issues≠0`** and **`has_open_prs=0`** → after D: **Section G** — **work on one** issue; **`gh pr create --draft`** when there is at least one commit, then **`gh issue comment`** with the PR link — **`tools/github-prs.md`**, **`tools/conventions.md`**. **Never** **`gh issue create`** while **`has_open_issues` is not `0`**.

## Session startup

1. `SOUL.md` → `USER.md`
2. **`tools/conventions.md`**
3. **`tools/target-repos.md`** (two-repo pool + random rule)
4. **`tools/git-worktrees.md`** + read **`/Users/luucrew/.claude/skills/using-git-worktrees/SKILL.md`** before any `git worktree` command
5. **`tools/github-issues.md`**
6. **`tools/github-prs.md`** (Section G draft PRs + **Section J** PR follow-up)
7. **`tools/scan-paths.md`**
8. This file → **`HEARTBEAT.md`** (execute **E then E.2 before D** — E.2 **always**; **F preflight** before `gh issue create`)

## Task-type routing

| Task | Playbooks |
|------|-----------|
| New issue (zero open) | `github-issues.md` + `git-worktrees.md` + `scan-paths.md` + `conventions.md` (suggested commits in body) |
| PR follow-up (**≥1 open PR**; open issues optional) | `HEARTBEAT.md` J + `github-prs.md` + `git-worktrees.md` + `conventions.md` — checks, comments, fix on PR head branch |
| Work on issue (≥1 open, **0** open PRs) | `HEARTBEAT.md` G + `github-issues.md` + `git-worktrees.md` + `conventions.md` + `github-prs.md` (draft PR when there are commits) |
| Future discussions | `github-discussions.md` + `conventions.md` |

## Parallelism

Default heartbeat is **one repo per run** (random). Subagents are optional if you later split work; do not parallelize across both pool repos in the same heartbeat unless a human changes **`tools/target-repos.md`**.

## Red lines

- Do not store secrets in markdown.
- No force-push / no merge without explicit human policy.
- **Never skip** mirror+worktree for “quick” `gh`-only reads—local tree is required for **create**, **pr_followup** (including **PR-only**: zero open issues, ≥1 open PR), and **work-on-issue** paths (Section J may add a **PR-branch** worktree after triage).
- **Never** open a **new** issue when **`has_open_issues` is not `0`** (probe in **`HEARTBEAT.md` Section E**; with **`-L 1`** that means **`1`**). **`HEARTBEAT.md` Section F preflight** must pass **immediately before** `gh issue create`.
- **Always** pass **`--repo "$REPO"`** (or **`-R "$REPO"`**) to **`gh`** so the picked pool repo is not confused with another default repo.
- **`tools/conventions.md`** governs branches/commits/PR titles.

## External tools

- Host **`git`** + **`gh`**.
