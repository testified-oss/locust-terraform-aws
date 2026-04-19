# HEARTBEAT.md — Testified-OSS improvement (random repo + worktrees + issues)

**Workspace root:** `/Users/luucrew/.openclaw/workspaces/testified-oss-coder`

Gateway and cron must read **exactly**:

`/Users/luucrew/.openclaw/workspaces/testified-oss-coder/HEARTBEAT.md`

**Git / local copy:** **Always** use a **bare mirror + worktree** for the chosen repo before reading files for scans/implementations, filing a new issue, or working an open issue. Announce and follow the **using-git-worktrees** skill: **`/Users/luucrew/.claude/skills/using-git-worktrees/SKILL.md`** — it **governs layout and safety** for every `git worktree` operation.

**Cron:** `/Users/luucrew/.openclaw/cron/jobs.json` — job `testified-oss-coder`.

---

## Routing law (read before Sections D–G)

Decisions use **only** the **`REPO`-scoped probes** in **Section E** (not “does it feel empty?”, not another repo, not the current shell’s default `gh` repo).

| `has_open_issues` (Section E) | `has_open_prs` (Section E.2 — **always** run after E) | After **Section D** |
|------------------------------|------------------------------------------------------|----------------------|
| **0** | **0** | **Section F** only — `gh issue create` allowed (still run **Section F preflight**). |
| **0** | **non‑zero** | **PR follow-up** — **Section J** (CI + PR/review comments on one open PR), then **Section H**. **No** `gh issue create`, **no** `gh pr create`, **no** merge. **Commits + push** only on the **picked PR’s head branch** when fixing checks or feedback. |
| **non‑zero** | **non‑zero** | **PR follow-up** — **Section J**, then **Section H**. Same red lines as the row above (no new issue, no second PR, no merge). |
| **non‑zero** | **0** | **Section G** only — no F, no `gh issue create`. |

**Important:** With the **canonical** probe below, **`has_open_issues` / `has_open_prs` are each only ever `0` or `1`** (`-L 1` caps the JSON array). If anyone runs `gh issue list` / `gh pr list` **without** `-L 1`, `jq 'length'` can be **`2`, `3`, …** — then treating “has issues” as **equals `1`** only is wrong. Treat **any value other than `0`** the same as **“at least one”** for routing (or always use the exact `-L 1` commands).

**Forbidden:** **`gh issue create`** whenever **`has_open_issues` is not `0`** (with the canonical probe, that is **`1`**).

Re-run **Section E and E.2 probes** immediately before **`gh issue create`** (**Section F preflight**). If **`has_open_issues`** is no longer **0**, or **`has_open_prs`** is no longer **0**, **abort** create and route to **Section J** (if PRs exist) or **Section G** (issues only) per **Section F preflight** bullets.

---

## When you are done

After **Sections A–I**: playbooks loaded, auth OK, **one random repo** selected, **`REPO`** set, **E/E.2** probes logged, **mirror + worktree** from **D**, outcome **one of**: **new issue created** (`run_path=create`), **open issue worked** (G), **PR follow-up** (J — includes **zero open issues** but **≥1 open PR**), or **no-op within J** (checks green, nothing actionable), teardown + memory written, and your **final assistant message** is the **full run summary** for Discord **`1069257061533233182`** when cron **`announce`** applies.

Do **not** end with only `HEARTBEAT_OK` or any token-only line.

---

## Section A — Read index + playbooks

1. **`TOOLS.md`** — confirm startup order.
2. **`tools/conventions.md`**, **`tools/target-repos.md`**, **`tools/git-worktrees.md`**, **`tools/github-issues.md`**, **`tools/github-prs.md`**, **`tools/scan-paths.md`**.
3. Extract **`issue_template_title`**, **`max_issues_per_run`**, and the **two-repo random pool** from `target-repos.md` / `github-issues.md`.

## Section B — Auth

```bash
gh auth status
```

On failure: log to `memory/YYYY-MM-DD.md` and stop.

## Section C — Random repo (exactly one)

