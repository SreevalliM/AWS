# 10. 📈 CloudWatch (Monitoring)

Amazon CloudWatch monitors AWS resources and applications in real-time.

## CloudWatch Components

### 1. Metrics
Pre-defined or custom performance data points.

**Default EC2 Metrics** (5-minute intervals):
- CPUUtilization
- NetworkIn/NetworkOut
- DiskReadOps/DiskWriteOps
- StatusCheckFailed

**Detailed Monitoring**: 1-minute intervals (extra cost)

### 2. Logs
Collect and store log files from resources.

**Log Structure**:
```
Log Groups
    ↓
Log Streams (individual sources)
    ↓
Log Events (individual log entries)
```

**Example: Query Logs**
```bash
# Create log group
aws logs create-log-group --log-group-name /aws/lambda/my-function

# Query logs
aws logs filter-log-events \
  --log-group-name /aws/lambda/my-function \
  --filter-pattern "ERROR"
```

### 3. Alarms
Automatically perform actions based on metric thresholds.

**Alarm States**:
- **OK**: Within threshold
- **ALARM**: Breached threshold
- **INSUFFICIENT_DATA**: Not enough data

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
  --alarm-actions arn:aws:sns:region:account:my-topic
```

### 4. Dashboards
Visual representation of metrics.

### 5. Events (EventBridge)
Event-driven automation.

**Example Use Cases**:
- Schedule Lambda functions (cron)
- React to AWS service events
- Route events to SQS, SNS, Lambda

### Custom Metrics
```python
import boto3
from datetime import datetime

cloudwatch = boto3.client('cloudwatch')

cloudwatch.put_metric_data(
    Namespace='MyApp',
    MetricData=[{
        'MetricName': 'ActiveUsers',
        'Value': 150,
        'Unit': 'Count',
        'Timestamp': datetime.now()
    }]
)
```

---

[← Previous: Auto Scaling & Load Balancing](09-Auto-Scaling-Load-Balancing.md) | [Back to Main Guide](README.md) | [Next: CloudTrail →](11-CloudTrail.md)
