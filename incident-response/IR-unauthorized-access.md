# Incident Response Runbook — Unauthorized Access

**Severity:** Critical  
**Trigger:** GuardDuty finding, CloudTrail anomaly, or direct report of unauthorized console/API access  
**Owner:** Security Operations / Cloud Platform Team  

---

## Detection Signals

- GuardDuty finding: `UnauthorizedAccess:IAMUser/ConsoleLogin` or `Persistence:IAMUser/UserPermissions`
- CloudTrail: Console login from unexpected IP/region
- AWS Security Hub: High-severity IAM finding
- User report: "I didn't do that action"

---

## Containment (First 15 minutes)

```bash
# Step 1: Identify the affected principal
aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=Username,AttributeValue=<USERNAME> \
  --start-time $(date -d '-24 hours' --iso-8601=seconds) \
  --query 'Events[].{Time:EventTime,Event:EventName,IP:CloudTrailEvent}'

# Step 2: Disable the compromised IAM user immediately
aws iam update-login-profile --user-name <USERNAME> --no-password-reset-required
aws iam delete-login-profile --user-name <USERNAME>  # Disables console access

# Step 3: Deactivate all access keys for the user
aws iam list-access-keys --user-name <USERNAME> --query 'AccessKeyMetadata[].AccessKeyId' --output text | \
  tr '\t' '\n' | while read -r key; do
    aws iam update-access-key --user-name <USERNAME> --access-key-id "$key" --status Inactive
    echo "Deactivated key: $key"
  done

# Step 4: Revoke all active sessions
aws iam attach-user-policy \
  --user-name <USERNAME> \
  --policy-arn arn:aws:iam::aws:policy/AWSDenyAll
```

---

## Investigation

```bash
# What did they do? Pull all API calls for the past 24 hours
aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=Username,AttributeValue=<USERNAME> \
  --start-time $(date -d '-24 hours' --iso-8601=seconds) \
  --query 'Events[].{Time:EventTime,Event:EventName,Region:CloudTrailEvent}' \
  --output table

# Were any new IAM users or roles created?
aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=EventName,AttributeValue=CreateUser \
  --start-time $(date -d '-24 hours' --iso-8601=seconds)

aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=EventName,AttributeValue=CreateRole \
  --start-time $(date -d '-24 hours' --iso-8601=seconds)

# Were any policies attached?
aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=EventName,AttributeValue=AttachUserPolicy \
  --start-time $(date -d '-24 hours' --iso-8601=seconds)
```

---

## Eradication & Recovery

1. Remove any backdoor IAM users, roles, or policies created during the incident
2. Rotate all access keys for the affected principal and any principals that interacted with them
3. Force MFA re-enrollment if the account will be re-activated
4. Review and tighten the IAM permissions that allowed the initial access
5. Enable GuardDuty threat intel feeds if not already active

---

## Communication Template

```
SEVERITY: Critical
INCIDENT: Unauthorized AWS Console/API Access
DETECTED: [timestamp]
AFFECTED PRINCIPAL: [IAM username or role]
CONTAINMENT STATUS: Access keys deactivated / Console access removed at [timestamp]
ACTIONS TAKEN: [list]
NEXT STEPS: Investigation ongoing — update in [30/60] minutes
POINT OF CONTACT: [name]
```

---

## Post-Incident

- [ ] Document the full incident timeline
- [ ] Identify root cause (phishing, leaked key, misconfigured policy)
- [ ] File findings in ticketing system
- [ ] Review detection gap — why wasn't this caught earlier?
- [ ] Update monitoring rules to catch this pattern faster next time