Pick **`OWNER/REPO`** for this run **uniformly at random** from the two entries in **`tools/target-repos.md`** (use the `shuf` recipe there). Log `picked_repo=…` to **`memory/YYYY-MM-DD.md`**.

**Immediately set** (same string as `picked_repo`, **no** host prefix unless your pool uses it):

```bash
REPO="OWNER/REPO"
```

**Every** `gh issue …` / `gh pr …` / `gh repo …` this run **must** pass **`--repo "$REPO"`** (or **`-R "$REPO"`**). Do **not** rely on the shell’s implicit `gh` default repository.

---

## Section E — Open issues? (**run before Section D** — route gate)

Use the **existence probe** (with **`-L 1`**, `jq 'length'` is **only `0` or `1`** — that is “none” vs “at least one”, not total count):

```bash
has_open_issues=$(gh issue list --repo "$REPO" --state open -L 1 --json number --jq 'length')
```

**Hard gate — probe must succeed:** if `gh` exits non‑zero, prints nothing useful, or `has_open_issues` is empty / not an integer → **STOP** this run after logging `probe_failed=open_issues` (and stderr) to **`memory/YYYY-MM-DD.md`**. **Do not** assume zero open issues; **do not** `gh issue create`.

Log to **`memory/YYYY-MM-DD.md`**: `REPO=…`, `has_open_issues=…`, and the **raw command + stdout** (one line) so the run is auditable.

- If **`has_open_issues` is `0`** → continue to **Section E.2** (still probe PRs — open PRs without open issues must enter **Section J**, not **Section F**).
- If **`has_open_issues` is not `0`** (canonical probe: equals **`1`**; if you mistakenly omitted `-L 1`, this is **any** `length` ≥ 1) → continue to **Section E.2** (routing is decided **after** both probes; **Section F** only if `run_path=create`).

### Section E.2 — Open PRs? (**always** after Section E)

```bash
has_open_prs=$(gh pr list --repo "$REPO" --state open -L 1 --json number --jq 'length')
```

**Hard gate:** same as Section E — on `gh`/`jq` failure, log `probe_failed=open_prs` and **STOP** (no create, no idle/work inference from missing data).

Log `has_open_prs=…` and raw output line to **`memory/YYYY-MM-DD.md`**.

**Set `run_path` from both probes:**

- **`has_open_issues` is `0`** and **`has_open_prs` is `0`** → `run_path=create`. Continue to **Section D**, then **Section F**.
- **`has_open_issues` is `0`** and **`has_open_prs` is not `0`** → `run_path=pr_followup`. Continue to **Section D**, then **Section J**, then **Section H** (no F, no G).
- **`has_open_issues` is not `0`** and **`has_open_prs` is not `0`** → `run_path=pr_followup`. Continue to **Section D**, then **Section J**, then **Section H** (no F, no G).
- **`has_open_issues` is not `0`** and **`has_open_prs` is `0`** → `run_path=work`. Continue to **Section D**, then **Section G**.

---

## Section D — Mirror + default-branch worktree (always)

For **`$REPO`** only:

1. Re-read **using-git-worktrees** skill; **`git check-ignore`** on worktree root per **`tools/git-worktrees.md`**.
2. Ensure **`git/mirrors/OWNER__REPO.git`** exists (`OWNER`/`REPO` parsed from `REPO`); `fetch --prune`.
3. Resolve default branch: `gh repo view "$REPO" --json defaultBranchRef -q .defaultBranchRef.name`
4. **`git worktree add`** to **`git/wt/OWNER__REPO-<BR>-scan`** on that branch (see **`tools/git-worktrees.md`**).
5. All file reads for scan/issue text use **this local tree**—not GitHub web alone.

---

## Section F — Create new issue (**only** when `run_path=create`)

**Precondition from routing:** **`has_open_issues` is `0`** and **`has_open_prs` is `0`** (Section E.2). If you reached this section any other way, **STOP** — routing bug.

### Section F preflight — **mandatory** immediately before `gh issue create`

