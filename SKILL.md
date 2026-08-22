---
name: git-release
description: Use when preparing a project for public release on GitHub - adds MIT license, verifies README and .gitignore, creates a GitHub remote if needed, applies branch protection requiring PRs, and creates a tagged GitHub release
---

# gitrelease

## Overview

Prepare a local git project for public release on GitHub. Ensures the project has an MIT license, essential files, a properly configured remote, branch protection requiring pull requests, and a tagged release.

## When to Use

- First time releasing a project publicly
- Setting up a new repo for collaboration
- User says "release this", "make this public", "set up GitHub for this project"

## When NOT to Use

- Project is already released and you just want to create a new version tag (use `gh release create` directly)
- Project is not a git repository

## Flags

### `--help`

If the user invokes this skill with a `--help` flag (e.g. `/git-release --help`), do not run the workflow. Instead, read and display the contents of `help.md` (in this skill's folder) verbatim, then stop.

### `--version`

If the user invokes this skill with a `--version` flag (e.g. `/git-release --version`), do not run the workflow. Instead:

1. Read the installed version from this skill's own manifest: `.claude-plugin/plugin.json` if present, else `.codex-plugin/plugin.json`, else `gemini-extension.json` — whichever exists for this platform install. If none exist (a bare Claude Code skill with only SKILL.md), read the topmost version heading in `CHANGELOG.md` instead.
2. Print: `git-release v<installed-version>`
3. Best-effort update check — determine this skill's GitHub source repo:
   a. If `.git` exists here and `git remote get-url origin` resolves to a `github.com` URL, use that `owner/repo`.
   b. Otherwise, search this skill's own `README.md` for the first `https://github.com/<owner>/<repo>` URL and use that.
   c. If neither yields a repo, or the `gh` CLI isn't installed/authenticated: stop here. Print nothing further — no status line, no error.
4. If a repo was found: run `gh api repos/<owner>/<repo>/releases/latest -q .tag_name` (strip a leading `v`). Compare to the installed version:
   - Equal → append: `Status: up to date`
   - Installed is older → append: `Status: newer version available (v<latest>). To update: if you installed this via a Claude Code marketplace, run /plugin marketplace update <marketplace-name> then reinstall; otherwise, git pull in your install directory if it's a git checkout, or re-copy from https://github.com/<owner>/<repo> per this README's Installation section.`
   - Installed is newer → append: `Status: ahead of latest release (development checkout)`
   - If the API call fails for any reason (network, auth, rate limit, malformed tag): print nothing further — no status line, no error shown to the user.
5. Stop — do not proceed to run the skill's actual workflow.

### `--marketplace`

If the user invokes this skill with a `--marketplace` flag (e.g. `/git-release --marketplace`), do not run the release workflow. Instead, run the marketplace-sync procedure below and stop.

**Applicability check:** determine whether the current project is itself a skill or plugin, using the same detection as Step 4 (a `SKILL.md` at the project root or in a `skills/*/` subdirectory, or a `.codex-plugin/plugin.json` manifest). If it is not: stop and tell the user "This project isn't a skill or plugin — `--marketplace` only applies to skill/plugin releases."

1. **Resolve the project's public repo.** Run `git remote get-url origin` and normalize it to `owner/repo`. If there's no remote or it isn't a `github.com` URL, stop and tell the user to set up a GitHub remote first (run `/git-release` without flags, which handles this in Step 8).

2. **Locate the config file.** This skill stores its marketplace config in its own installed folder (the same directory as this `SKILL.md` and `help.md`), not the project being released — at `.git-release/marketplace-config.json`. Determine that folder by finding the `SKILL.md` currently being executed on disk (e.g. it sits alongside `help.md` and `CHANGELOG.md` for this skill's own package). Create the `.git-release/` subfolder if it doesn't exist. The file is a JSON object keyed by `owner/repo` of the project being released, valued with the absolute path to that project's chosen `marketplace.json`:
   ```json
   { "someowner/some-project": "/Users/you/Projects/some-marketplace/.claude-plugin/marketplace.json" }
   ```

3. **Ask for the marketplace location.** If the config file has an existing entry for this project's `owner/repo`, use its path as the default when asking. Ask the user: "Where is the marketplace.json for the marketplace you want to list this project in?" (showing the default, if any, so they can just confirm it).
   - If the given path is a file, use it directly.
   - If it's a folder, check `<folder>/marketplace.json` and `<folder>/.claude-plugin/marketplace.json` first.
   - If neither exists, recursively search the folder's subfolders for any file named `marketplace.json`. If exactly one is found, use it. If several are found, list them and ask the user to pick one. If none are found, tell the user and ask again for a location.
   - Save the resolved **full absolute path** into the config file under this project's `owner/repo` key, preserving all other entries already in the file.

4. **Determine this project's current version.** Use the same version-detection logic as the `--version` flag: read the `version` field from `.claude-plugin/plugin.json` / `.codex-plugin/plugin.json` / `gemini-extension.json` (whichever exists), falling back to the topmost version heading in `CHANGELOG.md` if none of those manifests exist.

5. **Update (or create) the marketplace entry.** Read the target `marketplace.json`. Find the `plugins[]` entry whose `source` repo matches this project's `owner/repo`.
   - **If found:** take its existing `description`, and update the leading version prefix. If it already starts with a version pattern (`^v?\d+\.\d+\.\d+\s+—\s+`), replace just that prefix; otherwise prepend `<version> — ` to the existing text. Leave the rest of the description, and every other field, untouched.
   - **If not found:** create a new entry. Pull the `name` and a base description from this project's own manifest (or its `SKILL.md` frontmatter `description` if no manifest exists), prefix the description with `<version> — `, and set the `source` field to reference this project's public repo, matching the field shape already used by sibling entries in that `marketplace.json` (e.g. `{"source": "github", "repo": "owner/repo"}` or whatever convention that file already follows — don't invent a new shape). Ask the user for a `category` if the marketplace's other entries use one and it isn't obvious from context.
   - Write the updated `marketplace.json` back, preserving formatting and the ordering/content of every other entry.

6. **Update the marketplace project's README.md.** Find `README.md` at the root of the marketplace repo (the directory containing `marketplace.json`, or its parent if `marketplace.json` lives in a `.claude-plugin/` subfolder). Find this project's existing entry in the README (matched by plugin name) and update its description line to match the new version-prefixed description; if no entry exists yet, add one following the same structure/heading level as the README's other listed entries.
   - If this project has a `help.md` and/or `CHANGELOG.md` at its root, append a line at the end of that project's README entry linking directly to them in the project's public GitHub repo, e.g.:
     `[Help](https://github.com/<owner>/<repo>/blob/<default-branch>/help.md) · [Changelog](https://github.com/<owner>/<repo>/blob/<default-branch>/CHANGELOG.md)`
     (omit whichever of the two doesn't exist).

7. **Commit and push.** Stage `marketplace.json` and `README.md` in the marketplace repo, commit with a message like "Sync `<plugin-name>` to `<version>` in marketplace listing", and push using that marketplace repo's own push method (plain `git push`, or the `gh-push` helper / manual GraphQL fallback if its remote is under an account known to need it).

8. **Report** the marketplace path used (and whether it was newly saved to config or reused), whether the entry was created or updated, the version applied, whether the README was updated, and which of help.md/CHANGELOG.md were linked.

## Workflow

Run each step in order. Report what was done or skipped at each step.

### Step 1: Verify Git Repository

Confirm the current directory is a git repository. Identify the default branch name (usually `main` or `master`).

If not a git repo, stop and tell the user: "This directory is not a git repository. Run `git init` first, or navigate to a git project."

### Step 2: Add MIT License

Check if a `LICENSE` or `LICENSE.md` or `LICENSE.txt` file exists at the project root.

**If missing:** Create a `LICENSE` file with the full MIT License text. Use the current year and derive the copyright holder name from `git config user.name`. Stage and commit the file with the message "Add MIT license".

**If present:** Report "LICENSE already exists, skipping." Do not modify it.

### Step 3: Check README.md

Check if `README.md` exists and has more than just a title line.

**If missing or stub-only:** Warn the user: "No README.md found (or it's just a title). Consider running /readme to generate one before releasing." Do not generate a README — that is the `/readme` skill's job.

**If present:** Report "README.md found." Continue.

### Step 4: Check Help Mechanism (Skills/Plugins Only)

Determine whether this project is itself an agent skill or plugin: look for a `SKILL.md` file (at the project root or in a `skills/*/` subdirectory) or a `.codex-plugin/plugin.json` manifest.

**If this project is not a skill/plugin:** skip this step entirely — not applicable, don't mention it in the summary.

**If it is a skill/plugin:** check whether it already has a working help mechanism — a `--help` flag (skills) or `:help` command (plugins) documented in the core manifest file(s), backed by a `help.md` file alongside each one.

- **If present for every manifest copy:** report "Help mechanism found, skipping."
- **If missing or incomplete:** warn the user: "No `--help`/`:help` mechanism found for this skill/plugin. Consider running `/make-readme` to add one." Do not create it yourself — creating the mechanism is `/make-readme`'s job, not this skill's.

### Step 5: Check/Add `--version` Support (Skills/Plugins Only)

**If this project is not a skill/plugin** (per Step 4's detection): skip this step entirely — not applicable, don't mention it in the summary.

**If it is a skill/plugin:** check whether it already has a `--version` flag (skills) or `:version` command (plugins) documented in the core manifest file(s) (and any `skills/<name>/` copies used for cross-platform packaging).

- **If present for every manifest copy:** report "Version flag found, skipping."
- **If missing or incomplete:** add it now — unlike the help mechanism and CHANGELOG (which are `/make-readme`'s job), this skill adds `--version`/`:version` support itself, since it's directly tied to the version-management this skill already owns.

**For a skill** (has `SKILL.md`): add a `## Flags` entry (or extend the existing one) to every `SKILL.md` copy — root, `skills/<name>/`, `antigravity/` if present — with this exact procedure:

```
### `--version`

If the user invokes this skill with a `--version` flag (e.g. `/<skill-name> --version`), do not run the workflow. Instead:

1. Read the installed version from this skill's own manifest: `.claude-plugin/plugin.json` if present, else `.codex-plugin/plugin.json`, else `gemini-extension.json` — whichever exists for this platform install. If none exist (a bare Claude Code skill with only SKILL.md), read the topmost version heading in `CHANGELOG.md` instead.
2. Print: `<skill-name> v<installed-version>`
3. Best-effort update check — determine this skill's GitHub source repo:
   a. If `.git` exists here and `git remote get-url origin` resolves to a `github.com` URL, use that `owner/repo`.
   b. Otherwise, search this skill's own `README.md` for the first `https://github.com/<owner>/<repo>` URL and use that.
   c. If neither yields a repo, or the `gh` CLI isn't installed/authenticated: stop here. Print nothing further — no status line, no error.
4. If a repo was found: run `gh api repos/<owner>/<repo>/releases/latest -q .tag_name` (strip a leading `v`). Compare to the installed version:
   - Equal → append: `Status: up to date`
   - Installed is older → append: `Status: newer version available (v<latest>). To update: if you installed this via a Claude Code marketplace, run /plugin marketplace update <marketplace-name> then reinstall; otherwise, git pull in your install directory if it's a git checkout, or re-copy from https://github.com/<owner>/<repo> per this README's Installation section.`
   - Installed is newer → append: `Status: ahead of latest release (development checkout)`
   - If the API call fails for any reason (network, auth, rate limit, malformed tag): print nothing further — no status line, no error shown to the user.
5. Stop — do not proceed to run the skill's actual workflow.
```

**For a plugin** (commands-based, no `SKILL.md`): add a `commands/version.md` file (and its mirror under any build/packaging directory, e.g. `plugin/commands/`) implementing the same 5-step procedure, invoked as `/<plugin-name>:version`.

Present the added flag/command to the user before writing, mirroring how the help mechanism is added.

### Step 6: Check CHANGELOG.md

Check if `CHANGELOG.md` exists.

**If missing:** Warn the user: "No CHANGELOG.md found. Consider running /make-readme to add one." Do not create it yourself — creating the file is `/make-readme`'s job, not this skill's.

**If present:** Report "CHANGELOG.md found." Continue.

### Step 7: Check .gitignore

Check if `.gitignore` exists.

**If missing:** Warn the user: "No .gitignore found. Consider adding one to prevent committing build artifacts, secrets, or OS files."

**If present:** Report ".gitignore found." Continue.

### Step 8: Ensure GitHub Remote

Check if a remote named `origin` exists and points to a GitHub URL.

**If no remote exists:**
1. Derive the repo name from the current directory name
2. Ask the user: "No GitHub remote found. Create a public repo named `<repo-name>` on GitHub?"
3. If yes, run: `gh repo create <repo-name> --public --source=. --push`
4. Report the URL of the created repo

**If remote exists:** Report the remote URL. Ensure local branch is pushed and up to date. Push if behind.

### Step 9: Apply Branch Protection

Apply branch protection rules to the default branch using the GitHub API:

```
Required pull request reviews:
  - Required approving review count: 1
  - Dismiss stale reviews: true
Enforce admins: false
Force pushes: blocked
Branch deletion: blocked
```

Run this API call:

```bash
gh api repos/<owner>/<repo>/branches/<default-branch>/protection \
  --method PUT \
  --input - <<'EOF'
{
  "required_pull_request_reviews": {
    "required_approving_review_count": 1,
    "dismiss_stale_reviews": true
  },
  "enforce_admins": false,
  "required_status_checks": null,
  "restrictions": null
}
EOF
```

Report the settings applied:
- PRs required with 1 approving review
- Stale reviews dismissed on new pushes
- Force pushes blocked
- Branch deletion blocked
- Admin bypass enabled (repo owner can still push directly)

### Step 10: Bump Manifest Versions, Finalize CHANGELOG, and Create GitHub Release

Ask the user for a version tag. Suggest `v1.0.0` if this is the first release (no existing tags), or suggest the next patch/minor/major version based on the latest existing tag.

**If this project is itself a skill or plugin** (per Step 4's detection): before tagging, find every manifest file that declares a `"version"` field — `.claude-plugin/plugin.json`, `.codex-plugin/plugin.json`, `gemini-extension.json` — including any copies under `skills/<name>/` used for cross-platform packaging. Update each one's `version` field to match the new tag with the leading `v` stripped (e.g. tag `v1.2.0` → `"version": "1.2.0"`). This is what keeps a plugin's declared version from drifting out of sync with its actual release, a real bug this step exists to prevent.

**If this project is not a skill/plugin:** skip the manifest bump — not applicable, don't mention it in the summary.

**If `CHANGELOG.md` exists** (per Step 6): before finalizing, check whether `## [Unreleased]` has any content under it (bullets, or non-empty `### Added`/`### Changed`/etc. subsections).

- **If `Unreleased` is empty:** stop and ask the user: "CHANGELOG.md's Unreleased section is empty — this release would ship with no changelog entry. Add an entry now, or proceed anyway?" Wait for their answer before continuing. Do not write a placeholder entry yourself and do not silently proceed — this is a deliberate gate, not a step to route around.
- **If `Unreleased` has content, or the user confirmed proceeding anyway despite it being empty:** finalize it for this release — this is a mechanical rename, not content generation. Rename the `## [Unreleased]` heading to `## [<version>] - <YYYY-MM-DD>` (version without the leading `v`, date = today), then insert a fresh, empty `## [Unreleased]` heading above it. Do not invent, summarize, or generate changelog entries yourself — whatever prose already sits under `Unreleased` becomes that version's entry verbatim, unedited.

**If `CHANGELOG.md` is missing:** skip this — not applicable.

Stage and commit the manifest bump and CHANGELOG finalization together with the message "Bump version to <version>" before creating the tag, so the tagged commit already reflects the version being released.

Create the release:

```bash
gh release create <tag> --generate-notes --latest
```

Report the release URL, which manifest files were bumped (if any), and whether CHANGELOG.md was finalized.

### Step 11: Sync Marketplace Listing (Optional)

**If this project is not itself a skill or plugin** (per Step 4's detection): skip this step entirely — not applicable, don't mention it in the summary.

**If it is a skill or plugin:** check for an existing entry in this skill's own `.git-release/marketplace-config.json` (see the `--marketplace` flag above) for this project's `owner/repo`.

- **If a saved marketplace path exists:** ask the user: "Sync this release to `<saved-marketplace-path>` (as configured)?" If yes, run the full `--marketplace` procedure (steps 4-8 above — version detection, entry update/create, README sync, commit and push), reusing that saved path without re-asking for a location. If no: skip, not applicable.
- **If no saved marketplace path exists:** ask the user: "Sync this release to a marketplace listing?" If yes, run the full `--marketplace` procedure from the top (steps 1-8 above), which will ask for and save a marketplace location. If no: skip, not applicable.

Report which marketplace (if any) was updated, and to what version.

### Step 12: Summary

Present a summary table of everything that was done. Include the Help mechanism, Version flag, and Manifest version(s) rows only if Step 4 / Step 10's skill-or-plugin check applied (i.e. this project is a skill/plugin); include the CHANGELOG.md row only if Step 6 found the file present; include the Marketplace listing row only if Step 11 found a match:

```
## Release Summary

| Step | Status |
|------|--------|
| Git repo | Confirmed (branch: main) |
| LICENSE | Created (MIT) / Already existed |
| README.md | Found / Warning: missing |
| Help mechanism | Found / Warning: missing (run /make-readme) |
| Version flag | Found / Added |
| CHANGELOG.md | Finalized: [Unreleased] → [<version>] / Empty, confirmed by user / Warning: missing (run /make-readme) |
| .gitignore | Found / Warning: missing |
| GitHub remote | Created: <url> / Existing: <url> |
| Branch protection | Applied (PRs required, force push blocked) |
| Manifest version(s) | Bumped to <version> in <N> file(s) |
| Marketplace listing | Updated <marketplace>/marketplace.json to <version> / Not found |
| Release | Created: <tag> — <url> |
```

## Error Handling

- If `gh` CLI is not installed or not authenticated, stop and tell the user to install/authenticate it
- If branch protection fails (e.g., free plan limitations), warn but continue with the release
- If any step fails, report the failure and continue with remaining steps where possible
