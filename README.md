# Cloud Security Baseline

![AWS](https://img.shields.io/badge/AWS-Security-FF9900?style=flat&logo=amazon-aws&logoColor=white)
![Security+](https://img.shields.io/badge/CompTIA-Security%2B-E02020?style=flat&logo=comptia&logoColor=white)
![CIS](https://img.shields.io/badge/CIS-Benchmarks-003366?style=flat&logoColor=white)
![Status](https://img.shields.io/badge/status-active-brightgreen?style=flat)

Enterprise AWS cloud security baseline covering IAM least-privilege design, CIS Benchmark controls, logging and monitoring standards, and incident response runbooks. Built by a **CompTIA Security+** certified engineer and AWS Solutions Architect with enterprise infrastructure and US Navy operational experience.

> Designed to meet the security expectations of GovCon environments, enterprise cloud teams, and compliance-aware organizations.

---

## Framework Modules

| Module | Description | Status |
|---|---|---|
| [IAM Least-Privilege Design](./iam/) | Role structure, permission boundaries, policy templates, MFA enforcement | ✅ Complete |
| [Logging & Monitoring Standards](./logging/) | CloudTrail, CloudWatch Logs, VPC Flow Logs, Security Hub, Config | ✅ Complete |
| [Incident Response Runbooks](./incident-response/) | Network outage, unauthorized access, cost anomaly, data exposure playbooks | ✅ Complete |
| [CIS AWS Benchmark Controls](./cis-controls/) | CIS Level 1 & 2 control implementation guide for AWS | 🔵 In Progress |
| [Linux Hardening Baseline](./linux-hardening/) | CIS-aligned Linux host hardening for EC2 instances | 📝 Planned |

---

## Engineering Principles

- **Least Privilege** — No principal has more access than required. Policies scoped to specific resources, not wildcards.
- **Defense in Depth** — Security controls layered across IAM, network, logging, and detection.
- **Security by Design** — Security controls applied at provisioning time, not retrofitted.
- **Continuous Validation** — AWS Config rules and Security Hub findings reviewed on a defined cadence.
- **Documentation-Driven** — Every control has a documented rationale, implementation step, and verification test.

---

## Repository Structure

```
cloud-security-baseline/
├── iam/
│   ├── iam-design-principles.md      # Least-privilege design guide
│   ├── role-structure.md             # Account role hierarchy
│   ├── policies/
│   │   ├── finops-readonly.json       # Read-only Cost Explorer access
│   │   ├── ec2-operator.json          # EC2 operator (no IAM changes)
│   │   └── security-auditor.json      # Security audit read-only
│   └── scripts/
│       ├── audit-iam.sh               # List overprivileged roles
│       └── enforce-mfa.sh             # Enforce MFA for console users
├── logging/
│   ├── logging-standards.md          # Required logs and retention policy
│   └── scripts/
│       ├── enable-cloudtrail.sh       # Multi-region CloudTrail setup
│       ├── enable-vpc-flow-logs.sh    # VPC Flow Logs to CloudWatch
│       └── enable-config.sh           # AWS Config recorder setup
├── incident-response/
│   ├── IR-unauthorized-access.md     # Unauthorized access playbook
│   ├── IR-network-outage.md          # Network outage runbook
│   ├── IR-cost-anomaly.md            # Unexpected cost spike runbook
│   └── IR-data-exposure.md           # Accidental public S3 / data exposure
└── cis-controls/
    └── cis-aws-level1-checklist.md   # CIS AWS Level 1 control checklist
```

---

## Quick Reference — IAM Audit

```bash
# Find users with no MFA enrolled
aws iam generate-credential-report
aws iam get-credential-report --query Content --output text | base64 -d | \
  awk -F, 'NR>1 && $4=="true" && $8=="false" {print $1, "- Console access, NO MFA"}'

# Find roles with wildcard actions (overprivileged)
aws iam list-roles --query 'Roles[].RoleName' --output text | tr '\t' '\n' | \
  while read -r role; do
    aws iam list-role-policies --role-name "$role" --output text 2>/dev/null | \
    grep -q . && echo "Inline policy on: $role"
  done

# Check for unused access keys (> 90 days)
aws iam generate-credential-report
aws iam get-credential-report --query Content --output text | base64 -d | \
  awk -F, 'NR>1 {print $1, $10, $14}'
```

---

## Quick Reference — Logging Verification

```bash
# Verify CloudTrail is active and multi-region
aws cloudtrail describe-trails --query 'trailList[].{Name:Name,IsMultiRegion:IsMultiRegionTrail,LogStatus:HasCustomEventSelectors}'

# Verify VPC Flow Logs are enabled
aws ec2 describe-flow-logs --query 'FlowLogs[].{VPC:ResourceId,Status:FlowLogStatus,Dest:LogDestinationType}'

# Check AWS Config recorder status
aws configservice describe-configuration-recorder-status \
  --query 'ConfigurationRecordersStatus[].{Name:name,Recording:recording,LastStatus:lastStatus}'
```

---

## Related Projects

- [aws-cloud-labs](https://github.com/oumoeurtmm-code/aws-cloud-labs) — AWS infrastructure labs with security controls embedded
- [finops-cloud-governance](https://github.com/oumoeurtmm-code/finops-cloud-governance) — Cloud cost governance framework

---

<div align="center">
  <sub>Built with discipline · Powered by cloud · Secured by default</sub>
</div>
