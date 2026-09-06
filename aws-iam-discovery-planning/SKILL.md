---
name: aws-iam-discovery-planning
description: 'Plan AWS permissions for this security scanner before deployment. Use when choosing an IAM user or EC2 role, creating least-privilege discovery access, or configuring CloudFront and Route 53 inventory.'
argument-hint: 'Provide the AWS account, discovery sources, deployment runtime, and whether credentials will come from an IAM user or instance role.'
---

# AWS IAM Discovery Planning

## Runtime Choice

This scanner uses `boto3` standard credential resolution. Choose one production credential source:

- Preferred: an attached EC2 instance role or workload identity with no static keys.
- Approved alternative: a dedicated IAM user with an access key stored as an encrypted Coolify runtime secret.

Never configure AWS credentials in the frontend, a `VITE_*` variable, Git, Docker build arguments, screenshots, or logs. Do not use a personal IAM user.

## Required Permissions

The scanner implementation calls these read-only actions:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DiscoverCloudFrontDistributions",
      "Effect": "Allow",
      "Action": "cloudfront:ListDistributions",
      "Resource": "*"
    },
    {
      "Sid": "DiscoverRoute53Zones",
      "Effect": "Allow",
      "Action": "route53:ListHostedZones",
      "Resource": "*"
    },
    {
      "Sid": "DiscoverRoute53Records",
      "Effect": "Allow",
      "Action": "route53:ListResourceRecordSets",
      "Resource": "arn:aws:route53:::hostedzone/*"
    }
  ]
}
```

Attach this as an identity-based permissions policy to the dedicated IAM user or workload role. Do not paste it into a role trust relationship. Trust policies require `Principal` and must not contain `Resource`.

## Deployment Plan

For a dedicated IAM user, create access keys only after the policy is attached and record the access key ID without recording the secret. Configure the backend production application with runtime-only variables:

- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`
- `AWS_DEFAULT_REGION`, normally `us-east-1`

Do not enable build-time access for these variables. Use separate credentials for staging and production, set an owner and rotation date, and prefer the narrowest account scope available.

## Preflight

Before deployment, inspect `backend/app/scanner/aws.py` and confirm every required AWS API call is represented in the policy. Verify the target deployment environment, AWS account, regions, discovery sources, and credential source. Plan a safe caller-identity check that returns only the ARN, never credentials.
