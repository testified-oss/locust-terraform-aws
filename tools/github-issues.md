# GitHub — issues (v1 playbook)

## Config (edit for your org)

**Current values (defaults until you change them):**

- **`issue_template_title`:** `Bug report` — must match a template **`name:`** on the repo; **verify once** with `gh issue create --help` / repo UI before automation.
- **`repos_per_run`:** `3`
- **`max_issues_per_run`:** `5`
- **`dedupe_search_limit`:** `20`

| Key | Example | Meaning |
|-----|---------|---------|
| `issue_template_title` | `Bug report` | Exact **`name:`** from a repo’s `.github/ISSUE_TEMPLATE/*.md` YAML (not the filename). **Operator must verify** on one allowlisted repo before automation. |
| `repos_per_run` | `3` | Max repos to process per HEARTBEAT slice |
| `max_issues_per_run` | `5` | Cap new issues created per run |
| `dedupe_search_limit` | `20` | `gh issue list --limit` for dedupe passes |

If **`gh issue create --template`** fails with “template not found”, **log and skip** that create attempt (no interactive prompt).

### Issue forms (YAML) only

GitHub CLI may not support some YAML issue forms like classic markdown templates. If so: log to memory, skip create, or switch repo to a markdown template—do not block the whole run.

## Dedupe (before create)

For each candidate title or keyword set:

```bash
gh issue list --repo "OWNER/REPO" --state open --search "keywords here" --limit 20
```

If a substantively duplicate issue exists, **do not** create another.

## `gh issue create` — exact flags (v1)

Use a **body file** for multi-line bodies (avoids shell quoting bugs). Template supplies structure when using markdown templates; **`--title` is always required**.

```bash
gh issue create \
  --repo "OWNER/REPO" \
  --title "short imperative title" \
  --template "ISSUE_TEMPLATE_NAME_FROM_TOOLS" \
  --body-file "/Users/luucrew/.openclaw/workspaces/testified-oss-coder/scratch/issue-body-OWNER-REPO.md"
```

Replace `ISSUE_TEMPLATE_NAME_FROM_TOOLS` with the value of **`issue_template_title`** from the table above (must match GitHub’s template **name** field exactly).

**Note:** If your template does not accept `--body-file` together with `--template` for a given repo, try body-only create (no template) as a documented fallback—only if operators allow in this file:

```bash
gh issue create --repo "OWNER/REPO" --title "..." --body-file "..."
```

## Cursor / batching

- Persist slice offset or last repo index in **`memory/repo-cursor.json`** (optional JSON: `{ "nextIndex": 0 }`) or append cursor notes to **`memory/YYYY-MM-DD.md`**.
- After each run, advance `nextIndex` by processed count modulo allowlist length.

## Scratch files

Write `scratch/issue-body-*.md` under the workspace; paths are gitignored via `scratch/` in root `.gitignore`.

## Operator verification (run once before automation)

On **one** allowlisted `owner/repo`:

1. Confirm templates exist, e.g.  
   `gh api "repos/OWNER/REPO/contents/.github/ISSUE_TEMPLATE" --jq '.[].name'`  
   (or inspect the repo on GitHub: **Issues → New issue** template names).
2. Set **`issue_template_title`** in this file to the template **`name:`** string **exactly** (case-sensitive).
3. Dry-run mindset: `gh issue create --repo "OWNER/REPO" --title "test" --template "…" --body-file …` then **close the test issue** if you created one—or use a draft repo.

If no markdown template exists (forms only), expect Section F “skip on template error” until templates are added or you switch to body-only fallback documented above.
