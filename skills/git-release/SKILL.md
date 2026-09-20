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

### `--dry-run`

If the user invokes this skill with a `--dry-run` flag (e.g. `/git-release --dry-run`), run through the full Workflow below, but treat every step that would create, modify, commit, push, tag, or call a mutating GitHub API as report-only: state what it *would* do, then move on without doing it. Steps that are already pure checks (1, 3, 4, 6, 7) run exactly as normal — there's nothing to hold back. Combine with `--marketplace` (i.e. `--marketplace --dry-run`) to preview that flag's sync instead of the full release workflow, under the same rule.

Concretely, in dry-run mode:

- **Step 2 (LICENSE):** if missing, report "Would create LICENSE (MIT)" instead of creating/committing it.
- **Step 5 (`--version` support):** if missing, report which files would receive the flag/command (e.g. "Would add `--version` to SKILL.md, skills/<name>/SKILL.md") instead of writing them.
- **Step 8 (GitHub remote):** if no remote, report "Would create public GitHub repo `<repo-name>`" instead of running `gh repo create`. If a remote exists and local is behind, report "Would push <N> commit(s) to origin" instead of pushing.
- **Step 9 (repo description):** if missing, empty, or missing/mismatching the correct type prefix, report the proposed one-line description (with its type prefix) and "Would set repo description to: \"<proposed description>\"" instead of calling the API. If already set with the correct prefix, this step is a pure check and runs normally.
- **Step 10 (branch protection):** report "Would apply branch protection: PRs required (1 approval), stale reviews dismissed, force push blocked, branch deletion blocked" instead of calling the API.
- **Step 11 (version bump/CHANGELOG/release):** still ask the user for a version tag (this is informational, not mutating), then report "Would bump manifest version to `<version>` in `<N>` file(s)", "Would finalize CHANGELOG.md: [Unreleased] → [<version>] - <date>" (or "Unreleased is empty — would prompt before proceeding" if applicable), and "Would create tag `<version>` and GitHub release" — without touching any file, committing, tagging, or calling `gh release create`.
- **Step 12 / `--marketplace` procedure:** perform the read-only parts (resolving the repo, locating/asking for the marketplace config path, determining current version) normally, but report "Would update `<marketplace-name>`'s marketplace.json entry to `<version>`" / "Would create new entry for `<plugin-name>`", "Would update marketplace README.md entry", "Would update this project's own README.md Installation section" — without writing or pushing to either repo, and without saving a newly-given marketplace path into the config file (ask first: "Save this path for future runs?" — a `--dry-run` shouldn't silently persist state).

End with the same Step 12 summary table, but every row that describes a would-be action is prefixed `Would:` (e.g. `Would: Create (MIT)`), and the table itself is headed with **"## Dry Run — no changes were made"** instead of "## Release Summary".

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

5. **Determine the base description.** If this project has a GitHub repo description set (per Step 9 — fetch it live with `gh repo view <owner>/<repo> --json description -q .description` if not already known this run), use it verbatim as the base description. Otherwise fall back to this project's own manifest description (or its `SKILL.md` frontmatter `description` field if no manifest exists).

6. **Update (or create) the marketplace entry.** Read the target `marketplace.json`. Find the `plugins[]` entry whose `source` repo matches this project's `owner/repo`.
   - **If found:** replace its `description` with `<version> — <base description>` (from step 5), so the marketplace listing always mirrors the GitHub repo's own one-line description rather than drifting from it. Leave every other field untouched.
   - **If not found:** create a new entry. Use the `name` and the base description from step 5, prefix the description with `<version> — `, and set the `source` field to reference this project's public repo, matching the field shape already used by sibling entries in that `marketplace.json` (e.g. `{"source": "github", "repo": "owner/repo"}` or whatever convention that file already follows — don't invent a new shape). Ask the user for a `category` if the marketplace's other entries use one and it isn't obvious from context.
   - Write the updated `marketplace.json` back, preserving formatting and the ordering/content of every other entry.

