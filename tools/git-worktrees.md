# Git mirrors + worktrees

## Mandatory skill

Before creating or removing worktrees, read and follow:

**`/Users/luucrew/.claude/skills/using-git-worktrees/SKILL.md`**

Announce when applying it: *using-git-worktrees skill governs layout and safety.*

## Layout (this workspace)

| Path | Purpose |
|------|---------|
| `git/mirrors/<owner>__<repo>.git` | Bare mirror (`git clone --mirror`) |
| `git/wt/<owner>__<repo>-<branch-slug>` | Ephemeral read checkout via `git worktree add` |

**Workspace root:** `/Users/luucrew/.openclaw/workspaces/testified-oss-coder`

Set **`worktrees_root`** in practice to either `git/wt` or top-level `worktrees/` — both are **gitignored** (see root `.gitignore`).

## Safety

```bash
git check-ignore -q git/wt 2>/dev/null || git check-ignore -q worktrees 2>/dev/null
```

If not ignored, fix workspace `.gitignore` before first `worktree add`.

## Mirror lifecycle

**First time** (HTTPS; `gh` auth is used by the host for `gh repo clone` if preferred):

```bash
mkdir -p git/mirrors
git clone --mirror "https://github.com/OWNER/REPO.git" "git/mirrors/OWNER__REPO.git"
```

**Update:**

```bash
git -C "git/mirrors/OWNER__REPO.git" fetch --prune
```

## Worktree (read-only scan)

Resolve default branch (example with `gh`):

```bash
gh repo view "OWNER/REPO" --json defaultBranchRef -q .defaultBranchRef.name
```

Let `BR` be that branch name. Unique path per run:

```bash
WT="git/wt/OWNER__REPO-${BR}-scan"
git -C "git/mirrors/OWNER__REPO.git" worktree add "../wt/OWNER__REPO-${BR}-scan" "$BR"
```

(Adjust relative path from mirror: from `git/mirrors/OWNER__REPO.git`, `../wt/...` resolves to `git/wt/...`.)

## After scan

```bash
git worktree remove --force "git/wt/OWNER__REPO-${BR}-scan"
```

Keep the **mirror**; remove only the worktree.

## Failure

If `worktree add` fails (path exists, lock, etc.), log to `memory/YYYY-MM-DD.md`, `git worktree prune` in the mirror if needed, and continue to the next repo.
