git-release — prepare a local git project for public release on GitHub

WHAT IT DOES
  Runs an idempotent, 11-step release checklist against the current
  directory: adds an MIT license if missing, verifies README.md,
  CHANGELOG.md, and .gitignore exist, checks skill/plugin projects for
  a --help/:help mechanism (warns and defers to /make-readme rather
  than creating one), creates a GitHub remote if needed, applies branch
  protection requiring pull requests, bumps the "version" field in
  every plugin manifest (.claude-plugin/plugin.json,
  .codex-plugin/plugin.json, gemini-extension.json, including
  skills/<name>/ copies) to match the release tag, finalizes
  CHANGELOG.md by renaming its "Unreleased" section to the new version
  and date (never inventing changelog content, only renaming/stamping
  what's already written there), offers to bump this plugin's version
  in any locally-checked-out marketplace.json that lists it, and cuts
  a tagged GitHub release. Each step checks current state first and
  reports "already exists, skipping" rather than clobbering existing
  files or config.

WHAT IT NEEDS
  - The current directory must be a git repository
  - The `gh` CLI must be installed and authenticated (`gh auth status`)
  - `git config user.name` set, for the LICENSE copyright holder

USAGE
  /git-release              Run the full release workflow
  /git-release --help       Show this message and exit

FLAGS
  --help    Show this help message without making any changes
