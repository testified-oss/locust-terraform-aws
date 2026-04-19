# Git mirrors + worktrees

## Mandatory skill (all outcomes)

**Every** HEARTBEAT run—**create**, **PR follow-up** (**≥1 open PR** → **Section J**, with or without open issues), or **work on an existing issue**—runs **`HEARTBEAT.md` Sections E then E.2** first ( **`gh` probes only**), then needs a **local clone via mirror + worktree** on disk (**Section D**) before local file reads for scan/implement. Section J may add a **second** worktree on the **PR head branch** (`…-pr-${P}`); remove **all** ephemeral worktrees in **Section H**. Do not rely on `gh` alone to “read” the repo for triage or issue bodies.

Before any `git worktree` / mirror layout change, read and apply:

**`/Users/luucrew/.claude/skills/using-git-worktrees/SKILL.md`**

Announce when applying it: *using-git-worktrees skill governs layout and safety.*

## Layout (this workspace)

| Path | Purpose |
|------|---------|
| `git/mirrors/<owner>__<repo>.git` | Bare mirror (`git clone --mirror`) |
| `git/wt/<owner>__<repo>-<purpose>` | Ephemeral checkout: `scan` on default branch, `issue-N` for work-on-issue branches, or **`pr-P`** for **Section J** (existing PR head `HEAD`) |

**Workspace root:** `/Users/luucrew/.openclaw/workspaces/testified-oss-coder/codespaces/`

Worktree parents must be **gitignored** (see root `.gitignore`).

## Safety

```bash
git check-ignore -q git/wt 2>/dev/null || git check-ignore -q worktrees 2>/dev/null
```

If not ignored, fix `.gitignore` before first `worktree add`.

## Mirror lifecycle

```bash
mkdir -p git/mirrors
git clone --mirror "https://github.com/OWNER/REPO.git" "git/mirrors/OWNER__REPO.git"
git -C "git/mirrors/OWNER__REPO.git" fetch --prune
```

## Worktree — default branch (scan / create-issue path)

```bash
BR=$(gh repo view "OWNER/REPO" --json defaultBranchRef -q .defaultBranchRef.name)
git -C "git/mirrors/OWNER__REPO.git" worktree add "../wt/OWNER__REPO-${BR}-scan" "$BR"
```

Paths are relative to the mirror directory (`git/mirrors/`), so the checkout lands under **`git/wt/`**.

## Worktree — topic branch (work-on-issue path)

From the same mirror, add a second worktree **or** reuse after remove—never two checkouts of the same branch path:

```bash
git -C "git/mirrors/OWNER__REPO.git" worktree add -b "feat/issue-${N}-triage" "../wt/OWNER__REPO-issue-${N}" "$BR"
```

(Adjust branch prefix per **`tools/conventions.md`**.) Implement, commit with conventional messages, then push + `gh pr create --draft` per **`tools/github-prs.md`** (mirror remotes block plain `git push origin <branch>` — use the **URL push** recipe there).

## Worktree — existing PR head (**Section J**)

When **`run_path=pr_followup`**, fetch and check out the **PR’s head branch** (not default branch):

```bash
P=5
HEAD=feat/issue-5-ci-matrix   # from gh pr view / list JSON headRefName
git -C "git/mirrors/OWNER__REPO.git" fetch origin "$HEAD"
git -C "git/mirrors/OWNER__REPO.git" worktree add "../wt/OWNER__REPO-pr-${P}" "$HEAD"
```

Implement, commit, then **URL push** to the **same** `$HEAD` branch per **`tools/github-prs.md`**. **Never** `git worktree add -b …` for a new branch here unless you are explicitly abandoning the PR head (not allowed by default).

## Push caveat (`clone --mirror`)

Bare mirrors record **`remote.origin.mirror=true`**. Linked worktrees reuse that config, so **`git push origin <topic-branch>`** can fail with **`fatal: --mirror can't be combined with refspecs`**. That is **not** a mysterious “environment” limit — use the explicit **`https://github.com/${REPO}.git`** push form in **`tools/github-prs.md`** before **`gh pr create`**.

## After run

```bash
git worktree remove --force "git/wt/<path-used>"
```

Keep **mirrors**; remove **all** worktrees created this run.
