# 11. 🔍 CloudTrail (Auditing & Governance)

**Record every API call made in your AWS account**

## What is CloudTrail?

AWS CloudTrail records all API calls (console, CLI, SDK, service-to-service) for auditing, compliance, and security analysis. Think of it as the **CCTV of your AWS account**.

### Key Benefits
- ✅ **Governance** — Track who did what, when, where
- ✅ **Compliance** — Meet regulatory requirements
- ✅ **Operational auditing** — Troubleshoot changes
- ✅ **Security analysis** — Detect unauthorized access
- ✅ **Risk auditing** — Identify risky actions
- ✅ **Enabled by default** — 90-day event history free

---

## What CloudTrail Logs

### 1. Management Events (Control Plane)
Operations that **manage** AWS resources (logged by default):

| Action | Example |
|--------|---------|
| Create resource | `RunInstances`, `CreateBucket` |
| Modify resource | `ModifyDBInstance`, `PutBucketPolicy` |
| Delete resource | `TerminateInstances`, `DeleteBucket` |
| IAM changes | `CreateUser`, `AttachRolePolicy` |
| Login events | `ConsoleLogin`, `AssumeRole` |

### 2. Data Events (Data Plane)
Operations on **data within** resources (optional, extra cost):

| Service | Events |
|---------|--------|
| S3 | `GetObject`, `PutObject`, `DeleteObject` |
| Lambda | `Invoke` |
| DynamoDB | `GetItem`, `PutItem`, `DeleteItem` |

### 3. Insights Events
**Anomaly detection** for unusual API activity (optional, extra cost):

- Detects unusual volume of write management APIs
- Detects changes in error rate patterns
- Example: Sudden spike in `RunInstances` calls at 3 AM

---

## CloudTrail Event Structure

Each event record contains:

```json
{
  "eventVersion": "1.08",
  "userIdentity": {
    "type": "IAMUser",
    "userName": "developer-john",
    "arn": "arn:aws:iam::123456789012:user/developer-john",
    "accountId": "123456789012"
  },
  "eventTime": "2026-02-14T10:30:00Z",
  "eventSource": "ec2.amazonaws.com",
  "eventName": "TerminateInstances",
  "awsRegion": "us-east-1",
  "sourceIPAddress": "203.0.113.50",
  "requestParameters": {
    "instancesSet": {"items": [{"instanceId": "i-0abc123"}]}
  },
  "responseElements": {
    "instancesSet": {"items": [{"currentState": {"name": "shutting-down"}}]}
  }
}
```

**Key fields for investigation:**
- `userIdentity` — **Who** did it
- `eventName` — **What** they did
- `eventTime` — **When** they did it
- `sourceIPAddress` — **Where** they came from
- `requestParameters` — **What** resources were affected

---

## Trail Configuration

### Default (Event History)
- Free, automatic, every account
- 90-day retention
- Management events only
- View in CloudTrail console
- Not delivered to S3

### Custom Trail
- Deliver to S3 bucket (long-term storage)
- Optional: Send to CloudWatch Logs (real-time)
- Enable Data Events and Insights
- Multi-region or single-region
- Organization trail (all accounts)

```bash
# Create multi-region trail with S3 delivery
aws cloudtrail create-trail \
  --name my-org-trail \
  --s3-bucket-name my-cloudtrail-logs-bucket \
  --is-multi-region-trail \
  --enable-log-file-validation \
  --kms-key-id alias/cloudtrail-key

# Start logging
aws cloudtrail start-logging --name my-org-trail

# Enable Insights
aws cloudtrail put-insight-selectors \
  --trail-name my-org-trail \
  --insight-selectors '[{"InsightType":"ApiCallRateInsight"},{"InsightType":"ApiErrorRateInsight"}]'

# Enable S3 data events
aws cloudtrail put-event-selectors \
  --trail-name my-org-trail \
  --event-selectors '[{
    "ReadWriteType": "All",
    "DataResources": [{
      "Type": "AWS::S3::Object",
      "Values": ["arn:aws:s3:::sensitive-bucket/"]
    }]
  }]'
```

---

## CloudTrail vs CloudWatch

| Feature | CloudTrail | CloudWatch |
|---------|-----------|------------|
| **Purpose** | API auditing ("who did what") | Performance monitoring ("how is it running") |
| **Data** | API call records | Metrics, logs, events |
| **Use Case** | Security, compliance, change tracking | Operations, troubleshooting, alerting |
| **Default** | 90-day event history | Basic metrics (5-min intervals) |
| **Question answered** | "Who deleted that EC2 instance?" | "Why is CPU at 95%?" |

