# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/).

## [Unreleased]

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

