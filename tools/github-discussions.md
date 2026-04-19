# GitHub — discussions (stub)

Future: post digests or RFC threads for allowlisted repos (same **`owner/name`** as [`target-repos.md`](target-repos.md)).

## Rules

- Link back to [`conventions.md`](conventions.md) for any copy that suggests commits or branch names.

## API sketch

Discussions are often via GraphQL, for example category/query operations—fill in when enabling:

```bash
gh api graphql -f query='query { repository(owner:\"OWNER\", name:\"REPO\") { discussions(first: 5) { nodes { title number } } } }'
```

Adjust to org needs; validate permissions (`repo` scope) before automation.
