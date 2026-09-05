---
name: git-github-workflow
description: 'Manage Git and GitHub repository workflows safely. Use when creating or connecting repositories, deciding whether to push to main, creating branches, preparing commits, opening pull requests, resolving divergence, or verifying published changes.'
argument-hint: 'Describe the intended change, repository state, collaboration context, and whether direct main authorization exists.'
---

# Git and GitHub Workflow

## Purpose

Use this skill for repository setup and delivery decisions. Preserve existing history and uncommitted user work. Never expose secrets, force-push, or delete remote history without explicit user authorization.

## Initial Inspection

Before changing Git state, inspect:

```powershell
git status --short --branch
git branch --show-current
git remote -v
git log --oneline --decorate -5
git diff --stat
```

Also check for files that must not be committed:

- `.env` files and secret configuration
- AWS keys, tokens, passwords, private keys, and certificates
- Virtual environments, `node_modules`, build output, caches, and local databases
- Generated files unless the repository intentionally tracks them

Confirm `.gitignore` covers excluded files. If a likely secret is already tracked, stop and address it before pushing; do not assume that renaming or ignoring it removes it from history.

## Repository Creation or Connection

1. If a valid remote already exists, use it. Do not create a duplicate repository.
2. If no remote exists, ask for or confirm the GitHub organization, repository name, visibility, and whether an empty repository should be created.
3. Prefer the repository's existing default branch and remote URL.
4. If local `.git` metadata is missing but the project has a known remote, fetch the remote before committing. Preserve remote history and avoid force-pushing.
5. If local and remote histories are unrelated, inspect both trees and merge deliberately with `--allow-unrelated-histories` only when necessary. Resolve conflicts manually and verify the result.
6. Never create a public repository containing secrets or sensitive operational data.

## Branch Decision

Use a feature branch by default when any of the following is true:

- The repository is shared or has branch protection.
- The change is more than a small, well-understood fix.
- The change affects production infrastructure, authentication, data migrations, or security controls.
- Tests are incomplete, deployment is not yet verified, or another contributor may be working concurrently.
- The user asks for review, collaboration, or a pull request.

Use a branch name that describes the work, such as `feat/coolify-deployment` or `fix/api-cors`.

Directly use `main` only when all of the following are true:

- The user explicitly authorizes pushing this change to `main` in the current request.
- The repository is a personal or first-time setup, or the user confirms review is unnecessary.
- The working tree and staged diff have been inspected.
- No secrets or unrelated changes are included.
- Relevant focused validation has passed.
- The push is a normal fast-forward, or the user explicitly approves a non-fast-forward recovery. Never force-push by default.

An earlier general instruction to move quickly is not authorization to bypass branch protection or review.

## Commit Workflow

1. Review the current diff and status before staging.
2. Stage only files belonging to the requested change:

```powershell
git add <specific-files>
git diff --cached --check
git diff --cached --stat
git diff --cached --name-only
```

3. Use a concise imperative commit message, for example `Prepare application for Coolify deployment`.
4. Run the narrowest useful tests before committing and record failures accurately.
5. After committing, verify `git status --short --branch` and `git log --oneline -3`.

Do not amend, reset, rebase, or discard existing commits unless the user explicitly requests it. Do not stage unrelated user changes.

## Push Workflow

Before pushing:

```powershell
git fetch origin
 git rev-list --left-right --count origin/<branch>...<branch>
```

If the remote is ahead, stop and inspect the divergence. Integrate remote work with a merge or carefully planned rebase, preserving both sides. Never overwrite remote commits with `git push --force` or `--force-with-lease` unless the user explicitly approves the exact operation and its consequences.

Push with the least privilege needed:

```powershell
git push -u origin <branch>
```

After a successful push, verify:

```powershell
git status --short --branch
git log --oneline --decorate -3
git diff --check origin/<branch>~1..origin/<branch>
```

Report the repository, branch, commit, validation result, and any blocked checks.

## Pull Requests

Create a pull request when the work should be reviewed, when `main` is protected, or when the change has meaningful production, security, schema, or operational risk.

A good PR should include:

- What changed and why
- Deployment or migration implications
- Tests and checks that passed
- Known limitations or follow-up work
- Configuration and secret requirements without revealing secret values

Keep the PR focused. Do not mix unrelated refactors or generated artifacts. Link the PR to the feature branch and target the repository's default branch unless the user specifies another target.

## Conflict and Failure Handling

- Authentication failure: report that GitHub authentication is required; do not request or transmit passwords, tokens, or private keys through chat.
- Non-fast-forward push: fetch, inspect, and integrate; do not force-push automatically.
- Merge conflict: read each conflict, preserve intentional changes from both sides, remove all conflict markers, run focused validation, then commit the merge.
- Failed tests or builds: do not push a knowingly broken change unless the user explicitly accepts it; report the exact blocked check.
- Existing user changes: preserve them, stage selectively, and ask only when they make the requested operation unsafe or ambiguous.
