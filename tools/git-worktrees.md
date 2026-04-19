# Git mirrors + worktrees

## Mandatory skill (all outcomes)

**Every** HEARTBEAT run—**create**, **idle** (open issues + open PRs), or **work on an existing issue**—needs a **local clone via mirror + worktree** on disk before relying on local file reads. Do not rely on `gh` alone to “read” the repo for triage or issue bodies.

Before any `git worktree` / mirror layout change, read and apply:

**`/Users/luucrew/.claude/skills/using-git-worktrees/SKILL.md`**

Announce when applying it: *using-git-worktrees skill governs layout and safety.*

## Layout (this workspace)

| Path | Purpose |
|------|---------|
| `git/mirrors/<owner>__<repo>.git` | Bare mirror (`git clone --mirror`) |
| `git/wt/<owner>__<repo>-<purpose>` | Ephemeral checkout: `scan` on default branch, or `issue-N` for work-on-issue branches |

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

(Adjust branch prefix per **`tools/conventions.md`**.) Implement, commit with conventional messages, then `gh pr create --draft` if appropriate.

## After run

```bash
git worktree remove --force "git/wt/<path-used>"
```

Keep **mirrors**; remove **all** worktrees created this run.
