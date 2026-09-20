# git-release

Prepares a local git project for public release on GitHub in one pass: adds an MIT license if missing, verifies README and `.gitignore`, creates a GitHub remote if one doesn't exist, applies branch protection requiring pull requests, and cuts a tagged release.

## Highlights

- **Idempotent** — every step checks current state first and reports "already exists, skipping" rather than clobbering existing files or config
- **Doesn't overreach on docs** — if README.md is missing or a stub, it warns and points at `/make-readme` rather than generating one itself
- **Sane branch protection defaults** — requires 1 approving PR review, dismisses stale reviews on new pushes, blocks force-push and branch deletion, while leaving admin bypass on so the owner can still push directly
- **Version-aware releases** — suggests `v1.0.0` for a first release, or the next patch/minor/major based on the latest existing tag
- **Full summary table** — reports exactly what was done vs. skipped at each step
- **Defers, doesn't duplicate** — for both docs and its own help mechanism, this skill only checks and warns; generating either is `/make-readme`'s job

## Usage

Invoke `/git-release` (or say "release this," "make this public," "set up GitHub for this project") from inside a git repository you want to prepare for public release. Run `/git-release --help` to print what it does, what it needs, and usage without making any changes. Run `/git-release --marketplace` on a skill/plugin project to add or update its listing in a marketplace.json (asks for its location the first time, then remembers it) — mirrors the project's GitHub repo description (the same one confirmed in the release workflow's repo-description step) into the listing, version-prefixed, points its `source` at this project's public repo, and updates the marketplace's README.md entry, linking to `help.md`/`CHANGELOG.md` if present. It also adds/updates a "from the marketplace" section at the top of this project's own README.md Installation section with the marketplace's registration and install commands. Otherwise the full workflow walks through, in order:

1. Confirms the current directory is a git repo and identifies the default branch
2. Adds an MIT `LICENSE` if one doesn't exist
3. Checks `README.md` exists and has real content (warns and defers to `/make-readme` if not)
4. For skill/plugin projects, checks for a `--help`/`:help` mechanism backed by `help.md` (warns and defers to `/make-readme` if missing — this skill checks but never creates it)
5. Checks `.gitignore` exists (warns if missing)
6. Creates a GitHub remote via `gh repo create --public --source=. --push` if none exists, or confirms the existing one is up to date
7. Ensures the GitHub repo's `description` field is set and leads with what the project *is* — e.g. `"Claude/Codex/Gemini skill: "`, `"Claude Code plugin: "`, or another fitting noun for non-skill/plugin projects — proposing a one-line description that you can accept or replace, since this is what surfaces the project's summary on GitHub itself and third-party tools like dev.to's GitHub Connections, and it's also what gets mirrored into any marketplace.json listing for this project
8. Applies branch protection (PR required, 1 approval, stale reviews dismissed, force-push and deletion blocked) via the GitHub API
9. Creates a tagged GitHub release with `gh release create --generate-notes --latest`
10. Prints a summary table of what was done or skipped at each step

This skill's own `--help` follows the same convention it checks other skills for: `SKILL.md` has a short `## Flags` section pointing at a `help.md` file in the same folder, which is read and displayed verbatim. `/make-readme` is what actually generates this pattern for a project that's missing it — see that skill's docs for details.

## Installation

### From the mackayi marketplace (recommended)

```
/plugin marketplace add keithmackay/mackayi
/plugin install git-release@mackayi
```

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

## Changelog

See [CHANGELOG.md](CHANGELOG.md) for release history.

## License

[MIT](LICENSE)