**They work together:** CloudTrail logs → CloudWatch Logs → CloudWatch Alarm → SNS notification

---

## ✅ Hands-on Lab: CloudTrail Configuration

### Lab 1: Explore Event History

1. **CloudTrail Console** → **Event history**
2. Filter by event name: `RunInstances`
3. View who launched EC2 instances, when, and from where
4. Filter by user name to track specific user activity
5. Download events as CSV for offline analysis

### Lab 2: Create Custom Trail

**Step 1: Create S3 Bucket for Logs**
1. S3 → Create bucket: `my-cloudtrail-logs-[account-id]`
2. Enable versioning
3. Enable default encryption (SSE-S3 or SSE-KMS)
4. Block public access

**Step 2: Create Trail**
1. CloudTrail → **Trails** → **Create trail**
2. Trail name: `management-audit-trail`
3. Storage: Choose your S3 bucket
4. Log file SSE-KMS encryption: Enable
5. Log file validation: Enable
6. Multi-region: Yes
7. Management events: Read + Write
8. **Create trail**

**Step 3: Verify Logs**
1. Perform some actions (launch EC2, create S3 bucket)
2. Wait 5-15 minutes
3. Check S3 bucket for log files
4. Download and inspect JSON log entries

### Lab 3: CloudTrail + CloudWatch Alarms

**Detect unauthorized API calls:**

1. Create trail → Enable CloudWatch Logs integration
2. CloudWatch → Log Groups → Find CloudTrail log group
3. Create **Metric Filter**:
   - Pattern: `{ $.errorCode = "UnauthorizedAccess" }`
   - Metric: `UnauthorizedAPICalls`
4. Create **Alarm** on metric > 0
5. Action: SNS notification to security team

**Detect root account usage:**
- Filter pattern: `{ $.userIdentity.type = "Root" && $.userIdentity.invokedBy NOT EXISTS && $.eventType != "AwsServiceEvent" }`

**Detect console login without MFA:**
- Filter pattern: `{ $.eventName = "ConsoleLogin" && $.additionalEventData.MFAUsed != "Yes" }`

### Lab 4: Query with Athena

1. CloudTrail → **Event history** → **Create Athena table**
2. Select your trail's S3 bucket
3. Run queries:
```sql
-- Find all actions by a specific user
SELECT eventtime, eventname, sourceipaddress
FROM cloudtrail_logs
WHERE useridentity.username = 'developer-john'
ORDER BY eventtime DESC
LIMIT 50;

-- Find all security group changes
SELECT eventtime, useridentity.username, eventname, requestparameters
FROM cloudtrail_logs
WHERE eventsource = 'ec2.amazonaws.com'
  AND eventname LIKE '%SecurityGroup%'
ORDER BY eventtime DESC;
```

---

## CloudTrail Best Practices

1. ✅ **Enable in all regions** — Catch activity everywhere
2. ✅ **Enable log file validation** — Detect log tampering
3. ✅ **Encrypt logs with KMS** — Protect sensitive data
4. ✅ **Central logging account** — Aggregate across accounts
5. ✅ **Enable S3 object-level logging** — For sensitive buckets
6. ✅ **Set up CloudWatch alarms** — Real-time security alerts
7. ✅ **Restrict trail bucket access** — Only security team
8. ✅ **Enable Insights** — Detect anomalies automatically
9. ✅ **Use Athena** — Query logs at scale for investigations
10. ✅ **Integrate with GuardDuty** — Automated threat detection

## Common Interview Questions

1. **Q: What does CloudTrail log by default?**
   - Management events (API calls that manage resources) for the last 90 days.

2. **Q: How to enable S3 object-level logging?**
   - Create a trail and enable Data Events for S3.

3. **Q: How to detect if someone logs in as root?**
   - CloudTrail metric filter for `userIdentity.type = "Root"` → CloudWatch alarm → SNS.

4. **Q: Can CloudTrail logs be tampered with?**
   - Enable log file validation (digest files) to detect tampering. Store in a separate account's S3 bucket.

5. **Q: CloudTrail vs AWS Config?**
   - CloudTrail: Records API calls (actions). "Who changed it?"
   - Config: Records resource configuration changes over time. "What changed?"

---

[← Previous: CloudWatch](10-CloudWatch.md) | [Back to Main Guide](README.md) | [Next: Lambda →](12-Lambda.md)
