# Quarterly Access Review Process

Quarterly access reviews validate that user and privileged access remains appropriate for current roles and business need.

## Why It Matters

Stale, excessive, or orphaned access is one of the most common control findings in cloud environments. Regular reviews catch these before they become audit findings or security incidents.

## Review Scope

- Privileged accounts (admin, root, break-glass)
- IAM roles with broad permissions
- Service accounts
- Shared accounts
- Users with elevated or sensitive access
- Former users and inactive accounts

## Review Steps

1. Export current users, groups, and privileged roles from IAM or IdP
2. Validate account owner and current business need
3. Identify dormant, orphaned, or excessive access
4. Record revocation or adjustment actions
5. Retain reviewer sign-off and evidence
6. Track unresolved issues in POA&M or ticketing system

## AWS CLI — Pull Access Review Data

```bash
# Generate credential report (includes last activity)
aws iam generate-credential-report
aws iam get-credential-report --query Content --output text | base64 -d

# List all IAM users
aws iam list-users --query 'Users[].{User:UserName,Created:CreateDate}' --output table

# List users with no MFA
aws iam get-credential-report --query Content --output text | base64 -d | \
  awk -F, 'NR>1 && $4=="true" && $8=="false" {print $1, "- Console access, NO MFA"}'

# List all IAM roles
aws iam list-roles --query 'Roles[].{Role:RoleName,Created:CreateDate}' --output table
```

## Evidence to Retain

- IAM or IdP export used for review
- Reviewer name and sign-off date
- Ticket or remediation record for any actions taken
- Updated role or group membership (if changed)
