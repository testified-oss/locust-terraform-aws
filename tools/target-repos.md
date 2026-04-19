# Target repos (random pick — one per HEARTBEAT)

Each HEARTBEAT run processes **exactly one** repository, chosen **uniformly at random** from the pool below (do not round-robin unless you change this file).

## Pool (`owner/name`)

| URL | owner/name |
|-----|------------|
| https://github.com/testified-oss/behave-bdd-python | `testified-oss/behave-bdd-python` |
<!-- | https://github.com/testified-oss/awesome-testing-resources | `testified-oss/awesome-testing-resources` | -->

## Random selection (required)

After Section B (`gh auth`), pick **`OWNER/REPO`** for this run using **one** of:

```bash
printf '%s\n' "testified-oss/behave-bdd-python" "testified-oss/awesome-testing-resources" | shuf -n1
```

(or equivalent: two-line file + `shuf -n1`, or documented RNG—**must** be random each run, not always the first row).

Log the chosen repo in **`memory/YYYY-MM-DD.md`** (e.g. `picked_repo=testified-oss/behave-bdd-python`).

## Rules

- **No** `gh repo list` for scope—the pool is **only** these two rows unless a human edits this file.
- **Git worktrees skill** applies **before** any `git worktree` / mirror work: **`/Users/luucrew/.claude/skills/using-git-worktrees/SKILL.md`**.
- You must already check previous memory so we dont repeat work in same repository
