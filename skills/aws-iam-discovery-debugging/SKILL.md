---
name: aws-iam-discovery-debugging
description: 'Debug AWS CloudFront or Route 53 discovery failures in this scanner. Use for AccessDenied, missing credentials, incomplete credentials, wrong account, wrong IAM principal, or region-specific discovery errors.'
argument-hint: 'Provide the redacted discovery error, deployment resource, AWS source, region, and whether credentials come from an IAM user or workload role.'
---

# AWS IAM Discovery Debugging

## Read the Principal First

Start with the complete redacted AWS error. Identify:

- AWS account ID
- Principal ARN and whether it is an IAM user or assumed role
- Action denied
- Service and region
- Whether the error is `AccessDenied`, missing credentials, or incomplete credentials

An ARN such as `arn:aws:sts::ACCOUNT:assumed-role/ROLE/INSTANCE` means the application is using an assumed role, not an IAM user. The policy must be attached to that role. Do not add an IAM user policy until the running principal is confirmed.

## Policy Boundary

Distinguish these policy locations:

- IAM user or role **Permissions**: attach `cloudfront:ListDistributions`, `route53:ListHostedZones`, and `route53:ListResourceRecordSets` here.
- IAM role **Trust relationship**: controls who can assume the role; it requires `Principal` and has no `Resource`.
- Coolify environment variables: provide credentials only; they do not grant AWS permissions.

The scanner reads CloudFront distributions and Route 53 hosted-zone records. Check both the action name and the AWS account containing those resources. Do not broaden permissions to `AdministratorAccess` as a diagnostic shortcut.

## Safe Discriminating Checks

Use the backend runtime, not a developer workstation, as the source of truth. Run a caller-identity check that prints only the ARN:

```bash
python -c "import boto3; print(boto3.client('sts').get_caller_identity()['Arn'])"
```

Then test the specific read-only APIs through the scanner or an authorized backend terminal. Do not print environment variables, access keys, secret keys, session tokens, or full credential configuration.

Interpretation:

- `NoCredentialsError`: no usable credentials reached the backend container.
- `PartialCredentialsError`: one required credential is missing or malformed.
- `AccessDenied`: credentials work, but the identified principal lacks the denied action or is in the wrong account.
- Discovery succeeds in one source but not another: compare the source-specific action and policy statement.
- Errors mention an old role after adding IAM-user variables: the container was not redeployed, variables are in the wrong scope, or credentials are invalid and boto3 fell back to the instance role.

## Region Note

CloudFront and Route 53 are global services. The scanner may iterate configured regions for inventory, but regional iteration does not require separate regional IAM policies. Treat repeated errors across `us-east-1` and `us-east-2` as one principal/policy problem unless the error differs.

## Reporting

Report the confirmed principal, denied action, account, deployment scope, and one next check. Redact access keys, secret keys, session tokens, database URLs, and authorization headers.
