# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/).

## [Unreleased]

- `--marketplace` now mirrors the project's GitHub repo description into its marketplace.json listing (version-prefixed) instead of preserving a separately drifting description
- Add Step 9: ensure the GitHub repo `description` field is set, proposing a one-line description (derived from the project's manifest/README) that the user can accept or replace
- Add `--dry-run` flag: preview the full release workflow (or just `--marketplace`'s sync) and report what would happen at every mutating step, without creating/modifying/committing/pushing/tagging or calling any mutating GitHub API
- Document mackayi marketplace installation in README
- Fix manifest version drift: .codex-plugin/plugin.json and gemini-extension.json said 1.0.0 despite the v1.0.1 release tag
- `--marketplace` now also adds/updates a "from the marketplace" section at the top of the project's own README.md Installation section, with the marketplace's registration and install commands
- Add `--marketplace` flag: interactively locate a marketplace.json (asking, then remembering the path per project in a local config file), add/update this project's listing with a version-prefixed description and its public repo as source, and sync the marketplace's README.md entry — linking to help.md/CHANGELOG.md when present; Step 11 now reuses this same procedure
- Add its own --version flag (Flags section), matching what Step 5 adds to other skills/plugins
- Gate Step 10's CHANGELOG finalization: refuse to proceed silently if Unreleased is empty, ask the user to confirm instead
- Add Step 5: add a --version flag/:version command to skill/plugin projects that lack one, reporting installed version and a best-effort GitHub update check
- Add optional Step 10: offer to bump this plugin's version in any locally-checked-out marketplace.json that lists it
- Add Changelog section to README linking CHANGELOG.md
- Check for a help mechanism on skill/plugin projects, defer creation to /make-readme
- Bump plugin manifest versions to match release tag (Step 8)
- Add CHANGELOG.md seeded from commit history
- Add CHANGELOG.md check and release-time finalization (Steps 5, 9-10)

## [1.0.1] - 2026-08-19

- Rename skill to git-release for kebab-case consistency
- Add --help flag documentation to git-release skill
- Move --help text into separate help.md files
- Document the help.md convention in README
- Add .gitignore for OS files and local sessionstats state

## [1.0.0] - 2026-08-19

- Initial commit: gitrelease skill, README, MIT license
- Initial commit
- Port gitrelease skill to Codex and Gemini CLI, document install/compatibility

