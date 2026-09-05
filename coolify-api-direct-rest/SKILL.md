---
name: coolify-api-direct-rest
description: 'Use the Coolify REST API directly for writable provisioning and lifecycle operations. Use when creating or updating Coolify projects, PostgreSQL databases, private SSH applications, environment variables, or deployments without browser automation.'
argument-hint: 'Provide the Coolify API base URL, target repository, resource names, domains, and whether existing resources must be preserved.'
---

# Coolify Direct REST Operations

## Scope

Use this skill when the Coolify MCP surface is read-only or when deterministic REST writes are required. This workflow supports project creation, PostgreSQL creation, private SSH application creation, environment variables, deployment triggers, and status verification.

Do not use the browser unless the user explicitly authorizes it. Leave UI-only actions such as interactive login, MFA, private-key creation, or fields not exposed by the API to the user.

Never print or commit API tokens, database passwords, JWT secrets, SSH private keys, AWS credentials, authorization headers, or full environment-variable responses.

## Connection

Confirm the base URL and API prefix before writing. For this deployment the effective API root is:

```text
https://<coolify-host>/api/v1
```

Load the token only in memory. Prefer a secure environment variable; a user-provided local token file is acceptable when it is read without echoing its contents:

```powershell
$token = (Get-Content $env:COOLIFY_TOKEN_FILE -Raw).Trim()
$headers = @{
    Authorization = "Bearer $token"
    Accept = 'application/json'
    'Content-Type' = 'application/json'
}
```

Do not put the token directly in a command, URL, source file, or chat message. Clear local variables when the operation is complete where practical.

## Discovery Before Writes

1. `GET /projects`
2. `GET /servers`
3. `GET /destinations`
4. `GET /databases`
5. `GET /security/keys`
6. Search applications and databases by name before creating anything.

Record only UUIDs, names, configuration metadata, and status. Do not assume a missing relation field in a create response means the resource was not associated; read the resource back and inspect the project/resource inventory.

## Confirmed Write Routes

### Project

`POST /projects`

```json
{
  "name": "<project-name>",
  "description": "<description>"
}
```

A production environment is commonly created automatically. Read `GET /projects/{project_uuid}` before creating another environment.

### PostgreSQL

`POST /databases/postgresql`

Required deployment-specific fields normally include:

```json
{
  "server_uuid": "<server-uuid>",
  "destination_uuid": "<destination-uuid>",
  "project_uuid": "<project-uuid>",
  "environment_name": "production",
  "name": "<database-name>",
  "postgres_user": "<database-user>",
  "postgres_password": "<generated-password>",
  "postgres_db": "<database-name>",
  "image": "postgres:16-alpine",
  "is_public": false,
  "instant_deploy": false
}
```

Generate the password in memory. Never print it. Read the database back after creation and start it with:

`POST /databases/{database_uuid}/start`

The start endpoint may reject `GET` with `405`; use the documented `POST` form with an empty JSON body. Keep the database private and persistent.

Do not assume the database resource name is its internal DNS hostname. Prefer the connection template or the hostname returned by the target Coolify configuration. A resource name such as `aimy-scanner-postgres` can differ from the internal hostname used by application containers.

### Private SSH application

`POST /applications/private-deploy-key`

```json
{
  "name": "<application-name>",
  "project_uuid": "<project-uuid>",
  "server_uuid": "<server-uuid>",
  "environment_name": "production",
  "private_key_uuid": "<security-key-uuid>",
  "git_repository": "git@github.com:<org>/<repo>.git",
  "git_branch": "main",
  "build_pack": "dockerfile",
  "ports_exposes": "8000",
  "base_directory": "/backend",
  "dockerfile_location": "/Dockerfile",
  "instant_deploy": false
}
```

For a frontend application use its own name, `/frontend` base directory, port `80`, and the same SSH private-key UUID. The key UUID is obtained from `GET /security/keys`; do not confuse the key name with the UUID.

Validate repository access independently with an explicit SSH identity:

```powershell
$key = Join-Path $env:USERPROFILE '.coolify\<key-name>'
$ssh = "ssh -i `"$key`" -o IdentitiesOnly=yes -o StrictHostKeyChecking=accept-new"
git -c core.sshCommand=$ssh ls-remote git@github.com:<org>/<repo>.git refs/heads/main
```

### Application environment variables

`POST /applications/{application_uuid}/envs`

```json
{
  "key": "<name>",
  "value": "<value>",
  "is_preview": false,
  "is_literal": true,
  "is_multiline": false,
  "is_shown_once": true
}
```

Use `is_shown_once: true` for secrets such as `DATABASE_URL` and `SECRET_KEY`. Never list or read back their values. Build-time frontend configuration such as `VITE_API_BASE_URL` may be public, but it must contain only a public HTTPS API URL or a deliberate relative path.

Coolify may create preview copies of variables automatically. When changing an existing production variable, use the application env collection endpoint with `PATCH`, identify the target by its `key`, and omit the returned `uuid` from the JSON body:

`PATCH /applications/{application_uuid}/envs`

```json
{
  "key": "CORS_ORIGINS",
  "value": "https://<frontend-fqdn>",
  "is_preview": false,
  "is_literal": true,
  "is_multiline": false,
  "is_shown_once": false
}
```

Including `uuid` in this payload is rejected with `422`. Confirm the updated variable by listing names and metadata only.

### Deployment

Trigger a deployment with:

`POST /deploy`

```json
{
  "uuid": "<application_uuid>"
}
```

Capture the returned deployment UUID. Verify it with `GET /deployments/{deployment_uuid}` and treat `in_progress` as incomplete. A successful trigger response only means that Coolify queued the deployment.

Read the application after creation or deployment and use its generated `fqdn` when present. This is safer than inventing a domain. Update the backend CORS origin and frontend API URL from those generated FQDNs before the final redeploy.

## Deployment Order

1. Create or reuse the project.
2. Create and start PostgreSQL.
3. Create backend application.
4. Set backend variables and domain.
5. Deploy and verify backend health.
6. Create frontend application.
7. Set the frontend API build variable and domain.
8. Deploy and verify the frontend and a client-side deep link.
9. Bootstrap the first admin through the backend only after HTTPS and CORS are configured.

## Idempotency and Failure Handling

- Search by name and UUID before every POST.
- If a POST times out, query resources before retrying.
- Read every created or updated resource back.
- Keep failed deployment records; inspect status and reason before retrying.
- Do not delete failed resources until replacement services are healthy.
- Classify failures as source checkout, image build, container startup, database connectivity, proxy/routing, or application behavior.
- If the API schema differs from the published documentation, inspect the installed route response and stop rather than guessing destructive payloads.

## Domain Boundary

The API can provision resources without public domains, but production verification requires explicit backend and frontend HTTPS domains. Never invent domains. Ask the user for the intended domains when they are not discoverable from the target deployment configuration, then update `CORS_ORIGINS` and `VITE_API_BASE_URL` before the final deploy.
