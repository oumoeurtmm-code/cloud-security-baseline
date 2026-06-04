# IAM Least-Privilege Design Principles

IAM is the most critical security control in AWS. Misconfigured IAM is the root cause of the majority of cloud security incidents. This document defines the design principles applied throughout this baseline.

---

## Core Principles

### 1. Least Privilege
Grant only the permissions required to perform a specific task. Start with zero access and add only what is necessary. Never start with broad access and try to narrow it down.

```json
// ❌ Bad — wildcard on all S3 actions and resources
{
  "Effect": "Allow",
  "Action": "s3:*",
  "Resource": "*"
}

// ✅ Good — specific actions on specific bucket
{
  "Effect": "Allow",
  "Action": ["s3:GetObject", "s3:ListBucket"],
  "Resource": [
    "arn:aws:s3:::my-specific-bucket",
    "arn:aws:s3:::my-specific-bucket/*"
  ]
}
```

### 2. Separation of Duties
No single principal should have the ability to both perform an action and audit or approve it. Example: the role that deploys infrastructure should not have the ability to modify IAM policies.

### 3. No Long-Lived Root Credentials
- Root account: MFA required, access keys must not exist
- Use `aws iam delete-virtual-mfa-device` to audit
- Verify with: `aws iam get-account-summary --query 'SummaryMap.AccountMFAEnabled'` (must return `1`)

### 4. Roles Over Users
Prefer IAM Roles over IAM Users for all service-to-service access. Roles use temporary credentials; users use long-lived access keys.

### 5. Permission Boundaries
For delegated administration (e.g., developers creating their own roles), use Permission Boundaries to cap the maximum permissions any role they create can have.

```json
// Permission boundary example — cap at specific services
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": ["s3:*", "cloudwatch:*", "logs:*"],
    "Resource": "*"
  }]
}
```

---

## IAM Role Hierarchy (Recommended)

```
Organization Root
├── Security Account (centralized logging, GuardDuty, Security Hub)
├── Shared Services Account (DNS, networking, shared tooling)
└── Workload Accounts
    ├── Roles (attached to services/EC2/Lambda)
    │   ├── AppRole — application-specific, least privilege
    │   ├── PlatformOperatorRole — infrastructure operations, no IAM
    │   ├── SecurityAuditorRole — read-only across all services
    │   └── FinOpsRole — read-only billing and Cost Explorer
    └── Human Access (via SSO / Identity Center only)
        ├── Developer — dev account only, no prod
        └── Admin — break-glass only, MFA + logging required
```

---

## MFA Enforcement Policy

Apply this IAM policy to deny all actions (except MFA setup) if MFA is not active:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyWithoutMFA",
      "Effect": "Deny",
      "NotAction": [
        "iam:CreateVirtualMFADevice",
        "iam:EnableMFADevice",
        "iam:GetUser",
        "iam:ListMFADevices",
        "sts:GetSessionToken"
      ],
      "Resource": "*",
      "Condition": {
        "BoolIfExists": {
          "aws:MultiFactorAuthPresent": "false"
        }
      }
    }
  ]
}
```
