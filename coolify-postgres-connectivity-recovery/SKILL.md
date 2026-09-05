---
name: coolify-postgres-connectivity-recovery
description: 'Diagnose and repair Coolify application startup failures caused by PostgreSQL DNS, network, or DATABASE_URL configuration problems. Use when FastAPI or SQLAlchemy reports could not translate host name, connection refused, timeout, or application startup failed while the Coolify database is healthy.'
argument-hint: 'Provide the backend application UUID or name, database UUID or name, deployment error, and whether API-only operations are authorized.'
---

# Coolify PostgreSQL Connectivity Recovery

## Scope

Use this skill when a Coolify-hosted backend cannot start because its PostgreSQL connection fails. The common symptom is an SQLAlchemy or psycopg2 error such as `could not translate host name`, followed by an application startup failure in a lifespan hook, migration, schema inspection, or health check.

Use Coolify API and terminal operations only unless the user explicitly authorizes browser automation. Preserve the existing database resource, volume, and data. Do not modify application source code to hide a deployment configuration error.

Never print or commit database passwords, complete `DATABASE_URL` values, API tokens, JWT secrets, SSH private keys, or authorization headers. Treat any retrieved connection string as secret material.

## Root-Cause Model

Separate these failure boundaries before changing anything:

| Evidence | Likely boundary |
| --- | --- |
| Database resource is unhealthy or stopped | PostgreSQL resource lifecycle or server issue |
| Database is healthy but hostname cannot resolve | Wrong internal hostname, missing Docker network attachment, or stale variable |
| Host resolves but connection is refused or times out | Port, network policy, container binding, or database readiness |
| Connection succeeds but startup fails during schema work | Credentials, database name, TLS mode, schema state, or application logic |
| Backend is healthy but public URL fails | Proxy, port, health-check, domain, or CORS configuration |

A Coolify database display name is not necessarily its Docker DNS hostname. Prefer the hostname from Coolify's generated internal connection template. On standalone PostgreSQL resources this is commonly the database resource UUID, not the display name.

## Evidence-First Triage

1. Resolve the application and database by exact name or UUID.
2. Record only UUIDs, names, project/environment, destination, status, restart count, last-online time, FQDN, exposed port, and deployment UUIDs.
3. Confirm the application and database use the same Coolify project, environment, server, destination, and Docker network.
4. Read the database metadata and confirm it is `running:healthy` before changing the backend.
5. List application environment-variable names and metadata without revealing values. Check both production and preview scopes separately.
6. Read the failed deployment record before retrying. Preserve its UUID and earliest error category.
7. If runtime logs are available, request a bounded tail with a line limit. A stopped or crash-looping container may return no logs; that does not prove a build failure.

Useful Coolify API/MCP reads include:

- Search resources by application/database name.
- Get application and database details.
- List application environment-variable keys and metadata.
- List deployment history for the application.
- Get a deployment with a bounded redacted log summary.
- Get infrastructure overview and destination/server health.

## Discriminating Checks

### 1. Compare the configured host to Coolify's generated host

Compare only redacted URL components. Do not paste a full URL into chat or logs.

- Extract the `DATABASE_URL` hostname in a secure local context, or inspect it through an approved secret-handling path.
- Obtain the database's generated internal connection template through Coolify's protected metadata path.
- Compare hostname, port, and database name.
- Do not substitute the database display name merely because it looks plausible.
- Do not change credentials, TLS mode, or schema settings until a hostname mismatch is ruled out.

For a DNS error involving a name like `aimy-scanner-postgres`, the first hypothesis is a stale manually entered hostname. A generated host resembling the database UUID is expected in many Coolify standalone-Docker deployments.

### 2. Check network attachment only after the hostname check

Confirm that the application and database share the same destination/network. An application setting such as `connect_to_docker_network: false` can matter, but changing it is not a substitute for correcting a stale hostname. Change network attachment only when the platform's generated connection template and resource topology require it, and redeploy afterward.

### 3. Check scope and lifecycle

Production and preview variables are separate. A correct preview `DATABASE_URL` does not repair production. Confirm the deployment uses the production environment and that the variable is runtime-enabled. A successful image build does not prove the runtime variable was applied.

## Safe Repair Procedure

1. Keep the database running and do not recreate it.
2. Generate or obtain the corrected internal connection string through a secure, authorized path. Preserve the existing username/password unless credential rotation is explicitly requested.
3. Update only the production `DATABASE_URL`. Keep `is_runtime=true`, and keep `is_buildtime` consistent with the application configuration. Do not alter preview variables unless requested.
4. If a shown-once variable cannot be updated in place:
   - Do not guess or reconstruct its password.
   - Do not delete it until the replacement value is securely available.
   - Prefer a direct REST `PATCH /applications/{application_uuid}/envs` update by `key`, omitting the environment-variable UUID when the API requires that form.
   - If the API rejects the update but a secure replacement is available, delete only the targeted production variable and recreate it with the corrected value and secret metadata. Confirm that no duplicate production key remains.
5. Do not expose the replacement value in tool output, deployment descriptions, source files, or shell history.
6. Trigger one backend deployment after the variable change. Do not retry repeatedly at the same failure boundary.
7. Preserve the deployment UUID and wait for a terminal deployment status.

## Validation

A deployment trigger being accepted is not sufficient. Verify all of the following:

- The deployment reaches `finished`.
- The application container is present and no longer crash-looping.
- The backend health endpoint returns HTTP 200 with the expected small response.
- The PostgreSQL resource remains `running:healthy`.
- The application environment metadata contains exactly one production `DATABASE_URL` and a separate preview entry only when preview deployments are intended.
- No password, token, or complete connection string appears in returned logs or responses.
- A protected backend route still rejects an unauthenticated request with HTTP 401, when a safe endpoint is available for verification.

If the health endpoint still fails, classify the new error before changing another variable:

- `could not translate host name`: hostname or Docker network remains wrong.
- `connection refused` or timeout: destination/network/readiness issue.
- `password authentication failed`: credential mismatch; rotate or repair credentials through the database access procedure.
- `database does not exist`: database-name mismatch.
- SSL errors: align the URL scheme and `sslmode` with the database's configured TLS mode.
- schema or migration error after connection succeeds: investigate application schema handling separately.

## Security and Recovery Notes

- If a database password was revealed during diagnosis or entered into an unsafe command, rotate it and update the backend variable through a secret-safe path.
- Keep PostgreSQL private unless public exposure is explicitly authorized and allowlisted.
- Do not use the database resource display name as a DNS alias without verifying that Coolify created that alias.
- Do not reset schema or drop tables as a connectivity fix.
- Do not claim that a stale Coolify status field proves the backend is down when the live HTTPS health check succeeds; use the endpoint and deployment record as validation evidence.
- Report the root cause, changed resource/variable name, deployment UUID, and redacted validation results. Never report the connection string or password.
