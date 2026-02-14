# 17. 💰 Cost Optimization

**Maximize value while minimizing waste on AWS**

## Why Cost Optimization Matters

Cloud spending can spiral quickly without discipline. Cost optimization isn't about spending less — it's about spending **smart**. The Well-Architected Framework lists it as one of the 5 pillars.

---

## Cost Optimization Strategies

### 1. Right-Sizing

Match resource sizes to actual workload requirements.

| Signal | Action |
|--------|--------|
| CPU consistently < 20% | Downsize instance type |
| Memory consistently < 30% | Switch to compute-optimized |
| Idle instances | Stop or terminate |
| Over-provisioned EBS | Resize volume |

**Tools:**
- **AWS Compute Optimizer** — ML-based recommendations for EC2, EBS, Lambda
- **CloudWatch metrics** — Manual analysis of utilization
- **Cost Explorer** — Right-sizing recommendations

```bash
# Get Compute Optimizer recommendations
aws compute-optimizer get-ec2-instance-recommendations \
  --instance-arns arn:aws:ec2:us-east-1:123456789012:instance/i-1234567890abcdef0
```

### 2. Purchasing Options

| Option | Savings | Commitment | Best For |
|--------|---------|------------|----------|
| **On-Demand** | 0% | None | Unpredictable, short-term |
| **Reserved Instances** | Up to 72% | 1 or 3 years | Steady-state workloads |
| **Savings Plans** | Up to 72% | 1 or 3 years ($/hour) | Flexible compute commitment |
| **Spot Instances** | Up to 90% | None (can interrupt) | Fault-tolerant, batch jobs |
| **Dedicated Hosts** | Varies | 1 or 3 years | Licensing, compliance |

**Reserved Instances — Payment Options:**
| Payment | Discount |
|---------|----------|
| All upfront | Highest (~40-60%) |
| Partial upfront | Medium (~30-50%) |
| No upfront | Lowest (~20-30%) |

**Savings Plans Types:**
- **Compute SP** — Any EC2, Fargate, Lambda (most flexible)
- **EC2 Instance SP** — Specific instance family in a region (larger discount)

**Spot Instances Strategy:**
- Diversify across instance types and AZs
- Use Spot Fleet for optimal capacity
- Handle interruptions gracefully (2-min warning)
- Great for: CI/CD builds, data processing, batch jobs, dev/test

### 3. Storage Optimization

| Strategy | Savings |
|----------|---------|
| S3 Lifecycle policies | Move to cheaper classes automatically |
| S3 Intelligent-Tiering | Auto-optimize unknown access patterns |
| Delete old EBS snapshots | Eliminate unused storage costs |
| Use gp3 instead of gp2 | 20% cheaper, better performance |
| Delete unattached EBS volumes | $0.10/GB/month wasted |
| Compress data before storing | Reduce GB stored |
| S3 multipart upload cleanup | Delete incomplete uploads |

**S3 Lifecycle Example:**
```json
{
  "Rules": [{
    "Id": "OptimizeCosts",
    "Status": "Enabled",
    "Transitions": [
      {"Days": 30, "StorageClass": "STANDARD_IA"},
      {"Days": 90, "StorageClass": "GLACIER_IR"},
      {"Days": 365, "StorageClass": "DEEP_ARCHIVE"}
    ],
    "Expiration": {"Days": 2555},
    "AbortIncompleteMultipartUpload": {"DaysAfterInitiation": 7}
  }]
}
```

### 4. Compute Optimization

| Strategy | Details |
|----------|---------|
| **Auto Scaling** | Scale down during low demand |
| **Lambda** | Pay only for execution time (no idle cost) |
| **Fargate** | No idle EC2 instances |
| **Scheduled scaling** | Shut down dev/test at night and weekends |
| **Graviton instances** | 20% cheaper, 40% better price-performance |
| **Containerize** | Higher density, better utilization |

### 5. Data Transfer Optimization

| Strategy | Savings |
|----------|---------|
| **VPC Endpoints** | S3/DynamoDB access without NAT Gateway |
| **CloudFront** | Cache at edge, reduce origin requests |
| **Same-region** | Inter-AZ transfer is cheaper than cross-region |
| **Compress responses** | Reduce bytes transferred |
| **Direct Connect** | Lower per-GB rates for high volume |
| **PrivateLink** | Avoid NAT Gateway charges |

**NAT Gateway can be expensive:** $0.045/hour + $0.045/GB processed. Use VPC endpoints for S3 and DynamoDB (free).

### 6. Database Optimization

| Strategy | Details |
|----------|---------|
| Reserved Instances for RDS | Same 1/3-year commitment savings |
| Aurora Serverless | Pay per ACU-second (no idle cost) |
| DynamoDB on-demand → provisioned | When patterns are predictable |
| Read replicas | Offload reads, right-size primary |
| ElastiCache | Reduce expensive DB queries |
| Delete old snapshots | Manual snapshots persist until deleted |

---

## Cost Management Tools

### AWS Cost Explorer

Visualize, filter, and forecast spending.

**Key Features:**
- View cost by service, account, tag, region
- 12-month historical data
- 12-month cost forecast
- Right-sizing recommendations
- Reserved Instance utilization tracking
- Savings Plans recommendations

### AWS Budgets

Set custom budgets and get alerts.

| Budget Type | What It Tracks |
|-------------|---------------|
| **Cost budget** | Total spending |
| **Usage budget** | Service usage (hours, GB) |
| **Savings Plans** | SP coverage and utilization |
| **Reservation** | RI coverage and utilization |

**Budget Actions:**
- Send notification (SNS, email)
- Auto-apply IAM policy to restrict launches
- Auto-apply SCP (Service Control Policy)

