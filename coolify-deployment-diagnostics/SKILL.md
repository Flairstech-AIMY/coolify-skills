---
name: coolify-deployment-diagnostics
description: 'Diagnose Coolify deployment failures using deployment records, source checkout evidence, application status, proxy state, and Git credential checks. Use when a deployment fails quickly, logs are sparse, or Coolify reports an application is not running.'
argument-hint: 'Provide the application UUID, deployment UUID, status, and the earliest available error evidence.'
---

# Coolify Deployment Diagnostics

## Evidence-First Triage

1. Record the application UUID, deployment UUID, requested commit/branch, and timestamps.
2. Read the deployment record and summary before retrying.
3. Identify whether a commit, config hash, image tag, or build output exists.
4. Check application status and logs only when the application is running or the platform provides a useful stopped-state reason.
5. Check server and destination health, then proxy status if routing is implicated.

Interpret the lifecycle boundary:

| Evidence | Most likely boundary |
| --- | --- |
| No commit, config hash, image tag, or build output; failure within seconds | Source lookup, credentials, repository, branch, or Coolify configuration |
| Commit exists but no image | Dockerfile path, build context, dependency, or registry build |
| Image exists but container exits | Startup command, environment variable, migration, or application exception |
| Container is running but endpoint fails | Port, health check, domain, proxy, CORS, or application route |

Do not infer a build error from `Application is not running` alone. That message describes the current runtime state, not necessarily the original failure.

## Repository Access Pitfall

On Windows, a credential helper may satisfy local HTTPS Git commands using cached credentials. Disable or bypass helpers for a diagnostic command and test the same access mode configured in Coolify. For private SSH deployment, use the exact key and SSH URL configured in Coolify.

## Proxy and Platform Failures

A stopped proxy can explain routing and health-check failures, but it does not explain a deployment that fails before source checkout. Record proxy status separately, restart only with authorization, and retest after recovery. Do not combine unrelated proxy symptoms with a source-authentication diagnosis.

## Safe Retry Loop

- Preserve the original failure record.
- Change one relevant variable at a time.
- Verify repository access before retrying deployment.
- Retry only after the controlling issue has evidence-based remediation.
- Compare the new deployment boundary with the old one.

Do not loop on retries when the failure occurs at the same pre-check stage. Escalate with the deployment ID and redacted evidence.

## Reporting

Report confirmed facts, likely boundary, discriminating check, and next action. Redact tokens, private keys, database URLs, JWT secrets, AWS credentials, and authorization headers. Mark unverified controls as `control not verified` instead of assuming that a firewall, proxy, or Git integration is configured.
