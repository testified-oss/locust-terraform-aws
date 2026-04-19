# HEARTBEAT.md — Testified-OSS improvement (random repo + worktrees + issues)

**Workspace root:** `/Users/luucrew/.openclaw/workspaces/testified-oss-coder`

Gateway and cron must read **exactly**:

`/Users/luucrew/.openclaw/workspaces/testified-oss-coder/HEARTBEAT.md`

**Git / local copy:** **Always** use a **bare mirror + worktree** for the chosen repo before reading files, filing a new issue, or working an open issue. Announce and follow the **using-git-worktrees** skill: **`/Users/luucrew/.claude/skills/using-git-worktrees/SKILL.md`** — it **governs layout and safety** for every `git worktree` operation (both “create issue” and “work on issue” paths).

**Cron:** `/Users/luucrew/.openclaw/cron/jobs.json` — job `testified-oss-coder`.

---

## Routing law (read before Sections E–G)

| Open issues | Open PRs (`gh pr list … --state open` count) | Path this run |
|-------------|-----------------------------------------------|----------------|
| **0** | *(any)* | **Section F** — scan, dedupe, `gh issue create` at most **`max_issues_per_run`**. |
| **≥ 1** | **≥ 1** | **Idle** — **Section E.2** then **Section H** only: **no** Section F, **no** Section G (no new issue, no new commits, no new PR, no issue comment). Human/review work is already in flight. |
| **≥ 1** | **0** | **Section G** only — topic-branch worktree, **commit(s)**, **`gh pr create --draft`**, **`gh issue comment`** with PR link. |

**Forbidden:** **`gh issue create`** whenever open-issue count is **≥ 1**. If you already created an issue by mistake, stop; follow **Section G** or **Idle** per the table.

Re-check the open-issue count **immediately before** `gh issue create` (Section F). If it is no longer 0, **abort Section F** and re-apply this table (G vs Idle via **Section E.2**).

---

## When you are done

After **Sections A–I**: playbooks loaded, auth OK, **one random repo** selected, **mirror + worktree** used, outcome **one of**: **new issue created** (open issues were 0), **open issue worked** (Section G: open PRs were 0), or **idle** (open issues **and** open PRs both non-zero — no repo writes beyond memory/teardown), teardown + memory written, and your **final assistant message** is the **full run summary** for Discord **`1069257061533233182`** when cron **`announce`** applies.

Do **not** end with only `HEARTBEAT_OK` or any token-only line.

---

## Section A — Read index + playbooks

1. **`TOOLS.md`** — confirm startup order.
2. **`tools/conventions.md`**, **`tools/target-repos.md`**, **`tools/git-worktrees.md`**, **`tools/github-issues.md`**, **`tools/scan-paths.md`**.
3. Extract **`issue_template_title`**, **`max_issues_per_run`**, and the **two-repo random pool** from `target-repos.md` / `github-issues.md`.

## Section B — Auth

```bash
gh auth status
```

On failure: log to `memory/YYYY-MM-DD.md` and stop.

## Section C — Random repo (exactly one)

Pick **`OWNER/REPO`** for this run **uniformly at random** from the two entries in **`tools/target-repos.md`** (use the `shuf` recipe there). Log `picked_repo=…` to **`memory/YYYY-MM-DD.md`**.

## Section D — Mirror + default-branch worktree (always)

For the chosen `OWNER/REPO` only:

1. Re-read **using-git-worktrees** skill; **`git check-ignore`** on worktree root per **`tools/git-worktrees.md`**.
2. Ensure **`git/mirrors/OWNER__REPO.git`** exists; `fetch --prune`.
3. Resolve default branch: `gh repo view "OWNER/REPO" --json defaultBranchRef -q .defaultBranchRef.name`
4. **`git worktree add`** to **`git/wt/OWNER__REPO-<BR>-scan`** on that branch (see **`tools/git-worktrees.md`**).
5. All file reads for triage/issue text use **this local tree**—not GitHub web alone.

## Section E — Open issues? (branching gate)

```bash
gh issue list --repo "OWNER/REPO" --state open --json number --jq 'length'
```

Record `open_issue_count=…` in **`memory/YYYY-MM-DD.md`**.

