# GitHub — issues (v1 playbook)

## Per-HEARTBEAT flow (summary)

1. **Random repo** — exactly one of the two repos in **`tools/target-repos.md`** (see that file for `shuf` recipe).
2. **Local copy always** — mirror + **at least one worktree** on the default branch per **`tools/git-worktrees.md`** and the **using-git-worktrees** skill, **before** any issue create or “work on issue” triage.
3. **Branch on open issues + open PRs** (see **`HEARTBEAT.md`** **Routing law** / **Section E.2**):
   - If **`gh issue list --repo OWNER/REPO --state open … --jq 'length'`** is **0** → **create new issue** path (scan + dedupe + `gh issue create`).
   - If **≥ 1** open issue **and** **`gh pr list --repo OWNER/REPO --state open … --jq 'length'` ≥ 1** → **idle** — **no** `gh issue create`, **no** commits, **no** new PR, **no** issue comment; log and teardown.
   - If **≥ 1** open issue **and** **0** open PRs → **work on issue** path only (**no** `gh issue create`): pick an open issue, **`gh issue view`**, **second worktree** on a **topic branch** per **`tools/conventions.md`** / **`HEARTBEAT.md` Section G**, implement, **commit**, **`gh pr create --draft`** when there is at least one commit, then **`gh issue comment`** linking the PR—never merge without human approval.

## Config (edit for your org)

**Current values:**

- **`issue_template_title`:** `Bug report` — must match a template **`name:`** on the repo; **verify** per operator section below.
- **`max_issues_per_run`:** `1` (one heartbeat = one repo; at most one new issue when the create path runs)
- **`dedupe_search_limit`:** `20`

| Key | Example | Meaning |
|-----|---------|---------|
| `issue_template_title` | `Bug report` | Template **`name:`** for **`gh issue create --template`** |
| `max_issues_per_run` | `1` | Cap **new** issues created when the “no open issues” path runs |

## Open issue count (branching)

```bash
gh issue list --repo "OWNER/REPO" --state open --json number --jq 'length'
```

## Open PR count (when open issues ≥ 1)

```bash
gh pr list --repo "OWNER/REPO" --state open --json number --jq 'length'
```

If this is **≥ 1** while open issues are **≥ 1**, use the **idle** path (**`HEARTBEAT.md` Section E.2**): do nothing on the repo.

## Create path (no open issues)

1. Scan files per **`tools/scan-paths.md`** inside the **default-branch worktree** (local copy required).
2. Dedupe: `gh issue list --repo "OWNER/REPO" --state all --search "<keywords>" --limit 20` if you need to avoid reopening closed dupes.
3. `gh issue create` as below.

## Work-on-issue path (one or more open issues **and zero open PRs**)

**Precondition:** **`gh pr list --repo "OWNER/REPO" --state open --json number --jq 'length'`** must be **0**. Otherwise **idle**, not this section.

1. List: `gh issue list --repo "OWNER/REPO" --state open --json number,title,labels --limit 30`
2. **Pick one** issue `N` (document choice in memory).
3. **`gh issue view N --repo "OWNER/REPO"`** (web or `--json` as needed).
4. **Topic-branch worktree:** `git worktree add` a second path from the mirror when needed (governed by **using-git-worktrees**). Read relevant paths and **implement** a smallest coherent change for issue `N`; branch naming per **`tools/conventions.md`**.
5. **Commit** on the topic branch (messages per **`tools/conventions.md`**).
6. **`gh pr create --draft`** when there is at least one commit — see **`tools/github-prs.md`** and **`HEARTBEAT.md` Section G**.
7. **`gh issue comment N --repo "OWNER/REPO" --body-file scratch/comment-N.md`** — required: concrete repo-relative paths, summary, and **draft PR link** when step 6 ran. If no commits were possible, explain in the comment; still **do not** `gh issue create` while other issues are open.

If **`gh issue create --template`** fails with “template not found”, **log and skip** create (no interactive prompt).

### Issue forms (YAML) only

GitHub CLI may not support some YAML issue forms. If so: log to memory, use body-only create only if allowed here, or skip create.

## Dedupe (before create)

```bash
gh issue list --repo "OWNER/REPO" --state open --search "keywords here" --limit 20
```

If a substantively duplicate **open** issue exists, **do not** create another.

## `gh issue create` — exact flags

```bash
gh issue create \
  --repo "OWNER/REPO" \
  --title "short imperative title" \
  --template "ISSUE_TEMPLATE_NAME_FROM_TOOLS" \
  --body-file "/Users/luucrew/.openclaw/workspaces/testified-oss-coder/scratch/issue-body-OWNER-REPO.md"
```

Replace `ISSUE_TEMPLATE_NAME_FROM_TOOLS` with **`issue_template_title`** above.

Fallback (body-only), only if operators document approval in this file:

```bash
gh issue create --repo "OWNER/REPO" --title "..." --body-file "..."
```

## Scratch files

Use `scratch/issue-body-*.md` and `scratch/comment-*.md`; directory is gitignored.

## Operator verification (run once per repo template)

On **`testified-oss/behave-bdd-python`** and **`testified-oss/awesome-testing-resources`**:

1. `gh api "repos/OWNER/REPO/contents/.github/ISSUE_TEMPLATE" --jq '.[].name'` (or GitHub UI **Issues → New issue**).
2. Align **`issue_template_title`** with the real **`name:`** string (may differ per repo—if so, document both in this file or pick a template name common to both).
