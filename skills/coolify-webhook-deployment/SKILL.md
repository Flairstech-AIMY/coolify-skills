---
name: coolify-webhook-deployment
description: 'Configure, trigger, troubleshoot, and verify Coolify deployments initiated by deploy webhooks from GitHub Actions or another CI system. Use when handling COOLIFY_DEPLOY_WEBHOOK, COOLIFY_TOKEN, webhook authentication, HTTP errors, or asynchronous deployment results.'
argument-hint: 'Describe the CI provider, Coolify webhook/API URL type, target resource, HTTP status, and redacted response details.'
---

# Coolify Webhook Deployment

## Scope

Use this skill to connect a CI workflow or other automation to a Coolify deploy webhook and verify the resulting deployment. This skill covers trigger configuration and failure diagnosis; it does not authorize deploying or changing a production resource without the user's request.

## Before Configuring the Trigger

1. Confirm the intended Coolify application, service, or tag and the branch/event that should trigger deployment.
2. Copy the deploy webhook URL from the target Coolify resource, or validate the direct API endpoint against the official documentation for the running Coolify version: <https://coolify.io/docs/core/automation/deploy-webhooks> and <https://coolify.io/docs/api-reference>.
3. Confirm the instance has API access enabled. On self-hosted Coolify, this is controlled in **Settings > Advanced**.
4. Create a team-scoped API token with only the `deploy` permission and a suitable expiration. Save the token in the CI provider's secret store, for example as `COOLIFY_TOKEN`. Save the full copied URL as a secret too if its query string identifies a private resource, for example `COOLIFY_DEPLOY_WEBHOOK`.
5. Never put token values in workflow YAML, command-line arguments outside the process environment, repository files, URLs, logs, artifacts, or chat. Do not enable shell tracing or `curl -v` around secret-bearing requests.

## Triggering from CI

Prefer the copied resource webhook URL. It already identifies the deployment target; do not append or replace its parameters unless the documented workflow requires it. Coolify's deploy webhook documentation uses `GET` with an `Authorization: Bearer` header and also accepts `POST`. For a `POST` to the API endpoint, send documented options such as `uuid`, `tag`, and `force` in a JSON body. Use either `uuid` or `tag`, never both.

For GitHub Actions, inject secrets through step-level environment variables and do not echo them:

```yaml
- name: Trigger Coolify deployment
  env:
    COOLIFY_DEPLOY_WEBHOOK: ${{ secrets.COOLIFY_DEPLOY_WEBHOOK }}
    COOLIFY_TOKEN: ${{ secrets.COOLIFY_TOKEN }}
  run: |
    if [ -z "$COOLIFY_DEPLOY_WEBHOOK" ] || [ -z "$COOLIFY_TOKEN" ]; then
      echo "::error::Required Coolify secrets are not configured."
      exit 1
    fi
    curl --fail --silent --show-error --request GET "$COOLIFY_DEPLOY_WEBHOOK" \
      --header "Authorization: Bearer $COOLIFY_TOKEN"
```

Adapt the method and request body only after checking the URL type and current Coolify documentation. If switching to `POST`, do not assume an arbitrary webhook URL accepts the same payload as the API root; follow the documented contract for the copied URL/API route.

Restrict deployment jobs to trusted events and branches. Do not expose Coolify secrets to forked pull requests. Use workflow concurrency controls where overlapping deployments could race, and avoid force-rebuild options unless needed.

## Diagnose HTTP Failures

Do not guess a route or method from a status code. Check the exact endpoint type, documented method, configured URL, and Coolify version first. Keep response output redacted; never print authorization headers, token values, cookies, or secret-bearing URLs.

| Status | Checks |
| --- | --- |
| `401` | Token is present, unexpired, correctly scoped to the active team, and sent as an `Authorization: Bearer <token>` header. |
| `403` | Token has the `deploy` ability and the user/token is allowed to deploy the target resource. |
| `404` | Host, API prefix, copied webhook URL, resource UUID/tag, and any proxy path rewriting are correct. |
| `405` | URL and HTTP method match the documented webhook/API contract for this Coolify instance. Do not blindly change `GET` to `POST` (or vice versa); validate whether this is a copied webhook URL or the API endpoint and use the matching body/query format. |
| `422` | Request contains exactly one valid target (`uuid` or `tag`) and only documented options. |
| `5xx` | Inspect Coolify availability and the deployment record before retrying; avoid duplicate triggers. |

A missing-secret guard should fail without displaying secret values. Avoid `curl --location` unless redirects are expected and their destination has been validated; never forward a bearer token to an untrusted host.

## Verify the Deployment

1. Treat a successful HTTP response as an accepted/queued request, not proof that deployment completed or the app is healthy.
2. Capture the deployment UUID and target resource from the response when available, without retaining secret-bearing request details.
3. Check Coolify's deployment record for the intended resource and commit/branch. Confirm the deployment finishes successfully.
4. Verify the resource is running and healthy, then test its public endpoint and critical dependencies when applicable.
5. If the trigger was accepted but deployment fails, diagnose the deployment itself using `coolify-deployment-diagnostics`; do not repeatedly fire the webhook to compensate for an unverified failure.

## Reporting

Report the target resource, trigger method, HTTP status, deployment identifier/status, and verification performed. State whether the deployment was merely queued or confirmed healthy. Redact credentials and secret-bearing URLs from all output.
