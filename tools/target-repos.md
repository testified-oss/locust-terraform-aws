# Target repos (allowlist)

Only these **`owner/name`** entries are in scope for mirror sync, worktree scan, and issue filing. **Edit this list** before relying on automation; empty or placeholder rows are skipped by HEARTBEAT until populated.

| owner/name | default branch (if not default) | notes |
|------------|----------------------------------|--------|
| `testified-oss/REPLACE_ME` | *(use `gh repo view --json defaultBranchRef`)* | Replace with real repos |

## Rules

- One repo per row; use full `owner/name` as `gh` expects.
- Do **not** rely on `gh repo list` for scope—this file is the source of truth.
- Optional: track last processed `HEAD` SHA in `memory/repo-cursor.json` (see `tools/github-issues.md`).
