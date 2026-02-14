# 3. 🔐 IAM (Identity & Access Management)

**⚠️ CRITICAL SERVICE - Master this first!**

![IAM Screenshot](https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2022/06/09/PLU-screenshot-2-border-4.png)

![IAM Diagram](https://miro.medium.com/0%2A2qvQJOBLk8HEDQNd)

![IAM Users vs Roles](https://d2908q01vomqb2.cloudfront.net/cb4e5208b4cd87268b208e49452ed6e89a68e0b8/2022/02/11/IAM-Users-vs-IAM-Roles-1.png)

![IAM User vs Role Comparison](https://www.archerimagine.com/images/aws/IAM/IAM-User-Vs-Role.png)

## What is IAM?

AWS Identity and Access Management (IAM) enables you to manage access to AWS services and resources securely. It's a **global service** (not region-specific).

### Key Features
- ✅ Free to use
- ✅ Centralized access control
- ✅ Fine-grained permissions
- ✅ Multi-factor authentication (MFA)
- ✅ Identity federation support
- ✅ Temporary credentials

## IAM Components

### 1. Users
**Individual identities** for people or applications

- Each user has unique credentials
- Long-term credentials (password, access keys)
- Direct console or programmatic access
- Can belong to multiple groups

**Best Practices:**
- One user per person
- Never share credentials
- Rotate credentials regularly
- Enable MFA

### 2. Groups
**Collections of users** with common permissions

- Easier permission management
- Users inherit group permissions
- No nesting (groups can't contain groups)
- One user can be in multiple groups

**Example Groups:**
- Developers
- Administrators
- DevOps
- Read-Only-Users

### 3. Roles
**Temporary identities** for services or federated users

- No permanent credentials
- Assumed by AWS services or users
- Temporary security credentials
- Cross-account access

**Common Use Cases:**
- EC2 accessing S3
- Lambda accessing DynamoDB
- Cross-account access
- Federated users (SSO)

### 4. Policies
**JSON documents** defining permissions

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-bucket/*"
    }
  ]
}
```

**Policy Types:**
- **AWS Managed**: Created and maintained by AWS
- **Customer Managed**: Created by you, reusable
- **Inline**: Directly attached to single user/group/role

## Users vs Roles - When to Use What?

| Feature | IAM Users | IAM Roles |
|---------|-----------|-----------|
| **Identity Type** | Permanent | Temporary |
| **Credentials** | Password/Access Keys | Temporary tokens |
| **Best For** | Humans, specific apps | Services, cross-account |
| **Example** | Developer John | EC2 accessing S3 |

## Policy Structure Explained

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "UniqueIdentifier",
      "Effect": "Allow",          // Allow or Deny
      "Principal": "*",            // Who (for resource policies)
      "Action": [                  // What actions
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::bucket/*",  // Which resources
      "Condition": {               // When (optional)
        "IpAddress": {
          "aws:SourceIp": "203.0.113.0/24"
        }
      }
    }
  ]
}
```

### Common Actions
- `s3:*` - All S3 actions
- `ec2:Describe*` - All EC2 describe actions
- `dynamodb:GetItem` - Specific DynamoDB action
- `*:*` - All actions on all services (avoid!)

## Multi-Factor Authentication (MFA)

### Types of MFA
1. **Virtual MFA Device**: Google Authenticator, Authy
2. **Hardware MFA**: YubiKey, Gemalto
3. **SMS Text Message** (not recommended)

### Enable MFA Steps
1. Go to IAM → Users → Security credentials
2. Click "Assign MFA device"
3. Choose device type
4. Scan QR code (virtual MFA)
5. Enter two consecutive codes

## Least Privilege Principle

**Give only the permissions needed to perform a task**

❌ **Bad Practice:**
```json
{
  "Effect": "Allow",
  "Action": "*",
  "Resource": "*"
}
```

✅ **Good Practice:**
```json
{
  "Effect": "Allow",
  "Action": [
    "s3:GetObject",
    "s3:PutObject"
  ],
  "Resource": "arn:aws:s3:::my-app-bucket/*"
}
```

## Access Keys

Used for programmatic access (AWS CLI, SDK)

### Components
- **Access Key ID**: Like username (e.g., AKIAIOSFODNN7EXAMPLE)
- **Secret Access Key**: Like password (shown only once)

### Security Best Practices
- Never commit to code repositories
- Rotate regularly (every 90 days)
- Use roles instead when possible
- Store in environment variables or AWS Secrets Manager

## IAM Best Practices Summary

1. ✅ **Root Account**: Lock it down, enable MFA, don't use daily
2. ✅ **Individual Users**: One per person, no sharing
3. ✅ **Groups**: Manage permissions via groups
4. ✅ **Least Privilege**: Start with minimum, add as needed
5. ✅ **MFA**: Enable for privileged users
6. ✅ **Roles**: Use for AWS services
7. ✅ **Rotate Credentials**: Regular rotation policy
8. ✅ **Password Policy**: Enforce strong passwords
9. ✅ **Monitoring**: Use CloudTrail for auditing
10. ✅ **Remove Unused**: Delete old users and credentials

## ✅ Hands-on Lab: IAM Configuration

### Lab 1: Create IAM User
1. Go to **IAM Console**
2. Click **Users** → **Add users**
3. Enter username: `developer-john`
4. Select **AWS Management Console access**
5. Choose **Custom password**
6. Uncheck "Require password reset"
7. Click **Next**
8. Select **Attach policies directly**
9. Search and select `ReadOnlyAccess`
10. Click **Next** → **Create user**

### Lab 2: Create Custom Policy
1. Go to **IAM** → **Policies** → **Create policy**
2. Click **JSON** tab
3. Paste:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:ListBucket",
        "s3:GetObject"
      ],
      "Resource": [
        "arn:aws:s3:::my-app-bucket",
        "arn:aws:s3:::my-app-bucket/*"
      ]
    }
  ]
}
```
4. Click **Next**
5. Name: `S3ReadOnlyMyAppBucket`
6. Click **Create policy**

### Lab 3: Create Role for EC2
1. Go to **IAM** → **Roles** → **Create role**
2. Select **AWS service** → **EC2**
3. Click **Next**
4. Search and select `AmazonS3ReadOnlyAccess`
5. Click **Next**
6. Role name: `EC2-S3-ReadOnly-Role`
7. Click **Create role**

### Lab 4: Create IAM Group
1. Go to **IAM** → **User groups** → **Create group**
2. Group name: `Developers`
3. Attach policies: `PowerUserAccess`
4. Click **Create group**
5. Add users to group

### Lab 5: Enable MFA for User
1. Go to **IAM** → **Users** → Select user
2. Click **Security credentials** tab
3. In **Multi-factor authentication**, click **Assign MFA device**
4. Choose **Authenticator app**
5. Scan QR code with Google Authenticator
6. Enter two consecutive MFA codes
7. Click **Add MFA**

## Common IAM Interview Questions

1. **Q: Difference between IAM User and IAM Role?**
   - User: Permanent identity with long-term credentials
   - Role: Temporary identity assumed when needed

2. **Q: What is the IAM policy evaluation logic?**
   - Explicit Deny → Explicit Allow → Implicit Deny

3. **Q: How to provide cross-account access?**
   - Create IAM role in target account with trust policy
   - Source account assumes the role

4. **Q: What is IAM best practice for EC2 accessing S3?**
   - Use IAM role attached to EC2, not access keys

---

[← Previous: AWS Account Setup](02-AWS-Account-Setup.md) | [Back to Main Guide](README.md) | [Next: EC2 →](04-EC2.md)
