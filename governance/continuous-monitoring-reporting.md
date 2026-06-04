# Continuous Monitoring Reporting

Continuous monitoring (ConMon) reporting tracks the health of implemented controls over time and surfaces drift, findings, and remediation progress.

## Typical Inputs

- Vulnerability scan results
- Cloud security findings (Security Hub, GuardDuty, Config)
- Logging and monitoring coverage status
- IAM and access review results
- POA&M status updates
- Change activity affecting control posture

## Monthly Reporting Structure

| Section | Purpose |
|---|---|
| Executive Summary | High-level health status and major issues |
| Control Status | Open issues and control drift |
| Vulnerability Summary | Findings by severity and owner |
| Access Review Summary | Quarterly or ad hoc access results |
| Logging / Monitoring Status | Coverage and identified gaps |
| POA&M Summary | Open vs closed items and aging |
| Risks / Escalations | Issues requiring leadership attention |

## AWS Tools Supporting ConMon

| Tool | What It Monitors |
|---|---|
| AWS Security Hub | Aggregated security findings across services |
| Amazon GuardDuty | Threat detection — unusual API calls, network activity |
| AWS Config | Resource configuration drift and compliance rules |
| AWS CloudTrail | API activity logging and audit trail |
| Amazon Inspector | EC2 and container vulnerability assessment |

## Goal

The purpose of continuous monitoring is not only to collect findings — it is to demonstrate that controls remain effective and that identified issues are being actively tracked to closure.
