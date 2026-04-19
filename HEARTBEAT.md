# HEARTBEAT.md — Testified-OSS improvement (mirrors + worktrees + issues)

**Workspace root:** `/Users/luucrew/.openclaw/workspaces/testified-oss-coder`

**Git layout:** Announce and apply the **using-git-worktrees** skill — **`/Users/luucrew/.claude/skills/using-git-worktrees/SKILL.md`** — for directory choice, `git check-ignore`, and worktree lifecycle. That skill **governs layout and safety** for all `git worktree` operations.

**Cron registry (operators):** `/Users/luucrew/.openclaw/cron/jobs.json` — job for `agentId` `testified-oss-coder` when enabled.

---

## When you are done

After **Sections A–H**: you loaded playbooks, authenticated, processed the **repo slice**, updated memory / optional cursor, and your **final assistant message** is the **full run summary** (repos touched, issues created or skipped, errors). When cron **`announce`** applies, that summary is posted to Discord channel **`1069257061533233182`** (same pattern as `dependabot` / `remote-jobs-test`).

Do **not** end the turn with only `HEARTBEAT_OK` or any single-line token.

---

## Section A — Read index + playbooks

1. Open **`TOOLS.md`** (index) and confirm startup order.
2. Read **`tools/conventions.md`**, **`tools/target-repos.md`**, **`tools/git-worktrees.md`**, **`tools/github-issues.md`**, **`tools/scan-paths.md`**.
3. Extract from **`tools/github-issues.md`**: `issue_template_title`, `repos_per_run`, `max_issues_per_run`, dedupe limits.
4. Build the **allowlist** from **`tools/target-repos.md`** — skip placeholder rows like `testified-oss/REPLACE_ME` until replaced with real `owner/name`.

## Section B — Auth

```bash
gh auth status
```

On failure: log to `memory/YYYY-MM-DD.md` and stop.

## Section C — Select repo slice

1. Read optional **`memory/repo-cursor.json`** (`nextIndex`, integer). If missing, use `0`.
2. Take **`repos_per_run`** consecutive repos from the allowlist starting at `nextIndex` (wrap at end of list).
3. If allowlist has no valid repos, append note to memory, jump to **Section H** with summary “no valid repos configured”.

## Section D — Mirror + worktree per repo

For each `OWNER/REPO` in the slice:

1. Read **using-git-worktrees** skill; verify worktree parent is ignored (`git check-ignore` per **`tools/git-worktrees.md`**).
2. Ensure mirror exists at `git/mirrors/OWNER__REPO.git`; if not, `git clone --mirror` (or `gh repo clone OWNER/REPO git/tmp -- --mirror` then move — prefer documented commands in **`tools/git-worktrees.md`**).
3. `git -C git/mirrors/OWNER__REPO.git fetch --prune`
4. Resolve default branch: `gh repo view "OWNER/REPO" --json defaultBranchRef -q .defaultBranchRef.name`
5. `git worktree add` to `git/wt/OWNER__REPO-<branch>-scan` (see **`tools/git-worktrees.md`**).
6. **Optional:** if `HEAD` SHA matches last recorded SHA in memory for this repo, skip file read (optimization).
7. Read files per **`tools/scan-paths.md`**; produce a short internal summary of candidate improvements.

## Section E — Dedupe

For each candidate issue topic, search open issues:

```bash
gh issue list --repo "OWNER/REPO" --state open --search "<distinct keywords>" --limit 20
```

If duplicate likely exists, **do not** create.

## Section F — `gh issue create` (exact contract)

Use flags exactly as documented in **`tools/github-issues.md`**. In short:

```bash
gh issue create \
  --repo "OWNER/REPO" \
  --title "<imperative title>" \
  --template "<issue_template_title from tools/github-issues.md>" \
  --body-file "/Users/luucrew/.openclaw/workspaces/testified-oss-coder/scratch/issue-body-OWNER-REPO.md"
```

- Write the body file under **`scratch/`** first (multi-line, citations to paths).
- If `--template` fails (template not found or unsupported form): **log, skip create** for that item; continue.
- Enforce **`max_issues_per_run`** across the whole run.
- Suggested commit lines in the body **must** follow **`tools/conventions.md`**.

**Operator:** verify **`issue_template_title`** matches a template **`name:`** on at least one allowlisted repo before relying on automation.

---

## Section G — Teardown + memory

1. `git worktree remove` for each worktree created this run (keep mirrors).
2. Append audit lines to **`memory/YYYY-MM-DD.md`**: repos, SHAs, issues filed or skipped, template errors.
3. Update **`memory/repo-cursor.json`** `{ "nextIndex": <old + slice length mod n> }` if using rotation.

---

## Section H — Final message (Discord / operator)

Emit a **full run summary**: allowlist slice, repos processed, file scan highlights, issue URLs created, dedupe/template skips, errors. This text is what cron **`announce`** delivers to **`1069257061533233182`**.

---

## Run summary (this week)

- **Allowlist:** tools/target-repos.md contains 1 placeholder entry (`testified-oss/REPLACE_ME`). No processing occurred.
- **Mirrors:** none created.
- **Worktrees:** none created.
- **Scans:** none performed.
- **Issues created:** 0
- **Deduplication/skips:** not applicable
- **Errors/warnings:** Allowlist placeholder present — update with real repo(s) to enable weekly processing.
- **Next cursor:** unchanged (0)

This summary will be announced to Discord channel **1069257061533233182** via cron.