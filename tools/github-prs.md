# GitHub — PRs (stub)

When **working an open issue** (`tools/github-issues.md`, **`HEARTBEAT.md` Section G**), open a **draft** PR from the topic branch when there is at least one commit (typically from a **second worktree**; same mirror layout as [`git-worktrees.md`](git-worktrees.md)). Use **`--repo "$REPO"`** on all **`gh pr`** calls. If **`has_open_prs` is not `0`** from **`HEARTBEAT.md` Section E.2** (canonical probe: **`1`**), you are on the **idle** path — **do not** create another PR this run.

## Rules (do not duplicate)

- Branch names: see [`conventions.md`](conventions.md).
- **Every** commit message and PR title: [`conventions.md`](conventions.md) only.

## Commands (reference)

```bash
gh pr list --repo "OWNER/REPO" --state open
gh pr view <number> --repo "OWNER/REPO"
```

## Push then draft PR (**mirror worktrees** — required reading)

Worktrees created from **`git clone --mirror`** inherit **`remote.origin.mirror=true`**. In that setup:

```bash
git push -u origin <your-topic-branch>
```

often fails with:

```text
fatal: --mirror can't be combined with refspecs
```

Then **`gh pr create`** fails with *“you must first push the current branch…”* or GraphQL errors (*head sha blank*, *no commits between base and head*) because **GitHub never received the branch**.

**Fix (use every Section G run after commits on a topic branch):**

1. From the **topic-branch worktree** (same place you committed), set **`BRANCH=$(git branch --show-current)`** and ensure **`REPO`** is still `owner/name` from **`HEARTBEAT.md` Section C**.
2. **Push without using the mirror remote’s refspec rules** — push to a full GitHub URL (same repo; `gh auth` / credential helper supplies HTTPS auth):

```bash
git push "https://github.com/${REPO}.git" "HEAD:refs/heads/${BRANCH}"
```

3. Then create the draft PR (branch now exists on GitHub):

```bash
gh pr create --draft --repo "$REPO" --head "$BRANCH" --title "type(scope): …" --body "…"
```

4. Log **verbatim** push/PR stderr on failure in **`memory/YYYY-MM-DD.md`**. **Do not** claim “branch pushed” or “PR opened” unless **`git push`** and **`gh pr create`** both exited **0**.

## Policy

- No merge without explicit human approval.
- No force-push from automation unless a human updates this file to allow it.
