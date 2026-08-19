# git-release

Prepares a local git project for public release on GitHub in one pass: adds an MIT license if missing, verifies README and `.gitignore`, creates a GitHub remote if one doesn't exist, applies branch protection requiring pull requests, and cuts a tagged release.

## Highlights

- **Idempotent** — every step checks current state first and reports "already exists, skipping" rather than clobbering existing files or config
- **Doesn't overreach on docs** — if README.md is missing or a stub, it warns and points at `/readme` rather than generating one itself
- **Sane branch protection defaults** — requires 1 approving PR review, dismisses stale reviews on new pushes, blocks force-push and branch deletion, while leaving admin bypass on so the owner can still push directly
- **Version-aware releases** — suggests `v1.0.0` for a first release, or the next patch/minor/major based on the latest existing tag
- **Full summary table** — reports exactly what was done vs. skipped at each of the 7 steps

## Usage

Invoke `/git-release` (or say "release this," "make this public," "set up GitHub for this project") from inside a git repository you want to prepare for public release. It walks through, in order:

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
cp -r /path/to/git-release/ ~/.claude/skills/git-release/
```

Or symlink:
```bash
ln -s /path/to/git-release/ ~/.claude/skills/git-release
```

Then invoke with: `/git-release`

### Codex

Place the plugin directory where Codex can find it, then add an entry to your marketplace:

**`~/.agents/plugins/marketplace.json`** (create if absent):
```json
{
  "name": "personal",
  "interface": { "displayName": "Personal Plugins" },
  "plugins": [
    {
      "name": "git-release",
      "source": { "source": "local", "path": "/path/to/git-release/" },
      "policy": { "installation": "AVAILABLE", "authentication": "ON_INSTALL" },
      "category": "Productivity"
    }
  ]
}
```

### Antigravity

**Global install** (all workspaces):
```bash
cp -r /path/to/git-release/ ~/.gemini/antigravity/skills/git-release/
```

**Workspace install** (current project only):
```bash
cp -r /path/to/git-release/ .agents/skills/git-release/
```

The root `SKILL.md` has no Claude Code-specific metadata, so it is used as-is — no separate Antigravity variant is needed.

Skills are auto-discovered. You can also mention the skill by name to force activation.

### Gemini CLI

Gemini CLI installs extensions directly from GitHub:

```bash
gemini extensions install https://github.com/keithmackay/git-release
```

To update:
```bash
gemini extensions update git-release
```

The skill is auto-discovered from `GEMINI.md` after installation.

## Compatibility

| Feature | Claude Code | Codex | Antigravity | Gemini CLI |
|---------|:-----------:|:-----:|:-----------:|:----------:|
| Core skill | ✅ | ✅ | ✅ | ✅ |

No Claude Code-specific frontmatter (`metadata`, `retrieval`, `tags`), sub-documents, or subagent dispatch is used by this skill, so there are no platform gaps to document — it ports cleanly to all four platforms.

Legend: ✅ Supported · ❌ Not supported

## References

- **Claude Code Skills:** https://code.claude.com/docs/en/skills
- **Claude Code Complete Guide (PDF):** https://resources.anthropic.com/hubfs/The-Complete-Guide-to-Building-Skill-for-Claude.pdf
- **Codex Plugins:** https://developers.openai.com/codex/plugins/build
- **Antigravity Skills:** https://antigravity.google/docs/skills
- **Gemini CLI Extensions:** https://github.com/google-gemini/gemini-cli/blob/main/docs/extension.md
- **Agent Skills open standard:** https://agentskills.io/home

## Contributing

This is a personal skill, but improvements are welcome — fork, branch, and open a pull request.

## License

[MIT](LICENSE)