7. **Update the marketplace project's README.md.** Find `README.md` at the root of the marketplace repo (the directory containing `marketplace.json`, or its parent if `marketplace.json` lives in a `.claude-plugin/` subfolder). Find this project's existing entry in the README (matched by plugin name) and update its description line to match the new version-prefixed description; if no entry exists yet, add one following the same structure/heading level as the README's other listed entries.
   - If this project has a `help.md` and/or `CHANGELOG.md` at its root, append a line at the end of that project's README entry linking directly to them in the project's public GitHub repo, e.g.:
     `[Help](https://github.com/<owner>/<repo>/blob/<default-branch>/help.md) · [Changelog](https://github.com/<owner>/<repo>/blob/<default-branch>/CHANGELOG.md)`
     (omit whichever of the two doesn't exist).

8. **Commit and push the marketplace repo.** Stage `marketplace.json` and `README.md` there, commit with a message like "Sync `<plugin-name>` to `<version>` in marketplace listing", and push using that marketplace repo's own push method (plain `git push`, or the `gh-push` helper / manual GraphQL fallback if its remote is under an account known to need it).

9. **Update this project's own README.md Installation section.** Determine the marketplace's own name (the top-level `"name"` field in its `marketplace.json`) and its public `owner/repo` (from `git remote get-url origin` inside the marketplace repo). At the very top of this project's `## Installation` section — before any existing per-platform subsections — add or update a subsection pointing at marketplace installation, e.g.:

   ```markdown
   ### From the <marketplace-name> marketplace (recommended)

   ```
   /plugin marketplace add <marketplace-owner>/<marketplace-repo>
   /plugin install <plugin-name>@<marketplace-name>
   ```
   ```

   If this subsection already exists (from a prior `--marketplace` run), just update the marketplace name/repo/plugin name in place rather than duplicating it. Stage this README.md change alongside this project's own commits (commit it now with a message like "Document `<marketplace-name>` marketplace installation in README" — this is a change to the project being released, not the marketplace repo, so it does not get pushed as part of step 8).

10. **Report** the marketplace path used (and whether it was newly saved to config or reused), whether the entry was created or updated, the version applied, whether the marketplace's README was updated, which of help.md/CHANGELOG.md were linked, and whether this project's own README.md installation section was updated.

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

### Step 9: Ensure Repo Description

GitHub's repo `description` field is what surfaces this project's one-line summary in places like dev.to's GitHub Connections, `gh repo list`, and the repo header — it isn't derived from the README or `SKILL.md` frontmatter, so it has to be set explicitly.

**Determine the type prefix.** Every proposed/checked description leads with what the project *is*, before the actual description — this is what lets someone scanning a repo list or a marketplace tell at a glance whether something is a skill, a plugin, or something else entirely:

- **Skill, single platform:** detect which agent platform(s) this project is built for by checking for a `SKILL.md` (root or `skills/*/`, and/or an `antigravity/` copy) → Claude Code; a `.codex-plugin/plugin.json` manifest → Codex; a `gemini-extension.json` or `GEMINI.md` → Gemini. If exactly one platform is present, the prefix is `"<Platform> skill: "` (e.g. `"Claude skill: "`).
- **Skill, multiple platforms:** if more than one platform is present, join them with `/` in this fixed order — Claude, Codex, Gemini — e.g. `"Claude/Codex/Gemini skill: "` or `"Claude/Gemini skill: "`. Only include a platform actually present; never list one that isn't.
- **Plugin:** a commands-based project (a `.claude-plugin/plugin.json` manifest, `commands/*.md` files, no `SKILL.md`) is a Claude Code plugin — prefix `"Claude Code plugin: "`.
- **Neither:** for any other kind of project (a website, a library, a CLI tool, documentation, an application, etc.), infer the single most fitting noun from its structure/README and use `"<Noun>: "` (e.g. `"Website: "`, `"CLI tool: "`, `"Documentation: "`). Don't force skill/plugin phrasing onto a project that isn't one.

Run `gh repo view <owner>/<repo> --json description -q .description` to check the current value.

**If already set and it already starts with the correct type prefix determined above:** Report "Repo description already set: \"<description>\", skipping." Do not modify it.

**If missing, empty, or set but missing/mismatching the correct type prefix:** Propose a one-line description: `"<type prefix><summary>"` where the summary (roughly 10-15 words, no trailing period) is derived from — in order of preference — the existing repo description (stripped of any stale prefix) if one was set, then this project's `SKILL.md`/manifest `description` frontmatter field, then the README's opening summary/tagline, then a plain-language summary of what the project does inferred from its code/structure. Show the full proposed description (prefix + summary) to the user and ask them to accept it as-is or supply their own replacement. Once confirmed, set it:

```bash
gh repo edit <owner>/<repo> --description "<description>"
```

Report the description that was applied.

### Step 10: Apply Branch Protection

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

### Step 11: Bump Manifest Versions, Finalize CHANGELOG, and Create GitHub Release

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

### Step 12: Sync Marketplace Listing (Optional)

**If this project is not itself a skill or plugin** (per Step 4's detection): skip this step entirely — not applicable, don't mention it in the summary.

**If it is a skill or plugin:** check for an existing entry in this skill's own `.git-release/marketplace-config.json` (see the `--marketplace` flag above) for this project's `owner/repo`.

- **If a saved marketplace path exists:** ask the user: "Sync this release to `<saved-marketplace-path>` (as configured)?" If yes, run the full `--marketplace` procedure (steps 4-10 above — version detection, base-description resolution, entry update/create, marketplace README sync, commit and push, and this project's own README install-instructions update), reusing that saved path without re-asking for a location. If no: skip, not applicable.
- **If no saved marketplace path exists:** ask the user: "Sync this release to a marketplace listing?" If yes, run the full `--marketplace` procedure from the top (steps 1-10 above), which will ask for and save a marketplace location. If no: skip, not applicable.

Report which marketplace (if any) was updated, and to what version.

### Step 13: Summary

Present a summary table of everything that was done. Include the Help mechanism, Version flag, and Manifest version(s) rows only if Step 4 / Step 11's skill-or-plugin check applied (i.e. this project is a skill/plugin); include the CHANGELOG.md row only if Step 6 found the file present; include the Marketplace listing row only if Step 12 found a match:

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
| Repo description | Set: "<description>" / Already set / Kept user-supplied |
| Branch protection | Applied (PRs required, force push blocked) |
| Manifest version(s) | Bumped to <version> in <N> file(s) |
| Marketplace listing | Updated <marketplace>/marketplace.json to <version> / Not found |
| Release | Created: <tag> — <url> |
```

## Error Handling

- If `gh` CLI is not installed or not authenticated, stop and tell the user to install/authenticate it
- If branch protection fails (e.g., free plan limitations), warn but continue with the release
- If any step fails, report the failure and continue with remaining steps where possible