Re-run **both** existence probes (same as Sections E and E.2; catches drift, new PRs, and agent mistakes):

```bash
has_open_issues=$(gh issue list --repo "$REPO" --state open -L 1 --json number --jq 'length')
has_open_prs=$(gh pr list --repo "$REPO" --state open -L 1 --json number --jq 'length')
```

- If **`has_open_issues` is not `0`**: **STOP**. Do **not** `gh issue create`. Log **`preflight_abort=issue_create_blocked_open_issues_exist`** to memory. If **`has_open_prs` is not `0`** → **Section J** (PR follow-up) then **Section H**; if **`has_open_prs` is `0`** → **Section G** (work on issue).
- If **`has_open_issues` is `0`** but **`has_open_prs` is not `0`**: **STOP**. Do **not** `gh issue create`. Log **`preflight_abort=issue_create_blocked_open_pr_exists`** to memory → **Section J** (PR follow-up) then **Section H**.

Optional belt-and-suspenders (human-readable):

```bash
gh issue list --repo "$REPO" --state open -L 5 --json number,title
```

If that JSON array is **non-empty**, treat as **open issues exist** even if the probe misbehaved.

### Section F — create steps (only if preflight passed)

1. Read paths in **`tools/scan-paths.md`** under the **scan worktree**.
2. **Dedupe** per **`tools/github-issues.md`** (open + closed search as there).
3. **`gh issue create`** with **`--repo "$REPO"`**, **`--title`**, **`--template`** = `issue_template_title`, **`--body-file`** under **`scratch/`** — exact contract in **`tools/github-issues.md`**. Respect **`max_issues_per_run`**.
4. Suggested commit lines in the body must match **`tools/conventions.md`**.
5. If `--template` fails: log, skip create.

---

## Section G — Work on existing issue (`run_path=work`)

**Precondition:** **`has_open_issues` is not `0`** and **`has_open_prs` is `0`** (from **Section E.2**, or from **Section F preflight** after an abort). With **`-L 1`** probes, “not 0” / “0” read as **`1`** / **`0`**.

**Goal:** Land a **real change** from a **topic branch** in a **worktree**, **push** (or `gh` equivalent), open a **draft PR**, then **comment on the issue** with the PR link. **Do not** file a new issue on this path.

1. `gh issue list --repo "$REPO" --state open --json number,title,labels -L 30` — **pick one** issue `N` (prefer smallest `number` or `good first issue` when present); log `picked_issue=N` in memory.
2. **`gh issue view "$N" --repo "$REPO"`** (full context before edits).
3. **Worktree + branch:** **`git worktree add -b <topic-branch> …`** from the **same mirror** (second path preferred); naming in **`tools/conventions.md`** / **`tools/git-worktrees.md`**. The **using-git-worktrees** skill governs paths and safety.
4. **Implement** in that worktree: smallest coherent fix or doc change that addresses `N` (or a clear slice of it). If upstream already fixed it, **one** commit that only adds a comment or doc note is still valid—avoid empty PRs.
5. **Commit** — every message must match **`tools/conventions.md`**. Prefer PR body/footer **`Fixes #N`** when the change closes the issue; otherwise reference `#N` in the PR body. Never commit markdown files generated in session.
6. **`gh pr create --draft`** per **`tools/github-prs.md`** — **required** when there is at least one new commit: **push the topic branch to GitHub first** (mirror worktrees: use the **URL `git push`** recipe in **`tools/github-prs.md`** — plain `git push origin <branch>` often fails with **`--mirror can't be combined with refspecs`**); then run **`gh pr create --draft`**. No merge without human. If there are **zero** commits, skip PR creation and explain in the comment (step 7).
7. **`gh issue comment "$N" --repo "$REPO" --body-file scratch/comment-N.md`** — **required** last: summarize what changed (or why nothing could ship), repo-relative paths, and **link the draft PR** when step 6 ran. If step 6 was skipped, say so explicitly. **Never** call `gh issue create` while **`has_open_issues` is not `0`**.

