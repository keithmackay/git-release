git-release — prepare a local git project for public release on GitHub

WHAT IT DOES
  Runs an idempotent, 20-step release checklist (several apply only to
  skill/plugin projects, or only when relevant) against the current
  directory, following findsafeskills' skill-development and
  distribution best practices throughout:
    - Adds an MIT license if missing, and keeps an existing license's
      copyright year current
    - Verifies README.md exists, and warns if its install instructions
      pipe a download straight into a shell instead of
      download-verify-run
    - Checks skill/plugin projects for a --help/:help mechanism (warns
      and defers to /make-readme rather than creating one)
    - Adds a --version flag/:version command to skill/plugin projects
      that lack one, reporting installed version and -- best effort --
      whether a newer GitHub release is available
    - Ensures each skill/plugin packaging folder that actually travels
      with an install (skills/<name>/, antigravity/<name>/, etc.) has
      its own README.md naming who made it, the source repo, how to
      update it, its version, and its license
    - For skill/plugin projects with mutating actions, checks for a
      documented --dry-run flag, and checks any shipped scripts/CLIs
      for --help/--version/--dry-run support (warns only; doesn't
      write this code itself)
    - Verifies CHANGELOG.md, .gitignore, and SECURITY.md exist, and
      checks for a dependency lockfile when a package manifest is
      present
    - Creates a GitHub remote if needed
    - Ensures the GitHub repo's description field is set and leads
      with what the project is (e.g. "Claude/Codex/Gemini skill: ",
      "Claude Code plugin: ", or another fitting noun), proposing a
      one-line description you can accept or replace
    - Keeps the repo's GitHub topics in sync with its actual
      platform(s) -- adding any missing platform tag and removing any
      stale one left over from a dropped platform
    - Applies branch protection requiring pull requests
    - Bumps the "version" field in every plugin manifest
      (.claude-plugin/plugin.json, .codex-plugin/plugin.json,
      gemini-extension.json, including skills/<name>/ copies) to match
      the release tag, finalizes CHANGELOG.md by renaming its
      "Unreleased" section to the new version and date (never
      inventing changelog content, only renaming/stamping what's
      already written there -- and refusing to proceed silently if
      Unreleased is empty, asking for confirmation instead), and
      appends the same entry to versions.json if one exists
    - Optionally syncs this plugin's listing in a marketplace.json it's
      registered in (see --marketplace below)
    - Cuts a tagged GitHub release
  Each step checks current state first and reports "already exists,
  skipping" rather than clobbering existing files or config.

WHAT IT NEEDS
  - The current directory must be a git repository
  - The `gh` CLI must be installed and authenticated (`gh auth status`)
  - `git config user.name` set, for the LICENSE copyright holder

USAGE
  /git-release                Run the full release workflow
  /git-release --help         Show this message and exit
  /git-release --version      Show installed version and check for updates
  /git-release --dry-run      Preview the release workflow, no changes made
  /git-release --marketplace  Sync this skill/plugin's listing in a marketplace.json

FLAGS
  --help          Show this help message without making any changes
  --version       Show the installed version and check for a newer release
  --dry-run       Run the full checklist and report what WOULD happen at
                  every mutating step (LICENSE creation, --version
                  addition, remote/branch-protection setup, manifest
                  bump, CHANGELOG finalization, release creation,
                  marketplace sync) without creating, modifying,
                  committing, pushing, tagging, or calling any mutating
                  GitHub API. Combine with --marketplace to preview just
                  that sync. Read-only checks (README, help mechanism,
                  CHANGELOG/.gitignore presence) still run for real,
                  since there's nothing to hold back.
  --marketplace   For skill/plugin projects only: add or update this
                  project's entry in a marketplace.json (asks for its
                  location, remembering it for next time), mirrors the
                  project's GitHub repo description into the listing
                  (version-prefixed), sets its listed repo to this
                  project's public GitHub remote, and
                  updates the marketplace's README.md entry -- linking
                  to help.md/CHANGELOG.md if this project has them.
                  Also adds/updates a "from the marketplace" section at
                  the top of this project's own README.md Installation
                  section with the marketplace add/install commands