```bash
# Create cost budget with alert
aws budgets create-budget \
  --account-id 123456789012 \
  --budget '{
    "BudgetName": "Monthly-Budget",
    "BudgetLimit": {"Amount": "100", "Unit": "USD"},
    "TimeUnit": "MONTHLY",
    "BudgetType": "COST"
  }' \
  --notifications-with-subscribers '[{
    "Notification": {
      "NotificationType": "ACTUAL",
      "ComparisonOperator": "GREATER_THAN",
      "Threshold": 80,
      "ThresholdType": "PERCENTAGE"
    },
    "Subscribers": [{"SubscriptionType": "EMAIL", "Address": "you@example.com"}]
  }]'
```

### AWS Trusted Advisor

Automated check for cost, security, performance, fault tolerance.

**Free Tier Checks:**
- S3 bucket permissions
- Security groups (unrestricted access)
- IAM use
- MFA on root account
- Service limits

**Business/Enterprise Support Checks:**
- Idle EC2 instances
- Underutilized EBS volumes
- Unassociated Elastic IPs
- Idle load balancers
- Reserved Instance optimization
- Low-utilization RDS instances

### Cost Allocation Tags

Tag resources for cost tracking and reporting.

```bash
# Tag resources consistently
aws ec2 create-tags \
  --resources i-1234567890abcdef0 \
  --tags \
    Key=Project,Value=WebApp \
    Key=Environment,Value=Production \
    Key=Team,Value=Engineering \
    Key=CostCenter,Value=CC-1234

# Activate tags for cost allocation
# Billing Console → Cost Allocation Tags → Activate
```

**Recommended Tags:**
| Tag | Purpose |
|-----|---------|
| `Project` | Which project |
| `Environment` | dev / staging / prod |
| `Team` | Owning team |
| `CostCenter` | Finance tracking |
| `Owner` | Who to contact |
| `AutoShutdown` | Eligible for scheduled stop |

---

## ✅ Hands-on Lab: Cost Optimization

### Lab 1: Set Up Budget & Alerts

1. **Billing Console** → **Budgets** → **Create budget**
2. Template: Monthly cost budget
3. Budget name: `Monthly-$50-Budget`
4. Amount: $50
5. Email: Your email address
6. Alert thresholds: 50%, 80%, 100%
7. **Create budget**

### Lab 2: Find Waste with Cost Explorer

1. **Cost Explorer** → Enable (if first time)
2. Group by: Service → Identify top cost drivers
3. Filter: Last 3 months → Spot trends
4. Check **Recommendations** tab:
   - Right-sizing recommendations
   - Savings Plans recommendations
   - Reserved Instance recommendations

### Lab 3: Clean Up Unused Resources

```bash
# Find unattached EBS volumes
aws ec2 describe-volumes \
  --filters Name=status,Values=available \
  --query 'Volumes[*].[VolumeId,Size,CreateTime]' \
  --output table

# Find unassociated Elastic IPs (charged when unused!)
aws ec2 describe-addresses \
  --query 'Addresses[?AssociationId==null].[PublicIp,AllocationId]' \
  --output table

# Find idle load balancers
aws elbv2 describe-target-health \
  --target-group-arn <arn> \
  --query 'TargetHealthDescriptions[*].[Target.Id,TargetHealth.State]'

# Find old snapshots (>90 days)
aws ec2 describe-snapshots --owner-ids self \
  --query 'Snapshots[?StartTime<=`2025-11-14`].[SnapshotId,VolumeSize,StartTime]' \
  --output table
```

### Lab 4: Schedule Dev Instance Stop/Start

```bash
# Use EventBridge + Lambda to stop instances at 6 PM
# and start at 8 AM (save ~60% on dev environments)

# Tag instances: AutoShutdown=true
# Lambda function: stop/start based on tag
```

---

## Quick Wins Checklist

- [ ] Delete unused EC2 instances, EBS volumes, and Elastic IPs
- [ ] Stop non-production instances overnight and on weekends
- [ ] Enable S3 Lifecycle policies for all log buckets
- [ ] Switch EBS from gp2 to gp3 (cheaper, faster)
- [ ] Buy Reserved Instances/Savings Plans for steady workloads
- [ ] Enable Auto Scaling with scale-to-zero for dev
- [ ] Use VPC Endpoints instead of NAT Gateway for S3/DynamoDB
- [ ] Review and delete old EBS/RDS snapshots
- [ ] Use Graviton instances where possible
- [ ] Set billing alerts at multiple thresholds
- [ ] Tag every resource for cost allocation
- [ ] Review Trusted Advisor recommendations monthly

---

## Common Interview Questions

1. **Q: Name 5 ways to reduce AWS costs**
   - Right-sizing, Reserved Instances, Spot Instances, S3 Lifecycle, Auto Scaling, delete unused resources.

2. **Q: Reserved Instances vs Savings Plans?**
   - RI: Specific instance type/region. SP: Flexible (compute or EC2), committed $/hour.

3. **Q: How to track costs per project?**
   - Cost Allocation Tags → Activate in Billing → Use Cost Explorer to filter by tag.

4. **Q: What is Spot Instance and when to use it?**
   - Up to 90% discount, can be interrupted with 2-min warning. Use for fault-tolerant/batch workloads.

5. **Q: How to avoid unexpected high bills?**
   - Set AWS Budgets with alerts, enable Billing Alarms, use SCPs to restrict expensive services, review Cost Explorer weekly.

---

[← Previous: Security & Compliance](16-Security-Compliance.md) | [Back to Main Guide](README.md) | [Next: Well-Architected Framework →](18-Well-Architected-Framework.md)
