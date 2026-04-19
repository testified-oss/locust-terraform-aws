# Scan paths (relative to repo root)

After a worktree is checked out, read these paths **in order** when they exist (skip missing files without failing the repo).

1. `README.md`
2. `CONTRIBUTING.md`
3. `SECURITY.md`
4. `TODO.md` — or `ROADMAP.md` if `TODO.md` missing
5. `package.json` — if present (scripts, engines)
6. `go.mod` — if present
7. `Cargo.toml` — if present
8. `.github/workflows/` — list filenames only first pass; optional second pass: read selected YAML if small

## Output expectation

Summarize: docs gaps, CI surface, dependency or test hints, security disclaimers—**actionable** bullets suitable for an issue body, with paths cited.
