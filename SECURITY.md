# Security Policy

## Supported Versions

Only the latest released version of `git-release` is supported. Security fixes are released as a new patch version — please update before reporting an issue to confirm it still reproduces.

## Scope

This repository ships a Claude Code skill (`SKILL.md`, `help.md`, and the `skills/git-release/` packaging copy) that drives `git`/`gh` commands on your behalf. There is no running service, server, or stored user data — the main risks worth reporting are things like a workflow step that could be tricked into running an unintended destructive `git`/`gh` command, leaking credentials, or bypassing a safety check (e.g. the branch-protection or dry-run gates).

## Reporting a Vulnerability

- **Preferred:** use GitHub's [private vulnerability reporting](https://github.com/keithmackay/git-release/security/advisories/new) on this repository, so the report isn't public before a fix ships.
- **Alternative:** open an issue at [github.com/keithmackay/git-release/issues](https://github.com/keithmackay/git-release/issues) for anything that isn't security-sensitive, or if private reporting isn't available to you.

Please include the skill version (`/git-release --version`), the step/flag involved, and a minimal repro.

## Response Timeline

Reports are typically acknowledged within a few days. There's no formal SLA — this is a single-maintainer project — but security reports are prioritized over other issues.
