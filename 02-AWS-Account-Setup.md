# 2. 🔧 AWS Account Setup

## Creating Your AWS Account

### Prerequisites
- Valid email address
- Credit/debit card (for verification)
- Phone number

### Steps
1. Go to aws.amazon.com
2. Click "Create an AWS Account"
3. Enter email and account name
4. Verify email address
5. Set root user password
6. Choose Free Tier plan
7. Enter payment information
8. Verify identity (phone)
9. Select support plan (Basic - Free)

## AWS Free Tier

### Types
1. **Always Free**: DynamoDB (25 GB), Lambda (1M requests/month)
2. **12 Months Free**: EC2 (750 hours/month), S3 (5 GB), RDS (750 hours/month)
3. **Trials**: Short-term trials for specific services

## Exploring AWS Console

### Main Components
- **Services Menu**: Access all AWS services
- **Resource Groups**: Organize resources
- **CloudShell**: Browser-based shell
- **Notifications**: Service health and updates
- **Account Menu**: Billing, security credentials, settings

### Important Sections
1. **EC2 Dashboard**: Compute resources
2. **S3 Console**: Storage buckets
3. **VPC Dashboard**: Networking
4. **IAM Dashboard**: Security and access
5. **Billing Dashboard**: Cost management

## Setting Up Billing Alerts

### Using CloudWatch Alarms
1. Go to **Billing Dashboard**
2. Click **Billing Preferences**
3. Enable **Receive Billing Alerts**
4. Go to **CloudWatch** → **Alarms**
5. Create new alarm:
   - Metric: `Total Estimated Charge`
   - Threshold: $10 (or your preferred amount)
   - Notification: Email alert

### Using AWS Budgets
1. Go to **AWS Budgets**
2. Click **Create budget**
3. Choose **Cost budget**
4. Set monthly budget amount
5. Configure alerts at 80% and 100%

## Security Best Practices for Root Account
- ✅ Enable MFA immediately
- ✅ Don't use root account for daily tasks
- ✅ Create IAM users instead
- ✅ Delete root access keys if exist
- ✅ Use strong, unique password

---

[← Previous: Cloud Computing Basics](01-Cloud-Computing-Basics.md) | [Back to Main Guide](README.md) | [Next: IAM →](03-IAM.md)
