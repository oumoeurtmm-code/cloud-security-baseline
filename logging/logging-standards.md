# Logging & Monitoring Standards

Logging is not optional. Without comprehensive logging, there is no detection, no forensics, and no compliance. This document defines the minimum required logging posture for AWS environments in this baseline.

---

## Required Logs

| Log Source | Service | Purpose | Retention |
|---|---|---|---|
| **API activity** | AWS CloudTrail | All API calls — who did what, when, from where | 365 days minimum |
| **Network traffic** | VPC Flow Logs | Accepted/rejected traffic on all VPCs | 90 days |
| **DNS queries** | Route 53 Resolver Logs | DNS lookups within VPC — useful for threat detection | 90 days |
| **S3 access** | S3 Server Access Logging | Object-level access on sensitive buckets | 90 days |
| **Config changes** | AWS Config | Resource configuration history and drift detection | 365 days |
| **Auth events** | CloudWatch Logs (from CloudTrail) | Console sign-ins, MFA failures, root activity | 365 days |
| **Security findings** | AWS Security Hub | Aggregated findings from GuardDuty, Inspector, Config | Ongoing |

---

## CloudTrail Requirements

- **Multi-region trail** — must cover all regions, not just home region
- **S3 data events** — enable for sensitive buckets (read + write)
- **Management events** — all Read + Write API calls
- **Log file validation** — enabled to detect tampering
- **CloudWatch Logs integration** — stream CloudTrail to CloudWatch for alerting

```bash
# Verify multi-region trail exists and is logging
aws cloudtrail get-trail-status --name <your-trail-name> \
  --query '{IsLogging:IsLogging,LastDelivery:LatestDeliveryTime}'

# Check log file validation
aws cloudtrail describe-trails \
  --query 'trailList[].{Name:Name,Validation:LogFileValidationEnabled,MultiRegion:IsMultiRegionTrail}'
```

---

## CloudWatch Metric Alarms (Required)

Create metric filters + alarms in CloudWatch for these high-signal events:

| Event | Why It Matters |
|---|---|
| Root account login | Root should never be used for routine operations |
| Console login without MFA | Credential exposure risk |
| IAM policy changes | Privilege escalation vector |
| Security Group changes | Network exposure risk |
| CloudTrail disabled or stopped | Attacker covering tracks |
| S3 bucket policy changes | Data exposure risk |
| Failed console login (>5 in 1 hour) | Brute force indicator |

---

## Log Retention Policy

| Classification | Minimum Retention | Storage Tier |
|---|---|---|
| Security events (CloudTrail, auth) | 365 days | S3 Standard → Glacier after 90 days |
| Network logs (VPC Flow Logs) | 90 days | CloudWatch Logs |
| Application logs | 30-90 days | CloudWatch Logs |
| Config history | 365 days | AWS Config managed |

---

## FinOps Note

Logging has a real cost. CloudWatch Logs ingestion and storage, S3 object storage, and CloudTrail data events all generate charges. Apply these controls to minimize cost without sacrificing coverage:

- Use **S3 lifecycle policies** to transition logs from Standard to Glacier after 90 days
- Enable **S3 Intelligent-Tiering** for Config snapshots
- Scope CloudTrail **data events** to specific high-value buckets, not all buckets
- Set **CloudWatch Log Group retention** to match your retention policy (not unlimited)
