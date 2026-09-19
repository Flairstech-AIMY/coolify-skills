# coolify-skills

A collection of [Agent Skills](https://cli.github.com/manual/gh_skill) for Coolify deployment, diagnostics, security verification, and related AWS IAM / Git workflows. Each folder under [`skills/`](skills/) is a skill package containing a `SKILL.md` with frontmatter (`name`, `description`, `argument-hint`) plus the procedure an agent should follow.

## Installing skills with the GitHub CLI

These skills can be installed directly from this repository using the `gh skill` command ([manual](https://cli.github.com/manual/gh_skill)), which is currently in preview.

```bash
# Install the GitHub CLI if you don't already have it
# https://cli.github.com/

# Search for skills in this repo (or across GitHub)
gh skill search coolify

# Preview a skill before installing it
gh skill preview Flairstech-AIMY/coolify-skills coolify-deployment

# Install a specific skill from this repository
gh skill install Flairstech-AIMY/coolify-skills coolify-deployment

# List skills you've installed
gh skill list

# Update all installed skills
gh skill update --all
```

`gh skill` (alias `gh skills`) manages skills the same way `gh extension` manages CLI extensions: it clones/fetches the skill content from the referenced GitHub repository and makes it available to compatible agent tooling (e.g. VS Code, GitHub Copilot).

## Available skills

| Skill | Description |
| --- | --- |
| [aws-iam-discovery-debugging](skills/aws-iam-discovery-debugging/SKILL.md) | Debug AWS CloudFront or Route 53 discovery failures in this scanner. Use for AccessDenied, missing credentials, incomplete credentials, wrong account, wrong IAM principal, or region-specific discovery errors. |
| [aws-iam-discovery-fix](skills/aws-iam-discovery-fix/SKILL.md) | Fix AWS discovery authorization for this Coolify backend. Use after diagnosing IAM AccessDenied, missing credentials, wrong principal, incorrect trust policy placement, or missing AWS runtime variables. |
| [aws-iam-discovery-planning](skills/aws-iam-discovery-planning/SKILL.md) | Plan AWS permissions for this security scanner before deployment. Use when choosing an IAM user or EC2 role, creating least-privilege discovery access, or configuring CloudFront and Route 53 inventory. |
| [coolify-api-direct-rest](skills/coolify-api-direct-rest/SKILL.md) | Use the Coolify REST API directly for writable provisioning and lifecycle operations. |
| [coolify-api-provisioning](skills/coolify-api-provisioning/SKILL.md) | Provision and manage Coolify resources through the REST API, including writable project, PostgreSQL, private SSH application, variable, and deployment operations. |
| [coolify-api-reference-validation](skills/coolify-api-reference-validation/SKILL.md) | Validate Coolify REST API routes, payloads, and response semantics against the official API reference before making deployment changes. |
| [coolify-application-password-reset](skills/coolify-application-password-reset/SKILL.md) | Reset an application user's password in the production database hosted on Coolify. |
| [coolify-browser-operations](skills/coolify-browser-operations/SKILL.md) | Operate Coolify through a browser agent when API coverage is incomplete. |
| [coolify-database-access](skills/coolify-database-access/SKILL.md) | Safely access, expose, troubleshoot, and recover credentials for Coolify-managed PostgreSQL databases without exposing secrets or destroying data. |
| [coolify-deployment](skills/coolify-deployment/SKILL.md) | Deploy this FastAPI and Vite/React application to Coolify. |
| [coolify-deployment-diagnostics](skills/coolify-deployment-diagnostics/SKILL.md) | Diagnose Coolify deployment failures using deployment records, source checkout evidence, application status, proxy state, and Git credential checks. |
| [coolify-deployment-planning](skills/coolify-deployment-planning/SKILL.md) | Plan a Coolify deployment for this FastAPI and Vite/React application. |
| [coolify-deployment-verification](skills/coolify-deployment-verification/SKILL.md) | Verify a complete Coolify deployment of the FastAPI backend, Vite frontend, and PostgreSQL dependency. |
| [coolify-frontend-api-debugging](skills/coolify-frontend-api-debugging/SKILL.md) | Debug frontend-to-backend API failures in Coolify, especially 405, 404, CORS, unauthorized, stale-bundle, or login errors. |
| [coolify-frontend-api-fix](skills/coolify-frontend-api-fix/SKILL.md) | Fix and verify frontend-to-backend API routing failures in this Coolify deployment. |
| [coolify-mcp-operations](skills/coolify-mcp-operations/SKILL.md) | Use the Coolify MCP tools to inspect, configure, deploy, and verify team-owned resources without browser automation or secret exposure. |
| [coolify-postgres-connectivity-recovery](skills/coolify-postgres-connectivity-recovery/SKILL.md) | Diagnose and repair Coolify application startup failures caused by PostgreSQL DNS, network, or DATABASE_URL configuration problems. |
| [coolify-private-ssh-deployment](skills/coolify-private-ssh-deployment/SKILL.md) | Deploy private GitHub repositories to Coolify with an SSH deploy key. |
| [coolify-terminal-access](skills/coolify-terminal-access/SKILL.md) | Guide Coolify terminal access for a deployed application or database, including inspecting PostgreSQL roles safely and running bounded server-side commands without exposing secrets. |
| [deployment-secret-handling](skills/deployment-secret-handling/SKILL.md) | Handle Coolify, GitHub, database, JWT, and AWS deployment secrets safely. |
| [deployment-security-verification](skills/deployment-security-verification/SKILL.md) | Review and verify production deployment security for this scanner before or after Coolify deployment. |
| [git-github-workflow](skills/git-github-workflow/SKILL.md) | Manage Git and GitHub repository workflows safely. |

## Repository structure

Skills live under `skills/`, one folder per skill, matching the `skills/*/SKILL.md` convention used by `gh skill` and the [Agent Skills specification](https://agentskills.io/specification):

```
skills/
  <skill-name>/
    SKILL.md   # frontmatter (name, description, argument-hint) + procedure
```

`system-prompt.md` at the repository root is the operator-level system prompt that ties these skills together for Coolify deployment work.
