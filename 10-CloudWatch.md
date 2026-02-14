# 10. 📈 CloudWatch (Monitoring & Observability)

**Monitor AWS resources and applications in real-time**

## What is CloudWatch?

Amazon CloudWatch collects monitoring and operational data in the form of metrics, logs, and events — giving you a unified view of AWS resources, applications, and services.

### Key Benefits
- ✅ **Unified monitoring** — Single pane for all resources
- ✅ **Automated actions** — React to threshold breaches
- ✅ **Operational visibility** — Dashboards, logs, traces
- ✅ **Cost-effective** — Free tier for basic monitoring
- ✅ **Integration** — Works with 70+ AWS services

---

## CloudWatch Components

### 1. Metrics

Pre-defined or custom performance data points.

**Default EC2 Metrics** (5-minute intervals, free):
| Metric | Description |
|--------|-------------|
| CPUUtilization | Percentage of CPU used |
| NetworkIn / NetworkOut | Bytes of network traffic |
| DiskReadOps / DiskWriteOps | I/O operations (instance store only) |
| StatusCheckFailed | Instance or system status |
| CPUCreditBalance | Burstable instance credits (T-series) |

**NOT collected by default** (need CloudWatch Agent):
- Memory utilization
- Disk space / disk usage
- Number of processes
- Custom application metrics

**Detailed Monitoring**: 1-minute intervals (extra cost, ~$3.50/instance/month)

**Metric Retention:**
| Resolution | Retention |
|------------|-----------|
| 1-second (high resolution) | 3 hours |
| 60-second | 15 days |
| 5-minute | 63 days |
| 1-hour | 455 days (15 months) |

### 2. Logs

Collect, monitor, and store log files from resources.

**Log Structure:**
```
Log Groups (container, e.g., /aws/lambda/my-function)
    ↓
Log Streams (individual sources, e.g., per instance)
    ↓
Log Events (individual log entries with timestamp)
```

**Common Log Sources:**
- EC2 instances (via CloudWatch Agent)
- Lambda functions (automatic)
- API Gateway (access logs)
- VPC Flow Logs
- CloudTrail logs
- RDS / Aurora logs
- ECS / EKS container logs

**Log Insights** — Query logs with SQL-like syntax:
```sql
-- Find top 10 errors in the last hour
fields @timestamp, @message
| filter @message like /ERROR/
| sort @timestamp desc
| limit 10

-- Count HTTP status codes
fields @message
| parse @message "HTTP/1.1\" * " as statusCode
| stats count(*) by statusCode

-- P95 latency over time
filter @type = "REPORT"
| stats percentile(@duration, 95) as p95 by bin(5m)
```

**Log Retention:**
- Configurable: 1 day to 10 years, or never expire
- Default: Never expire (costs accumulate!)
- Set retention policies to control costs

**Export / Stream:**
- Export to S3 (batch, hours delay)
- Stream to Lambda, OpenSearch, Kinesis (real-time)
- Subscription filters for targeted streaming

### 3. Alarms

Automatically perform actions based on metric thresholds.

**Alarm States:**
| State | Meaning |
|-------|---------|
| `OK` | Metric is within threshold |
| `ALARM` | Metric has breached threshold |
| `INSUFFICIENT_DATA` | Not enough data to evaluate |

**Alarm Actions:**
- **SNS** — Send email/SMS notification
- **Auto Scaling** — Scale in/out
- **EC2** — Stop, terminate, reboot, recover instance
- **Systems Manager** — Run automation documents
- **Lambda** — Trigger custom remediation

**Composite Alarms:**
- Combine multiple alarms with AND/OR logic
- Reduce alarm noise
- Example: Only alert if BOTH CPU > 80% AND memory > 90%

**Example: CPU Alarm**
```bash
aws cloudwatch put-metric-alarm \
  --alarm-name cpu-high \
  --alarm-description "Alert when CPU exceeds 80%" \
  --metric-name CPUUtilization \
  --namespace AWS/EC2 \
  --statistic Average \
  --period 300 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 2 \
  --alarm-actions arn:aws:sns:region:account:my-topic \
  --dimensions Name=InstanceId,Value=i-1234567890abcdef0
```

### 4. Dashboards

Visual representation of metrics — custom, shareable, cross-region.

**Features:**
- Mix metrics from multiple AWS services
- Automatic refresh (10s, 1m, 5m, etc.)
- Full-screen mode for NOC displays
- Share via link or embed
- Pricing: $3/dashboard/month (after first 3 free)

### 5. EventBridge (formerly CloudWatch Events)

Event-driven automation and routing.

**Event Sources:**
- AWS service events (EC2 state change, S3 upload, etc.)
- Scheduled events (cron / rate)
- Custom application events
- SaaS partner events (Datadog, PagerDuty, etc.)

**Targets:**
- Lambda, Step Functions
- SQS, SNS, Kinesis
- EC2, ECS tasks
- CodePipeline, CodeBuild
- SSM Run Command

**Examples:**
```bash
# Trigger Lambda every 5 minutes
aws events put-rule \
  --name "every-5-minutes" \
  --schedule-expression "rate(5 minutes)"

# React to EC2 instance state changes
aws events put-rule \
  --name "ec2-state-change" \
  --event-pattern '{"source":["aws.ec2"],"detail-type":["EC2 Instance State-change Notification"]}'
```

---

## CloudWatch Agent

Install on EC2 to collect OS-level metrics and logs.

