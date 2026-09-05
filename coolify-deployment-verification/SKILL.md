---
name: coolify-deployment-verification
description: 'Verify a complete Coolify deployment of the FastAPI backend, Vite frontend, and PostgreSQL dependency. Use after deployment, migration, domain changes, health-check changes, or replacement of failed applications.'
argument-hint: 'Provide the frontend URL, API URL, database resource status, and whether this is a fresh deployment or a replacement.'
---

# Coolify Deployment Verification

## Dependency and Runtime Checks

Verify in order:

1. PostgreSQL is healthy, private, persistent, and unchanged if it contains existing data.
2. Backend is running and responds to `GET /health` over HTTPS.
3. Frontend is running and responds to `/health`.
4. Frontend root and a client-side deep link both return the SPA through the Nginx fallback.
5. Frontend API requests reach the backend using the configured `/api` base URL.

Use bounded requests and record status codes, response shape, and timing without recording response secrets or authorization headers.

## Functional Checks

- Bootstrap the first admin only through the intended route and only when the database is new.
- Confirm login succeeds and protected routes reject missing, invalid, and expired tokens.
- Reload a protected frontend route and confirm the intended session behavior.
- Confirm a backend `401` clears stale frontend authentication and returns the user to login.
- Confirm CORS allows the exact frontend HTTPS origin and required headers, not `*` with credentials.
- Run a minimal discovery operation only when AWS variables are configured and authorized.
- Redeploy the backend and verify database records survive.

## Replacement and Cleanup

Keep old failed applications until the replacement backend and frontend pass the checks above. Compare domains and environment variables before removing old resources. Never delete or recreate PostgreSQL as part of application replacement unless data migration and backup/restore have been explicitly planned.

## Schema Warning

Inspect `backend/app/main.py` before deploying against an existing database. This application performs startup schema checks and may reset scan tables when it detects incompatibility. Back up the database and use a staging deployment before schema-affecting changes.

## Evidence to Record

Record service UUIDs, deployed commit, health status, domain status, and test results. Do not record database URLs, passwords, JWT secrets, AWS credentials, private keys, API tokens, or full logs.
