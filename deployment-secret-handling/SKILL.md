---
name: deployment-secret-handling
description: 'Handle Coolify, GitHub, database, JWT, and AWS deployment secrets safely. Use when generating keys, configuring environment variables, debugging authentication, reviewing logs, rotating tokens, or cleaning up deployment artifacts.'
argument-hint: 'Describe the secret type and lifecycle operation without providing the secret value.'
---

# Deployment Secret Handling

## Never Expose

Treat these as secret material:

- Coolify API tokens
- SSH private keys and unredacted key-management forms
- `DATABASE_URL`, database passwords, and connection strings
- JWT `SECRET_KEY`
- AWS access keys, session tokens, and secret access keys
- Authorization headers, cookies, and reset/bootstrap credentials

Do not place them in Git, skills, screenshots, browser dumps, shell transcripts, deployment logs, URLs, frontend `VITE_*` variables, or assistant messages.

## Storage Rules

- Store runtime secrets in Coolify encrypted variables or the platform's secret store.
- Keep deploy keys outside the repository in a user-controlled directory with restrictive permissions.
- Commit only templates with placeholder values, such as `.env.example`.
- Keep frontend build variables limited to public configuration, typically the HTTPS API base URL.
- Use separate credentials for local, staging, and production environments.

## Rotation and Incident Response

If a token or private key may have been exposed:

1. Revoke or replace it at the issuing system.
2. Replace the corresponding Coolify variable/key.
3. Update dependent services and redeploy.
4. Review access and deployment history for unauthorized use.
5. Remove the value from untracked local artifacts and logs where practical.
6. Search Git history before declaring the repository clean; ignoring a file does not remove an already committed secret.

Never assume a secret is harmless because there is no evidence it was used. Report exposure and recommend rotation without claiming active compromise.

## Logging and Diagnostics

Redact values before sharing errors. Prefer variable names, resource UUIDs, timestamps, status codes, and commit IDs. Do not use broad log or DOM extraction on pages that can contain environment values. Health endpoints must return status information only and must not reveal credentials, connection strings, tokens, or detailed internal exceptions.
