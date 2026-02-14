# 11. 🔍 CloudTrail (Auditing)

AWS CloudTrail records AWS API calls for auditing and compliance.

## Key Features
- ✅ **Governance** and **compliance**
- ✅ **Operational auditing**
- ✅ **Risk auditing**
- ✅ **API call history**
- ✅ **Who, what, when, where**

## What CloudTrail Logs
- **Management Events**: Control plane operations
  - Creating EC2 instance
  - Deleting S3 bucket
  - Creating IAM user
- **Data Events**: Data plane operations (optional, extra cost)
  - S3 object-level operations
  - Lambda function invocations
- **Insights Events**: Unusual API activity (extra cost)

## Trail Configuration
```bash
# Create trail
aws cloudtrail create-trail \
  --name my-trail \
  --s3-bucket-name my-cloudtrail-bucket \
  --is-multi-region-trail

# Start logging
aws cloudtrail start-logging --name my-trail

# Query recent events
aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=EventName,AttributeValue=DescribeInstances \
  --max-results 10
```

## CloudTrail vs CloudWatch

| Feature | CloudTrail | CloudWatch |
|---------|-----------|------------|
| **Purpose** | API auditing | Performance monitoring |
| **What** | Who did what | Resource metrics |
| **Data** | API calls | Performance data |
| **Use Case** | Security, compliance | Operations, troubleshooting |

## Best Practices
1. ✅ Enable in all regions
2. ✅ Encrypt logs with KMS
3. ✅ Enable log file validation
4. ✅ Central logging account
5. ✅ Set up CloudWatch alarms for suspicious activity

---

[← Previous: CloudWatch](10-CloudWatch.md) | [Back to Main Guide](README.md) | [Next: Lambda →](12-Lambda.md)
