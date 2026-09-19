---
name: coolify-api-reference-validation
description: 'Validate Coolify REST API routes, payloads, and response semantics against the official API reference before making deployment changes.'
argument-hint: 'Provide the Coolify base URL, API version, target resource, and intended read or write operation.'
---

# Coolify API Reference Validation

## Scope

Use this skill before direct REST writes or when the MCP surface does not expose a required Coolify operation. Consult the official API reference for the installed Coolify version, then validate the route against the running instance before changing resources.

Official documentation:

`https://coolify.io/docs/api-reference`

Do not use the browser unless the user explicitly authorizes it. Use the REST API and local documentation retrieval where possible. Do not guess a route or payload when the installed API rejects or differs from the reference.

## Authentication

Load the Coolify token from a protected environment variable or local token file into memory only. Send it in an `Authorization: Bearer` header. Never print the token, headers, cookies, full environment-variable responses, or secret request bodies.

Use the deployment API root:

`https://<coolify-host>/api/v1`

Verify the host and API prefix before writing.

## Read Before Write

1. Read the official reference for the target resource and operation.
2. Resolve the resource by UUID or exact name.
3. Read the current resource state.
4. Record the minimum non-secret fields needed to compare before and after.
5. Use the smallest documented payload that satisfies the requested change.
6. Re-read the resource after a successful write.

For database exposure changes, compare `is_public`, `public_port`, `ports_mappings`, status, and restart timestamps. A `200` response proves only that Coolify accepted the request; it does not prove network reachability.

## Write Safety

- Preserve existing data and resource UUIDs.
- Do not recreate a healthy database to change configuration.
- Do not include secret values unless the operation explicitly requires them.
- Make one configuration change at a time.
- Capture the HTTP status and redacted error category.
- Stop on an undocumented validation error rather than retrying guessed payloads.
- Roll back a reversible setting when the requested operation fails verification, unless the user explicitly asks to leave the change in place.

## Deployment Changes

Treat deployment and restart operations as asynchronous. Capture the deployment UUID when returned, inspect its status, and verify the application afterward. Keep failed deployment records for diagnosis.

For environment variables, list names and metadata only. Never read back or include `DATABASE_URL`, `SECRET_KEY`, AWS credentials, passwords, or tokens in output.

## Reporting

Report the documentation URL, route and method used, resource UUID, redacted request fields, HTTP result, effective post-change state, and any unverified control. Clearly distinguish configuration state from confirmed external connectivity.
