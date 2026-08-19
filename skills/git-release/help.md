git-release — prepare a local git project for public release on GitHub

WHAT IT DOES
  Runs an idempotent, 8-step release checklist against the current
  directory: adds an MIT license if missing, verifies README.md and
  .gitignore exist, creates a GitHub remote if needed, applies branch
  protection requiring pull requests, and cuts a tagged GitHub release.
  Each step checks current state first and reports "already exists,
  skipping" rather than clobbering existing files or config.

WHAT IT NEEDS
  - The current directory must be a git repository
  - The `gh` CLI must be installed and authenticated (`gh auth status`)
  - `git config user.name` set, for the LICENSE copyright holder

USAGE
  /git-release              Run the full release workflow
  /git-release --help       Show this message and exit

FLAGS
  --help    Show this help message without making any changes
