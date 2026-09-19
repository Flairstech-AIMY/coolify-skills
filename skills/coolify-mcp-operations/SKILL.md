---
name: coolify-mcp-operations
description: 'Use the Coolify MCP tools to inspect, configure, deploy, and verify team-owned resources without browser automation or secret exposure.'
argument-hint: 'Provide the Coolify resource UUIDs, desired read or write operation, and whether deployment or lifecycle changes are authorized.'
---

# Coolify MCP Operations

## Scope

Use this skill when the Coolify MCP tools are available for resource discovery, diagnostics, environment variables, deployment control, or runtime verification. Prefer the richer Coolify2 tools when they provide the required operation; use the original Coolify tools for capabilities that Coolify2 does not expose.

Do not open a browser unless the user explicitly authorizes it. Do not print or request API tokens, database passwords, JWT secrets, SSH private keys, AWS credentials, authorization headers, or full secret-bearing environment responses.

## Configured MCP Servers

This workspace has two Coolify MCP registrations pointing at the same Coolify instance:

### `coolify`

An HTTP MCP server exposed by Coolify:

```jsonc
"coolify": {
	"url": "https://<coolify-host>/mcp",
	"type": "http",
	"headers": {
		"Authorization": "Bearer <coolify-access-token>"
	}
}
```

### `coolify-community-mcp`

The community stdio MCP package, launched through `npx`:

```jsonc
"coolify-community-mcp": {
	"command": "npx",
	"args": ["-y", "@masonator/coolify-mcp"],
	"env": {
		"COOLIFY_BASE_URL": "https://<coolify-host>",
		"COOLIFY_ACCESS_TOKEN": "<coolify-access-token>"
	},
	"type": "stdio"
}
```

Keep tokens in VS Code user settings or a secret manager, never in repository files or skill documentation. If a token is pasted into chat, committed, or otherwise exposed, revoke it and create a replacement before continuing.

The HTTP registration is the primary Coolify MCP surface used for team-scoped, redacted operations. Use the community registration when it exposes a needed operation that the HTTP server does not provide, and verify its behavior before making writes.

## Discovery and Diagnosis

1. Start with the infrastructure overview and resource inventory.
2. Resolve the target resource by name before using a UUID.
3. Read application, database, server, destination, and deployment details before changing state.
4. For deployment failures, inspect deployment status and redacted summaries before retrying.
5. Use live logs only when the resource is running; a stopped resource should return a structured reason rather than repeated log requests.
6. Record proxy/server health separately from application health. A stopped Traefik proxy can cause `502` or `503` responses for otherwise healthy containers.

## Environment Variables

Use the Coolify2 environment-variable tools for listing keys and metadata. Values are masked by default. Keep production and preview scopes separate; the same key may legitimately exist in both scopes.

When changing a secret-bearing URL, preserve the existing credentials and options. Never replace a masked value with an incomplete connection string. If the required value cannot be safely transformed without reading a secret, stop and request a user-approved secure update path.

## Database Connectivity

Do not assume a database display name is its Docker DNS hostname. Coolify application containers do not have stable UUID hostnames, while database containers do. Use the database connection template or verified database hostname from Coolify configuration.

Keep PostgreSQL private unless public access is explicitly required. After troubleshooting, verify whether `is_public` should be reverted to `false`.

## Lifecycle and Deployment

- Treat deployment queue success as incomplete until the deployment is finished and the application is running.
- Change one configuration variable at a time and preserve failed deployment records.
- Restart or redeploy only when the evidence identifies the controlling resource and the user has authorized lifecycle changes.
- Do not delete or recreate a database during application recovery.
- After recovery, verify the backend health route, frontend route, login behavior, and database persistence.

## Reporting

Report confirmed status, resource UUIDs, deployment commit, health results, and the next discriminating check. Redact secrets and full connection strings. Label controls that cannot be verified as `control not verified`.
