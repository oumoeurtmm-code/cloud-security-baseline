# Incident Response Runbook — Unexpected Cost Spike

**Severity:** High (financial impact)  
**Trigger:** AWS Cost Anomaly Detection alert, budget threshold breach, or manual discovery  
**Owner:** FinOps / Cloud Platform Team  

---

## Detection Signals

- AWS Cost Anomaly Detection email alert
- Budget threshold alert (80% or 100% of monthly budget)
- Manual review of Cost Explorer showing unexpected spike
- Service quota alert (sometimes precedes cost spike)

---

## Immediate Assessment (First 30 minutes)

```bash
# Step 1: Identify which service is driving the anomaly
aws ce get-cost-and-usage \
  --time-period Start=$(date -d '-7 days' '+%Y-%m-%d'),End=$(date '+%Y-%m-%d') \
  --granularity DAILY \
  --metrics BlendedCost \
  --group-by Type=DIMENSION,Key=SERVICE \
  --query 'ResultsByTime[].Groups[?Metrics.BlendedCost.Amount>`10`].{Service:Keys[0],Cost:Metrics.BlendedCost.Amount}'

# Step 2: Check for active anomalies
aws ce get-anomalies \
  --date-interval Start=$(date -d '-7 days' '+%Y-%m-%d'),End=$(date '+%Y-%m-%d') \
  --query 'Anomalies[].{Service:RootCauses[0].Service,MaxImpact:Impact.MaxImpact,Status:AnomalyEndDate}'

# Step 3: Find top resource contributors (EC2 example)
aws ce get-cost-and-usage \
  --time-period Start=$(date -d '-3 days' '+%Y-%m-%d'),End=$(date '+%Y-%m-%d') \
  --granularity DAILY \
  --metrics BlendedCost \
  --group-by Type=TAG,Key=Project
```

---

## Common Root Causes & Fixes

| Cause | Detection | Fix |
|---|---|---|
| EC2 instances not terminated after lab/test | EC2 console, Cost Explorer by service | Terminate unused instances immediately |
| NAT Gateway left running | Cost Explorer — high Data Transfer charges | Delete NAT GW if VPC not in use |
| S3 data transfer (egress) spike | Cost Explorer — Data Transfer line item | Review S3 access logs, check for public exposure |
| RDS instance left running | RDS console | Stop or snapshot + delete |
| Lambda runaway loop | CloudWatch Logs for Lambda errors + invocations | Throttle or disable the function |
| Forgotten Elastic IP | EC2 → Elastic IPs | Release unattached EIPs |

---

## Containment — Stop the Bleeding

```bash
# Find and stop all running EC2 instances in non-prod environments
aws ec2 describe-instances \
  --filters Name=tag:Environment,Values=learning,dev,test Name=instance-state-name,Values=running \
  --query 'Reservations[].Instances[].InstanceId' --output text | \
  tr '\t' '\n' | while read -r id; do
    echo "Stopping: $id"
    aws ec2 stop-instances --instance-ids "$id"
  done

# Find unattached EIPs (cost money even when idle)
aws ec2 describe-addresses \
  --query 'Addresses[?AssociationId==null].{IP:PublicIp,AllocationId:AllocationId}'
```

---

## Post-Incident

- [ ] Document root cause and exact dollar impact
- [ ] Add cleanup to the project that caused the spike
- [ ] Lower anomaly detection threshold if needed
- [ ] Add tagging to untagged resources that made root cause harder to find
- [ ] Schedule a cost review for the following week
