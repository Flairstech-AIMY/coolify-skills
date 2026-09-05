---
name: deployment-security-verification
description: 'Review and verify production deployment security for this scanner. Use before or after Coolify deployment to check secrets, JWT configuration, CORS, database exposure, AWS IAM access, backups, health endpoints, and cross-origin authentication.'
argument-hint: 'Provide the deployed frontend and API domains plus the environment and database status.'
---

# Deployment Security Verification

## When to Use

Use this skill before exposing the scanner publicly, after changing Coolify variables or domains, and after a redeploy.

## Pre-Deployment Checks

1. Confirm `SECRET_KEY` is a generated production secret and is not committed or logged.
2. Confirm `DATABASE_URL` uses the private Coolify PostgreSQL network and is not exposed to the frontend.
3. Confirm PostgreSQL is not publicly exposed unless there is a documented operational need.
4. Set `CORS_ORIGINS` to the exact HTTPS frontend origin. Do not use `*` with credentials.
5. Store AWS credentials only as Coolify secrets. Prefer a least-privilege IAM identity limited to discovery operations and rotate it regularly.
6. Confirm the frontend build contains only the public API URL. Never put database credentials, JWT signing keys, or AWS secrets in `VITE_*` variables.
7. Take a database backup before the first production deploy and before schema-affecting upgrades.

## Authentication Checks

Test the deployed API without a token and with an invalid or expired token. Protected endpoints must fail closed with `401` or `403`. Verify that authorization uses the verified JWT claims and does not trust client-supplied user, account, tenant, or role identifiers.

Create the first admin only through the intended bootstrap route, then verify that later user creation and admin configuration require authorization.

## Browser Checks

From the deployed frontend origin:

1. Log in through the UI.
2. Confirm API requests use the HTTPS API domain and an `Authorization: Bearer` header.
3. Reload a protected route and confirm the session behavior is intentional.
4. Confirm a `401` clears the token and redirects to login.
5. Confirm preflight requests permit only the configured frontend origin and required headers.

## Operational Checks

- Health checks expose no credentials, tokens, database URLs, or detailed internal errors.
- Backend logs do not contain JWTs, passwords, AWS keys, or full authorization headers.
- Coolify TLS is enabled for both public services.
- Database backups are scheduled and a restore has been tested.
- Redeploying the backend does not delete application data.
- AWS failures are visible as actionable errors without revealing secret material.

## Required Findings

Report concrete failures with the affected endpoint or configuration, severity, realistic impact, and a specific remediation. Mark controls that cannot be verified from the available Coolify configuration as `control not verified`; do not assume a proxy, firewall, or IAM restriction exists without evidence.
