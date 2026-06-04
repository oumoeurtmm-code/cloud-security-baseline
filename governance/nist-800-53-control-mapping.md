# NIST 800-53 Control Mapping

This document outlines a practical approach for mapping cloud security controls and operational evidence to NIST SP 800-53 control families.

## Objective

The goal of control mapping is to show how implemented safeguards, processes, and evidence align to required controls in a structured, reviewable way.

## Core Control Families Often Touched in Cloud Environments

- AC — Access Control
- AU — Audit and Accountability
- CA — Assessment, Authorization, and Monitoring
- CM — Configuration Management
- CP — Contingency Planning
- IA — Identification and Authentication
- IR — Incident Response
- RA — Risk Assessment
- SC — System and Communications Protection
- SI — System and Information Integrity

## Mapping Approach

For each control:
1. Define the control requirement
2. Identify the implemented technical or procedural safeguard
3. Identify the source of evidence
4. Document ownership and review cadence
5. Record gaps or planned improvements

## Example Mapping Format

| Control | Requirement | Implementation | Evidence | Owner | Review Cadence |
|---|---|---|---|---|---|
| AC-2 | Account management | IAM user/role review process | Access review record | Security / IAM | Quarterly |
| AU-2 | Event logging | CloudTrail + CloudWatch logging baseline | Logging configuration export | Cloud Team | Monthly |
| CA-7 | Continuous monitoring | Security findings review + reporting cadence | Monthly ConMon report | Security Ops | Monthly |
| IA-2 | User identification | MFA enforced for all console users | IAM credential report | IAM Admin | Monthly |
| IR-4 | Incident handling | Documented incident response runbooks | IR runbook + exercise log | Security Ops | Quarterly |

## Notes

Control mapping is strongest when it connects technical configurations to repeatable operational processes and retained evidence.