### What It Collects
- **Memory** utilization, available, used
- **Disk** space, I/O, inodes
- **Swap** utilization
- **Network** detailed stats
- **Processes** count, states
- **Custom log files** from applications

### Installation
```bash
# Install agent
sudo yum install -y amazon-cloudwatch-agent

# Run configuration wizard
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-config-wizard

# Start agent
sudo systemctl start amazon-cloudwatch-agent
sudo systemctl enable amazon-cloudwatch-agent
```

---

## Custom Metrics

Publish your own application metrics to CloudWatch.

```python
import boto3
from datetime import datetime

cloudwatch = boto3.client('cloudwatch')

cloudwatch.put_metric_data(
    Namespace='MyApp',
    MetricData=[
        {
            'MetricName': 'ActiveUsers',
            'Value': 150,
            'Unit': 'Count',
            'Timestamp': datetime.now(),
            'Dimensions': [
                {'Name': 'Environment', 'Value': 'Production'},
                {'Name': 'Region', 'Value': 'us-east-1'}
            ]
        },
        {
            'MetricName': 'OrderProcessingTime',
            'Value': 1.25,
            'Unit': 'Seconds',
            'Timestamp': datetime.now()
        }
    ]
)
```

**High-Resolution Metrics:**
- Standard: 60-second granularity
- High-resolution: 1-second granularity (`StorageResolution=1`)
- Useful for: real-time dashboards, fast auto-scaling

---

## ✅ Hands-on Lab: CloudWatch Monitoring

### Lab 1: Create Dashboard with Alarms

**Step 1: Create SNS Topic for Notifications**
1. SNS Console → **Create topic** → Standard
2. Name: `CloudWatch-Alerts`
3. Create subscription: Email → your email
4. Confirm subscription in your inbox

**Step 2: Create CloudWatch Alarm**
1. CloudWatch → **Alarms** → **Create alarm**
2. Select metric: EC2 → Per-Instance Metrics → CPUUtilization
3. Select your instance
4. Conditions:
   - Threshold: Greater than 70%
   - Evaluation period: 1 out of 1 (5 min)
5. Actions:
   - In alarm: Send to `CloudWatch-Alerts` topic
6. Name: `High-CPU-Alert`
7. **Create alarm**

**Step 3: Create Dashboard**
1. CloudWatch → **Dashboards** → **Create dashboard**
2. Name: `My-App-Dashboard`
3. Add widgets:
   - **Line chart**: EC2 CPUUtilization
   - **Number**: Current active alarms
   - **Logs table**: Recent error logs
   - **Stacked area**: Network In/Out
4. Save dashboard

**Step 4: Test Alarm**
```bash
# SSH into instance and generate CPU load
sudo yum install -y stress
stress --cpu 4 --timeout 300

# Watch alarm state change: OK → ALARM
# Check email for SNS notification
```

### Lab 2: CloudWatch Logs with Agent

1. Attach IAM role with `CloudWatchAgentServerPolicy` to EC2
2. Install and configure agent (see Installation above)
3. Configure to collect `/var/log/httpd/access_log`
4. View logs in CloudWatch → Log Groups
5. Create metric filter: count of "404" errors
6. Create alarm on 404 count > 10

### Lab 3: EventBridge Scheduled Rule

1. EventBridge → **Rules** → **Create rule**
2. Name: `daily-snapshot`
3. Schedule: `cron(0 3 * * ? *)` (3 AM daily)
4. Target: Lambda function (create snapshot)
5. **Create**

---

## CloudWatch Pricing Summary

| Feature | Free Tier | Paid |
|---------|-----------|------|
| Basic metrics | 5-min intervals | Detailed: ~$3.50/instance/month |
| Alarms | 10 alarms | $0.10/alarm/month |
| Dashboards | 3 dashboards | $3/dashboard/month |
| Logs — Ingestion | — | $0.50/GB |
| Logs — Storage | — | $0.03/GB/month |
| Custom metrics | — | $0.30/metric/month |
| Log Insights queries | — | $0.005/GB scanned |

---

## CloudWatch Best Practices

1. ✅ **Enable detailed monitoring** for production EC2 instances
2. ✅ **Install CloudWatch Agent** for memory/disk metrics
3. ✅ **Set log retention** policies to control costs
4. ✅ **Use composite alarms** to reduce noise
5. ✅ **Create dashboards** for operational visibility
6. ✅ **Use Logs Insights** for ad-hoc troubleshooting
7. ✅ **Export logs to S3** for long-term, cost-effective storage
8. ✅ **Tag metrics** with dimensions for filtering

## Common Interview Questions

1. **Q: What metrics does CloudWatch collect for EC2 by default?**
   - CPU, network, disk I/O, status checks. NOT memory or disk space (need Agent).

2. **Q: How to monitor memory utilization?**
   - Install CloudWatch Agent on the instance.

3. **Q: CloudWatch vs CloudTrail?**
   - CloudWatch: Performance monitoring (metrics, logs, alarms)
   - CloudTrail: API auditing (who did what, when)

4. **Q: What is a composite alarm?**
   - Combines multiple alarms with AND/OR logic to reduce noise.

5. **Q: How to reduce CloudWatch Logs costs?**
   - Set retention policies, export to S3, use subscription filters to send only relevant logs.

---

[← Previous: Auto Scaling & Load Balancing](09-Auto-Scaling-Load-Balancing.md) | [Back to Main Guide](README.md) | [Next: CloudTrail →](11-CloudTrail.md)