---

## Section J — PR follow-up (`run_path=pr_followup`)

**Precondition:** **`has_open_prs` is not `0`** (Section E.2). **`has_open_issues`** may be **`0`** (PR-only triage) or **non‑zero** (triage before more issue work). **Goal:** Inspect **one** open PR’s **CI/checks** and **conversation + review** comments; if something is broken or clearly actionable, fix it **on that PR’s existing head branch** and push—**never** open a **new** PR and **never** `gh issue create`.

**Forbidden (unchanged red lines):** `gh issue create`, **`gh pr create`** (second PR), merge, force-push (unless a human updates **`tools/github-prs.md`**).

**Playbook:** **`tools/github-prs.md`** (Section J / checks / comments).

1. **List** open PRs: `gh pr list --repo "$REPO" --state open --json number,title,isDraft,headRefName,url -L 15`. **Pick one** PR number **`P`** — prefer **`isDraft: true`** when multiple exist, else **lowest `number`**. Log `picked_pr=P` and URL to **`memory/YYYY-MM-DD.md`**.
2. **Checks / build:** `gh pr view "$P" --repo "$REPO" --json statusCheckRollup,mergeStateStatus,headRefName,baseRefName` and **`gh pr checks "$P" --repo "$REPO"`** (treat non‑zero exit + stderr as “investigate”; parse table/JSON as in **`github-prs.md`**). Log a one-line summary: `checks_pass|checks_fail|checks_pending|checks_unknown`.
3. **Comments:** Pull **issue** comments on the PR thread and **review** line comments per **`tools/github-prs.md`** (`gh pr view` / `gh api`). Note **requested changes** or **blocking** bot/CI text. Log whether **`actionable_feedback=yes|no`**.
4. **No-op path:** If checks are **green** (or only allowed neutral/skipped per playbook) **and** `actionable_feedback=no` → log `pr_followup_outcome=no_changes`; **do not** add a second worktree for edits; continue to **Section H**.
5. **Fix path:** Otherwise (`checks_fail`, **`pending` older than playbook staleness**, or **`actionable_feedback=yes`**):
   - **`gh pr view "$P" --repo "$REPO"`** (or `--json`) to confirm **`headRefName`** (branch to push to).
   - From the **mirror** used in Section D: **`git fetch origin "$HEAD"`** (or `git fetch --prune` then fetch the head ref GitHub shows for that PR).
   - **`git worktree add`** a **PR worktree** on **`headRefName`** (path pattern **`tools/git-worktrees.md`** / **`github-prs.md`** — e.g. `…-pr-${P}`). **Do not** checkout default branch for implementation here.
   - **Implement** smallest fix for CI or review (conventional commits per **`tools/conventions.md`**).
   - **Push** with the **HTTPS URL refspec** from **`tools/github-prs.md`** (same branch name as **`headRefName`**). **Do not** `gh pr create`.
   - **`gh pr comment "$P" --repo "$REPO" --body-file scratch/pr-followup-P.md`** — what changed, paths, and whether checks should re-run.
6. If you **cannot** safely fix (permissions, flaky external only, needs human): log `pr_followup_outcome=blocked` with reason; optional **`gh pr comment`** only if it adds real signal (no spam).

---

## Section H — Teardown + memory

1. **`git worktree remove --force`** for every worktree path created this run (scan + issue branch if any). Keep mirrors.
2. Append **`memory/YYYY-MM-DD.md`**: `REPO`, `run_path` (**create** / **work** / **pr_followup**), `has_open_issues`, `has_open_prs` (if computed), picked issue/PR URLs, **`pr_followup_outcome`** when Section J ran, errors.

## Section I — Final message (Discord / operator)

Full summary: **`$REPO`** (random), **`run_path`** (**create** / **work** / **pr_followup**), probe values, worktree paths used, files touched, issue/PR links, **`picked_pr`** / checks summary / **`pr_followup_outcome`** when applicable, skips/errors. This is the **`announce`** payload to **`1069257061533233182`**.
