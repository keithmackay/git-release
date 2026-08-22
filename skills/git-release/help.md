git-release — prepare a local git project for public release on GitHub

WHAT IT DOES
  Runs an idempotent, 12-step release checklist against the current
  directory: adds an MIT license if missing, verifies README.md exists,
  checks skill/plugin projects for a --help/:help mechanism (warns and
  defers to /make-readme rather than creating one), adds a --version
  flag/:version command to skill/plugin projects that lack one (this
  step creates it directly, reporting installed version and -- best
  effort -- whether a newer GitHub release is available), verifies
  CHANGELOG.md and .gitignore exist, creates a GitHub remote if needed,
  applies branch protection requiring pull requests, bumps the
  "version" field in every plugin manifest (.claude-plugin/plugin.json,
  .codex-plugin/plugin.json, gemini-extension.json, including
  skills/<name>/ copies) to match the release tag, finalizes
  CHANGELOG.md by renaming its "Unreleased" section to the new version
  and date (never inventing changelog content, only renaming/stamping
  what's already written there -- and refusing to proceed silently if
  Unreleased is empty, asking for confirmation instead), optionally
  syncs this plugin's listing in a marketplace.json it's registered in
  (see --marketplace below), and cuts a tagged GitHub release. Each
  step checks current state first and reports "already exists,
  skipping" rather than clobbering existing files or config.

WHAT IT NEEDS
  - The current directory must be a git repository
  - The `gh` CLI must be installed and authenticated (`gh auth status`)
  - `git config user.name` set, for the LICENSE copyright holder

USAGE
  /git-release                Run the full release workflow
  /git-release --help         Show this message and exit
  /git-release --version      Show installed version and check for updates
  /git-release --marketplace  Sync this skill/plugin's listing in a marketplace.json

FLAGS
  --help          Show this help message without making any changes
  --version       Show the installed version and check for a newer release
  --marketplace   For skill/plugin projects only: add or update this
                  project's entry in a marketplace.json (asks for its
                  location, remembering it for next time), prefixes its
                  description with the current version, sets its listed
                  repo to this project's public GitHub remote, and
                  updates the marketplace's README.md entry -- linking
                  to help.md/CHANGELOG.md if this project has them.
                  Also adds/updates a "from the marketplace" section at
                  the top of this project's own README.md Installation
                  section with the marketplace add/install commands
