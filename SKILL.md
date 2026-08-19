---
name: gitrelease
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

### Step 4: Check .gitignore

Check if `.gitignore` exists.

**If missing:** Warn the user: "No .gitignore found. Consider adding one to prevent committing build artifacts, secrets, or OS files."

**If present:** Report ".gitignore found." Continue.

### Step 5: Ensure GitHub Remote

Check if a remote named `origin` exists and points to a GitHub URL.

**If no remote exists:**
1. Derive the repo name from the current directory name
2. Ask the user: "No GitHub remote found. Create a public repo named `<repo-name>` on GitHub?"
3. If yes, run: `gh repo create <repo-name> --public --source=. --push`
4. Report the URL of the created repo

**If remote exists:** Report the remote URL. Ensure local branch is pushed and up to date. Push if behind.

### Step 6: Apply Branch Protection

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

### Step 7: Create GitHub Release

Ask the user for a version tag. Suggest `v1.0.0` if this is the first release (no existing tags), or suggest the next patch/minor/major version based on the latest existing tag.

Create the release:

```bash
gh release create <tag> --generate-notes --latest
```

Report the release URL.

### Step 8: Summary

Present a summary table of everything that was done:

```
## Release Summary

| Step | Status |
|------|--------|
| Git repo | Confirmed (branch: main) |
| LICENSE | Created (MIT) / Already existed |
| README.md | Found / Warning: missing |
| .gitignore | Found / Warning: missing |
| GitHub remote | Created: <url> / Existing: <url> |
| Branch protection | Applied (PRs required, force push blocked) |
| Release | Created: <tag> — <url> |
```

## Error Handling

- If `gh` CLI is not installed or not authenticated, stop and tell the user to install/authenticate it
- If branch protection fails (e.g., free plan limitations), warn but continue with the release
- If any step fails, report the failure and continue with remaining steps where possible
