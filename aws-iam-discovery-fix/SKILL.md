---
name: aws-iam-discovery-fix
description: 'Fix AWS discovery authorization for this Coolify backend. Use after diagnosing IAM AccessDenied, missing credentials, wrong principal, incorrect trust policy placement, or missing AWS runtime variables.'
argument-hint: 'Provide the confirmed runtime principal, denied AWS actions, backend application UUID, and whether the fix is an IAM policy or Coolify configuration change.'
---

# AWS IAM Discovery Fix

## Fix the Correct Layer

For an EC2 role principal, attach the read-only discovery policy to the role's **Permissions** tab. Do not modify its trust relationship unless the instance cannot assume the role.

For a dedicated IAM user, attach the same policy to the user's permissions, then add the credentials only to the Coolify backend application in the production environment:

- `AWS_ACCESS_KEY_ID`: runtime on, build time off
- `AWS_SECRET_ACCESS_KEY`: runtime on, build time off
- `AWS_DEFAULT_REGION`: runtime on, build time off, usually `us-east-1`

The backend already uses `boto3` standard environment credential loading, so no source change is required for normal IAM user credentials. Do not add them to the frontend or any `VITE_*` variable.

## Deployment Sequence

1. Confirm the IAM policy is attached to the selected principal.
2. Confirm production Coolify variables exist under the backend application, not preview or frontend scope.
3. Keep secret values masked and never copy them into chat or logs.
4. Redeploy the backend so runtime variables are injected into the container.
5. Wait for deployment status `finished` and confirm `/health` returns `200`.
6. Run a caller-identity check from the backend runtime and compare only the principal ARN.
7. Rerun discovery through the application.

If the runtime ARN still shows the EC2 role after configuring an IAM user, stop and correct scope or redeploy rather than changing application code. If the ARN is the IAM user but `AccessDenied` remains, correct the IAM user policy or account/resource scope.

## Rotation and Cleanup

After successful verification, document the credential owner and rotation date. Revoke unused old keys. If a secret was pasted into chat, committed, or exposed in logs, rotate it immediately and replace the Coolify variable before continuing.

Do not delete or recreate PostgreSQL while fixing AWS discovery. Preserve deployment records and database data.

## Closeout Evidence

Record the backend application, deployment commit or deployment ID, runtime principal type, health result, and discovery result. Do not record the access key, secret key, session token, full environment output, or credential-bearing logs.
