# HEARTBEAT.md — Testified-OSS improvement (random repo + worktrees + issues)

**Workspace root:** `/Users/luucrew/.openclaw/workspaces/testified-oss-coder`

Gateway and cron must read **exactly**:

`/Users/luucrew/.openclaw/workspaces/testified-oss-coder/HEARTBEAT.md`

**Git / local copy:** **Always** use a **bare mirror + worktree** for the chosen repo before reading files for scans/implementations, filing a new issue, or working an open issue. Announce and follow the **using-git-worktrees** skill: **`/Users/luucrew/.claude/skills/using-git-worktrees/SKILL.md`** — it **governs layout and safety** for every `git worktree` operation.

**Cron:** `/Users/luucrew/.openclaw/cron/jobs.json` — job `testified-oss-coder`.

---

## Routing law (read before Sections D–G)

Decisions use **only** the **`REPO`-scoped probes** in **Section E** (not “does it feel empty?”, not another repo, not the current shell’s default `gh` repo).

| `has_open_issues` (Section E) | `has_open_prs` (Section E.2) | After **Section D** |
|------------------------------|------------------------------|----------------------|
| **0** | *(skip E.2)* | **Section F** only — `gh issue create` allowed (still run **Section F preflight**). |
| **non‑zero** | **1** | **Idle** — **Section H** only (no F, no G): no new issue, no commits, no new PR, no issue comment. |
| **non‑zero** | **0** | **Section G** only — no F, no `gh issue create`. |

**Important:** With the **canonical** probe below, **`has_open_issues` is only ever `0` or `1`** (`-L 1` caps the JSON array). If anyone runs `gh issue list` **without** `-L 1`, `jq 'length'` can be **`2`, `3`, …** — then treating “has issues” as **equals `1`** is wrong and **skips Section E.2**, which leads to duplicate **`gh issue create`** and no PR work. Treat **any value other than `0`** the same as **`1`** in the two rows above (or always use the exact `-L 1` command).

**Forbidden:** **`gh issue create`** whenever **`has_open_issues` is not `0`** (with the canonical probe, that is **`1`**).

Re-run **Section E probes** immediately before **`gh issue create`** (**Section F preflight**). If **`has_open_issues`** is no longer **0**, **abort** create and follow this table (E.2 → G or Idle from **current** probe results).

---

## When you are done

After **Sections A–I**: playbooks loaded, auth OK, **one random repo** selected, **`REPO`** set, **E/E.2** probes logged, **mirror + worktree** from **D**, outcome **one of**: **new issue created** (`has_open_issues` was 0), **open issue worked** (G), or **idle**, teardown + memory written, and your **final assistant message** is the **full run summary** for Discord **`1069257061533233182`** when cron **`announce`** applies.

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

- If **`has_open_issues` is `0`** → set `run_path=create`; **skip Section E.2**; continue to **Section D**, then **Section F**.
- If **`has_open_issues` is not `0`** (canonical probe: equals **`1`**; if you mistakenly omitted `-L 1`, this is **any** `length` ≥ 1) → continue to **Section E.2** (do **not** run Section F).

### Section E.2 — Open PRs? (**only** when `has_open_issues` is **not** `0`)

```bash
has_open_prs=$(gh pr list --repo "$REPO" --state open -L 1 --json number --jq 'length')
```

**Hard gate:** same as Section E — on `gh`/`jq` failure, log `probe_failed=open_prs` and **STOP** (no create, no idle/work inference from missing data).

Log `has_open_prs=…` and raw output line to **`memory/YYYY-MM-DD.md`**.

- If **`has_open_prs` is not `0`** (canonical: **`1`**) → `run_path=idle`. Continue to **Section D**, then **Section H** (no F, no G).
- If **`has_open_prs` is `0`** → `run_path=work`. Continue to **Section D**, then **Section G**.

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

### Section F preflight — **mandatory** immediately before `gh issue create`

Re-run the **open-issue** probe (same as Section E; catches drift and agent mistakes):

```bash
has_open_issues=$(gh issue list --repo "$REPO" --state open -L 1 --json number --jq 'length')
```

- If **`has_open_issues` is not `0`**: **STOP**. Do **not** `gh issue create`. Log **`preflight_abort=issue_create_blocked_open_issues_exist`** to memory. Run **`has_open_prs=$(gh pr list --repo "$REPO" --state open -L 1 --json number --jq 'length')`**; if **`has_open_prs` is not `0`** → **Section H** (idle); if **`has_open_prs` is `0`** → **Section G** (work on issue).

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
5. **Commit** — every message must match **`tools/conventions.md`**. Prefer PR body/footer **`Fixes #N`** when the change closes the issue; otherwise reference `#N` in the PR body.
6. **`gh pr create --draft`** per **`tools/github-prs.md`** — **required** when there is at least one new commit: **push the topic branch to GitHub first** (mirror worktrees: use the **URL `git push`** recipe in **`tools/github-prs.md`** — plain `git push origin <branch>` often fails with **`--mirror can't be combined with refspecs`**); then run **`gh pr create --draft`**. No merge without human. If there are **zero** commits, skip PR creation and explain in the comment (step 7).
7. **`gh issue comment "$N" --repo "$REPO" --body-file scratch/comment-N.md`** — **required** last: summarize what changed (or why nothing could ship), repo-relative paths, and **link the draft PR** when step 6 ran. If step 6 was skipped, say so explicitly. **Never** call `gh issue create` while **`has_open_issues` is not `0`**.

---

## Section H — Teardown + memory

1. **`git worktree remove --force`** for every worktree path created this run (scan + issue branch if any). Keep mirrors.
2. Append **`memory/YYYY-MM-DD.md`**: `REPO`, `run_path`, `has_open_issues`, `has_open_prs` (if computed), picked issue/PR URLs, errors.

## Section I — Final message (Discord / operator)

Full summary: **`$REPO`** (random), **`run_path`** (**create** / **work** / **idle**), probe values, worktree paths used, files touched, issue/PR links, skips/errors. This is the **`announce`** payload to **`1069257061533233182`**.
