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

### Step 5: Check CHANGELOG.md

Check if `CHANGELOG.md` exists.

**If missing:** Warn the user: "No CHANGELOG.md found. Consider running /make-readme to add one." Do not create it yourself — creating the file is `/make-readme`'s job, not this skill's.

**If present:** Report "CHANGELOG.md found." Continue.

### Step 6: Check .gitignore

Check if `.gitignore` exists.

**If missing:** Warn the user: "No .gitignore found. Consider adding one to prevent committing build artifacts, secrets, or OS files."

**If present:** Report ".gitignore found." Continue.

### Step 7: Ensure GitHub Remote

Check if a remote named `origin` exists and points to a GitHub URL.

**If no remote exists:**
1. Derive the repo name from the current directory name
2. Ask the user: "No GitHub remote found. Create a public repo named `<repo-name>` on GitHub?"
3. If yes, run: `gh repo create <repo-name> --public --source=. --push`
4. Report the URL of the created repo

**If remote exists:** Report the remote URL. Ensure local branch is pushed and up to date. Push if behind.

### Step 8: Apply Branch Protection

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

### Step 9: Bump Manifest Versions, Finalize CHANGELOG, and Create GitHub Release

Ask the user for a version tag. Suggest `v1.0.0` if this is the first release (no existing tags), or suggest the next patch/minor/major version based on the latest existing tag.

**If this project is itself a skill or plugin** (per Step 4's detection): before tagging, find every manifest file that declares a `"version"` field — `.claude-plugin/plugin.json`, `.codex-plugin/plugin.json`, `gemini-extension.json` — including any copies under `skills/<name>/` used for cross-platform packaging. Update each one's `version` field to match the new tag with the leading `v` stripped (e.g. tag `v1.2.0` → `"version": "1.2.0"`). This is what keeps a plugin's declared version from drifting out of sync with its actual release, a real bug this step exists to prevent.

**If this project is not a skill/plugin:** skip the manifest bump — not applicable, don't mention it in the summary.

**If `CHANGELOG.md` exists** (per Step 5): finalize it for this release — this is a mechanical rename, not content generation. Rename the `## [Unreleased]` heading to `## [<version>] - <YYYY-MM-DD>` (version without the leading `v`, date = today), then insert a fresh, empty `## [Unreleased]` heading above it. Do not invent, summarize, or generate changelog entries yourself — whatever prose already sits under `Unreleased` (written during development) becomes that version's entry verbatim, unedited. If `Unreleased` is empty or missing entirely, still perform the rename/date-stamp so the file stays well-formed, but do not fabricate content to fill it.

**If `CHANGELOG.md` is missing:** skip this — not applicable.

Stage and commit the manifest bump and CHANGELOG finalization together with the message "Bump version to <version>" before creating the tag, so the tagged commit already reflects the version being released.

Create the release:

```bash
gh release create <tag> --generate-notes --latest
```

Report the release URL, which manifest files were bumped (if any), and whether CHANGELOG.md was finalized.

### Step 10: Summary

Present a summary table of everything that was done. Include the Help mechanism and Manifest version(s) rows only if Step 4 / Step 9's skill-or-plugin check applied (i.e. this project is a skill/plugin); include the CHANGELOG.md row only if Step 5 found the file present:

```
## Release Summary

| Step | Status |
|------|--------|
| Git repo | Confirmed (branch: main) |
| LICENSE | Created (MIT) / Already existed |
| README.md | Found / Warning: missing |
| Help mechanism | Found / Warning: missing (run /make-readme) |
| CHANGELOG.md | Finalized: [Unreleased] → [<version>] / Warning: missing (run /make-readme) |
| .gitignore | Found / Warning: missing |
| GitHub remote | Created: <url> / Existing: <url> |
| Branch protection | Applied (PRs required, force push blocked) |
| Manifest version(s) | Bumped to <version> in <N> file(s) |
| Release | Created: <tag> — <url> |
```

## Error Handling

- If `gh` CLI is not installed or not authenticated, stop and tell the user to install/authenticate it
- If branch protection fails (e.g., free plan limitations), warn but continue with the release
- If any step fails, report the failure and continue with remaining steps where possible
