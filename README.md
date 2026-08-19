# gitrelease

Prepares a local git project for public release on GitHub in one pass: adds an MIT license if missing, verifies README and `.gitignore`, creates a GitHub remote if one doesn't exist, applies branch protection requiring pull requests, and cuts a tagged release.

## Highlights

- **Idempotent** — every step checks current state first and reports "already exists, skipping" rather than clobbering existing files or config
- **Doesn't overreach on docs** — if README.md is missing or a stub, it warns and points at `/readme` rather than generating one itself
- **Sane branch protection defaults** — requires 1 approving PR review, dismisses stale reviews on new pushes, blocks force-push and branch deletion, while leaving admin bypass on so the owner can still push directly
- **Version-aware releases** — suggests `v1.0.0` for a first release, or the next patch/minor/major based on the latest existing tag
- **Full summary table** — reports exactly what was done vs. skipped at each of the 7 steps

## Usage

Invoke `/gitrelease` (or say "release this," "make this public," "set up GitHub for this project") from inside a git repository you want to prepare for public release. It walks through, in order:

1. Confirms the current directory is a git repo and identifies the default branch
2. Adds an MIT `LICENSE` if one doesn't exist
3. Checks `README.md` exists and has real content (warns and defers to `/readme` if not)
4. Checks `.gitignore` exists (warns if missing)
5. Creates a GitHub remote via `gh repo create --public --source=. --push` if none exists, or confirms the existing one is up to date
6. Applies branch protection (PR required, 1 approval, stale reviews dismissed, force-push and deletion blocked) via the GitHub API
7. Creates a tagged GitHub release with `gh release create --generate-notes --latest`
8. Prints a summary table of what was done or skipped at each step

## Installation

### Claude Code

```bash
cp -r ~/.claude/skills/gitrelease/ ~/.claude/skills/gitrelease/
```

Already installed if this file is at `~/.claude/skills/gitrelease/`. Invoke with `/gitrelease`.

## Contributing

This is a personal skill, but improvements are welcome — fork, branch, and open a pull request.

## License

[MIT](LICENSE)
