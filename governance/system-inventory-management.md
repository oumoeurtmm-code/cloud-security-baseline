# System Inventory Management

System inventory management supports governance, incident response, vulnerability management, and compliance by maintaining an accurate record of in-scope assets and services.

## Why It Matters

You cannot govern, secure, or monitor what you cannot see. An accurate inventory is the foundation for control mapping, vulnerability prioritization, and incident response scoping.

## Inventory Should Include

- AWS accounts and account IDs
- VPCs and subnets
- EC2 instances
- RDS databases
- S3 buckets (especially publicly accessible or sensitive)
- IAM roles and privileged identities
- Logging and monitoring services
- Key third-party integrations

## Minimum Fields Per Asset

| Field | Description |
|---|---|
| Asset ID | Unique identifier |
| Asset Type | EC2, RDS, S3, IAM role, Lambda, etc. |
| Environment | Dev / Test / Prod / Shared |
| Owner | Technical or business owner |
| Data Classification | Public / Internal / Confidential |
| Logging Enabled | Yes / No |
| Criticality | Low / Moderate / High |
| Last Review Date | Governance review date |

## AWS CLI — Pull Inventory Data

```bash
# List all running EC2 instances with tags
aws ec2 describe-instances \
  --filters Name=instance-state-name,Values=running \
  --query 'Reservations[].Instances[].{ID:InstanceId,Type:InstanceType,Name:Tags[?Key==`Name`].Value|,Env:Tags[?Key==`Environment`].Value|}' \
  --output table

# List all RDS instances
aws rds describe-db-instances \
  --query 'DBInstances[].{ID:DBInstanceIdentifier,Engine:Engine,Status:DBInstanceStatus}' \
  --output table

# List all S3 buckets
aws s3api list-buckets --query 'Buckets[].{Name:Name,Created:CreationDate}' --output table

# Find untagged resources missing the Environment tag
aws resourcegroupstaggingapi get-resources \
  --tag-filters Key=Environment \
  --query 'ResourceTagMappingList[].ResourceARN' --output text
```

## Notes

A strong inventory improves control mapping, speeds incident response, and makes vulnerability remediation easier to prioritize. Tie it to your tagging standard so inventory and cost allocation stay in sync.
