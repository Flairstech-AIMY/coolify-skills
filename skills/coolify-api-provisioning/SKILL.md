---
name: coolify-api-provisioning
description: 'Provision and manage Coolify resources through the REST API, including writable project, PostgreSQL, private SSH application, variable, and deployment operations. Use when browser automation is unavailable or not authorized.'
argument-hint: 'Provide the Coolify base URL, project/environment target, repository source, and whether existing resources must be preserved.'
---

# Coolify API Provisioning

## Scope

Use this skill for repeatable Coolify resource provisioning. Prefer the API for deterministic resource creation and updates, then use the UI or browser automation only for fields or flows that the API cannot safely express.

Never place a real API token, database password, JWT secret, private key, or AWS credential in this file, shell history, source code, or chat output.

## Preconditions

1. Confirm the Coolify base URL and API prefix. This deployment uses `/api/v1`.
2. Load the token from a secure environment variable such as `$env:COOLIFY_TOKEN`; do not echo it.
3. Identify the target project, environment, server, and destination UUIDs.
4. Search for existing resources by UUID or name before creating anything. Keep healthy PostgreSQL resources unless recreation is explicitly approved.
5. Confirm the repository branch and whether the repository is public, GitHub App-backed, or private SSH-backed.

## Request Pattern

Use an authorization header and JSON bodies. Example PowerShell shape:

```powershell
$token = (Get-Content $env:COOLIFY_TOKEN_FILE -Raw).Trim()
$headers = @{
	Authorization = "Bearer $token"
	Accept = 'application/json'
	'Content-Type' = 'application/json'
}
Invoke-RestMethod -Method Get -Uri "$env:COOLIFY_URL/api/v1/projects" -Headers $headers
```

Use the API documentation or the running instance to confirm exact endpoint names and required fields for the installed Coolify version. Do not guess a UUID or silently treat a `404` as an empty result.

For the concrete writable request schemas and PowerShell patterns, load `coolify-api-direct-rest`. Confirmed routes include `POST /projects`, `POST /databases/postgresql`, `POST /applications/private-deploy-key`, `POST /applications/{uuid}/envs`, `POST /databases/{uuid}/start`, and `POST /deploy`.

## Provisioning Order

1. Project and production environment.
2. Persistent PostgreSQL database.
3. Backend application.
4. Backend variables, domain, and health check.
5. Frontend application.
6. Frontend build variables, domain, and health check.
7. Deploy database, backend, then frontend.

After each create or update operation, read the resource back and verify the effective configuration. Record UUIDs and non-secret status only.

## Idempotency and Safety

- Search before POSTing; do not create duplicate applications after a timeout without checking resource state.
- Preserve existing database UUIDs and data.
- Use PATCH/update operations for configuration changes where supported.
- Treat deployment initiation as asynchronous. Capture the deployment UUID and poll status with bounded retries.
- On failure, retain the failed deployment record and inspect it before retrying.
- Delete old failed applications only after replacement services are healthy and verified.

## Variables and Secrets

Set secret values through Coolify's secret/environment-variable mechanism. List variable names and metadata for auditing, never print values. Backend values normally include `DATABASE_URL`, `SECRET_KEY`, token expiry, and exact frontend `CORS_ORIGINS`; frontend build-time values may include only the public API URL.

## Failure Handling

Classify failures as API validation, source checkout, image build, container startup, proxy/routing, or application behavior. A successful API response means only that Coolify accepted the request; it does not prove that checkout or deployment succeeded.
