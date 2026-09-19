---
name: coolify-browser-operations
description: 'Operate Coolify through a browser agent when API coverage is incomplete. Use for login, private-key selection, creating Private Repository with Deploy Key applications, editing configuration forms, and validating visible resource state.'
argument-hint: 'Provide the authenticated Coolify URL, target project/environment, desired resource type, and non-secret fields to configure.'
---

# Coolify Browser Operations

## Scope

Use browser automation for UI-only operations such as selecting a private deploy key, completing a resource wizard, editing a generated application name, configuring health checks, or confirming visible status. Prefer API reads for repeatable IDs and status when available.

## Safe Session Procedure

1. Open the Coolify URL and confirm the session is authenticated.
2. Navigate to the exact project and environment; do not rely on a stale tab or generated name.
3. Capture only non-secret labels and status. Do not inspect, copy, or print private-key fields, API tokens, environment values, or full deployment logs.
4. Use stable visible labels and scoped selectors. Re-read the page after navigation because Livewire-style forms may update asynchronously.
5. Before submitting, review repository URL, branch, key name, base directory, build pack, and exposed port.
6. After submitting, wait for the resource UUID or configuration page and verify the saved fields.

## Private Repository Wizard

Select `Private Git Repository (with Deploy Key)`, choose the intended named key, enter the SSH repository URL and branch, then select Dockerfile. Configure the application root and port according to the service:

- Backend: `/backend`, port `8000`, health path `/health`.
- Frontend: `/frontend`, port `80`, health path `/health`.

Do not paste a private key into a repository form. The wizard should reference the Coolify key by name.

## Authentication and Failure Handling

If redirected to login, stop and require interactive user authentication; never request credentials through chat or attempt to bypass MFA. If a button is disabled, inspect required fields and validation state rather than clicking repeatedly. If an application is created but remains `Exited`, switch to deployment diagnostics and inspect the deployment record.

## Secret Hygiene

Avoid screenshots or DOM extraction that includes secrets. Do not use broad page-text dumps on pages containing environment variables or private keys. When recording results, keep only resource names, UUIDs, status, and redacted configuration facts.