- If **0** → **only** **Section F**. Do **not** run Section E.2 or Section G.
- If **≥ 1** → go to **Section E.2** (do **not** run Section F until E.2 says idle is ruled out).

### Section E.2 — Open PRs? (only when `open_issue_count >= 1`)

```bash
gh pr list --repo "OWNER/REPO" --state open --json number --jq 'length'
```

Record `open_pr_count=…` in **`memory/YYYY-MM-DD.md`**.

- If **`open_pr_count >= 1`** → **Idle path**: **no** Section F, **no** Section G. Log `run_path=idle_open_issues_and_open_prs`. Jump to **Section H**.
- If **`open_pr_count == 0`** → **only** **Section G**. **No** Section F — **no** `gh issue create` this run (see **Routing law**).

## Section F — Create new issue (**only** when `open_issue_count == 0`)

**Precondition:** Re-run the Section E command; if count is **not** 0, **stop** — apply **Routing law** (E.2 then G or Idle, never create while open issues exist).

1. Read paths in **`tools/scan-paths.md`** under the **scan worktree**.
2. **Dedupe** per **`tools/github-issues.md`** if needed (including search for open dupes before create).
3. **`gh issue create`** with **`--repo`**, **`--title`**, **`--template`** = `issue_template_title`, **`--body-file`** under **`scratch/`** — exact contract in **`tools/github-issues.md`**. Respect **`max_issues_per_run`**.
4. Suggested commit lines in the body must match **`tools/conventions.md`**.
5. If `--template` fails: log, skip create.

## Section G — Work on existing issue (`open_issue_count >= 1` **and** `open_pr_count == 0`)

**Precondition:** **Section E.2** must have recorded **`open_pr_count == 0`**. If any open PR exists, you should be on the **Idle** path instead.

**Goal:** Land a **real change** from a **topic branch** in a **worktree**, **push** (or `gh` equivalent), open a **draft PR**, then **comment on the issue** with the PR link. **Do not** file a new issue on this path.

1. `gh issue list --repo "OWNER/REPO" --state open --json number,title,labels --limit 30` — **pick one** issue `N` (prefer smallest `number` or `good first issue` when present); log `picked_issue=N` in memory.
2. **`gh issue view N --repo "OWNER/REPO"`** (full context before edits).
3. **Worktree + branch:** Use **`git worktree add -b <topic-branch> …`** from the **same mirror** for implementation work (second path is preferred over committing on the default-branch scan tree); naming in **`tools/conventions.md`** / **`tools/git-worktrees.md`**. The **using-git-worktrees** skill governs paths and safety.
4. **Implement** in that worktree: smallest coherent fix or doc change that addresses `N` (or a clear slice of it). If upstream already fixed it, **one** commit that only adds a comment or doc note is still valid—avoid empty PRs.
5. **Commit** — every message must match **`tools/conventions.md`** (conventional commits). Prefer PR body/footer **`Fixes #N`** when the change closes the issue; otherwise reference `#N` in the PR body.
6. **`gh pr create --draft`** per **`tools/github-prs.md`** — **required** when there is at least one new commit (push branch first if your flow needs it); no merge without human. If there are **zero** commits, skip PR creation and explain in the comment (step 7).
7. **`gh issue comment N --repo "OWNER/REPO" --body-file scratch/comment-N.md`** — **required** last: summarize what changed (or why nothing could ship), repo-relative paths, and **link the draft PR** when step 6 ran (number/URL from `gh pr create`). If step 6 was skipped, say so explicitly. **Never** call `gh issue create` on this path while the repo still has open issues.

## Section H — Teardown + memory

1. **`git worktree remove --force`** for every worktree path created this run (scan + issue branch if any). Keep mirrors.
2. Append **`memory/YYYY-MM-DD.md`**: picked repo, path taken (**create** / **work-on-issue** / **idle**), `open_issue_count`, `open_pr_count`, issue URLs, PR URL if any, errors.

## Section I — Final message (Discord / operator)

Full summary: **which repo** (random), **create vs work-on-issue vs idle** (if idle: say open issues **and** open PRs were both non-zero), worktree paths used, files touched, issue/PR links, skips/errors. This is the **`announce`** payload to **`1069257061533233182`**.
