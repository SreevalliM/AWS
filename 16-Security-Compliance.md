# 16. 🔒 Security & Compliance

**Protect your data, accounts, and workloads on AWS**

## Security is Job Zero

AWS follows the **Shared Responsibility Model** (covered in Topic 1). Your job is to secure everything IN the cloud — data, IAM, network config, encryption, and application security.

---

## Key Security Services

### 1. AWS KMS (Key Management Service)

Create and manage **encryption keys** for data protection.

| Feature | Details |
|---------|---------|
| Key types | Symmetric (AES-256), Asymmetric (RSA, ECC) |
| Integration | S3, EBS, RDS, Lambda, Secrets Manager, 100+ services |
| Key rotation | Automatic (yearly) or on-demand |
| Audit | All key usage logged in CloudTrail |
| Pricing | $1/key/month + $0.03 per 10,000 API calls |

**Customer Managed Keys (CMK)** — You create, manage, and control
**AWS Managed Keys** — AWS creates and manages (per-service)
**AWS Owned Keys** — Shared across accounts (free, no control)

```bash
# Create a KMS key
aws kms create-key --description "My encryption key"

# Encrypt data
aws kms encrypt \
  --key-id alias/my-key \
  --plaintext fileb://secret.txt \
  --output text --query CiphertextBlob | base64 --decode > encrypted.dat

# Decrypt data
aws kms decrypt \
  --ciphertext-blob fileb://encrypted.dat \
  --output text --query Plaintext | base64 --decode > decrypted.txt
```

### 2. AWS Secrets Manager

Store and automatically rotate **secrets** (passwords, API keys, tokens).

| Feature | Details |
|---------|---------|
| Auto-rotation | Built-in for RDS, Redshift, DocumentDB |
| Versioning | Track secret versions |
| Access control | IAM policies + resource policies |
| Cross-account | Share secrets across accounts |
| Pricing | $0.40/secret/month + $0.05 per 10,000 API calls |

```python
import boto3
import json

client = boto3.client('secretsmanager')

# Retrieve secret
response = client.get_secret_value(SecretId='myapp/database')
secret = json.loads(response['SecretString'])

db_host = secret['host']
db_user = secret['username']
db_pass = secret['password']
```

**Secrets Manager vs SSM Parameter Store:**

| Feature | Secrets Manager | Parameter Store |
|---------|----------------|-----------------|
| Auto-rotation | ✅ Built-in | ❌ Manual (via Lambda) |
| Cost | $0.40/secret/month | Free (standard) / $0.05 (advanced) |
| Cross-account | ✅ | Limited |
| Best for | Database creds, API keys | Config values, feature flags |

### 3. AWS WAF (Web Application Firewall)

Protect web apps from common exploits at **Layer 7**.

**What it blocks:**
- SQL injection
- Cross-site scripting (XSS)
- Known bad IPs (IP reputation lists)
- Bot traffic
- Geo-based restrictions
- Rate-based rules (DDoS at Layer 7)

**Deployment points:** CloudFront, ALB, API Gateway, AppSync

**Rule Groups:**
- AWS Managed Rules (free/paid)
- Partner rules (F5, Fortinet, etc.)
- Custom rules

```bash
# Block requests from specific country
aws wafv2 create-ip-set \
  --name blocked-countries \
  --scope REGIONAL \
  --ip-address-version IPV4 \
  --addresses []
```

### 4. AWS Shield

**DDoS protection** for AWS resources.

| Feature | Shield Standard | Shield Advanced |
|---------|----------------|-----------------|
| Cost | Free | $3,000/month |
| Protection | Layer 3/4 DDoS | Layer 3/4/7 DDoS |
| Response team | ❌ | 24/7 DRT (DDoS Response Team) |
| Cost protection | ❌ | ✅ Credits for scaling costs |
| Visibility | Basic | Advanced metrics & reporting |
| Applies to | All AWS customers | CloudFront, ALB, Route 53, Global Accelerator |

### 5. Amazon GuardDuty

**Intelligent threat detection** using ML and anomaly detection.

**Data sources analyzed:**
- CloudTrail management & data events
- VPC Flow Logs
- DNS query logs
- S3 data events
- EKS audit logs

**Finding types:**
- Unauthorized API calls
- Compromised EC2 instances (crypto-mining, C&C communication)
- Unusual S3 access patterns
- Brute-force SSH/RDP attempts
- Malicious IP communication

**Setup:** One click to enable. Findings → EventBridge → Lambda (auto-remediate)

### 6. Amazon Inspector

**Automated vulnerability scanning** for EC2, containers, and Lambda.

- Network reachability analysis
- OS & software vulnerability scanning (CVEs)
- Continuous scanning (not just one-time)
- Risk score prioritization
- Integration with Security Hub

### 7. AWS Config

**Configuration compliance** — Track resource configurations and evaluate against rules.

| Feature | Details |
|---------|---------|
| Configuration history | Full timeline of every resource change |
| Compliance rules | 300+ managed rules |
| Remediation | Auto-fix non-compliant resources |
| Aggregation | Multi-account, multi-region dashboard |

**Example rules:**
- `s3-bucket-public-read-prohibited` — Ensure no S3 bucket is publicly readable
- `ec2-instance-no-public-ip` — EC2 instances shouldn't have public IPs
- `rds-instance-public-access-check` — RDS should not be publicly accessible
- `iam-root-access-key-check` — Root account should not have access keys

### 8. AWS Security Hub

**Central dashboard** that aggregates findings from GuardDuty, Inspector, Config, Macie, Firewall Manager, and third-party tools.

- Security score based on compliance standards
- CIS AWS Foundations Benchmark
- PCI DSS, NIST 800-53
- Automated compliance checks

---

## Encryption

### At Rest

| Service | Encryption Method |
|---------|-------------------|
| S3 | SSE-S3, SSE-KMS, SSE-C |
| EBS | AES-256 via KMS |
| RDS | TDE (Oracle/SQL Server), KMS for others |
| DynamoDB | AWS owned key or CMK |
| Lambda | Environment variables encrypted via KMS |
| EFS | KMS encryption |

### In Transit

| Method | Use Case |
|--------|----------|
| TLS/SSL | HTTPS for APIs, websites |
| VPN | Site-to-site encrypted tunnel |
| Direct Connect + VPN | Private + encrypted connection |
| ACM (Certificate Manager) | Free public SSL/TLS certificates |

---

## Network Security Layers

```
Internet
    ↓
AWS Shield (DDoS protection)
    ↓
WAF (Application firewall)
    ↓
CloudFront (CDN + edge security)
    ↓
Route 53 (DNS filtering)
    ↓
VPC (Network isolation)
    ↓
NACLs (Subnet-level stateless firewall)
    ↓
Security Groups (Instance-level stateful firewall)
    ↓
OS Firewall (iptables / Windows Firewall)
    ↓
Application Security (code-level)
```

---

## Compliance Programs

AWS infrastructure is certified for major compliance standards:

| Standard | Focus |
|----------|-------|
| **PCI DSS** | Payment card data |
| **HIPAA** | Healthcare data (requires BAA) |
| **SOC 1/2/3** | Service organization controls |
| **ISO 27001** | Information security management |
| **GDPR** | EU data protection |
| **FedRAMP** | US government |
| **NIST 800-53** | Federal security controls |

Use **AWS Artifact** to download compliance reports and agreements.

---

## ✅ Hands-on Lab: Security Configuration

### Lab 1: Enable GuardDuty

1. GuardDuty Console → **Get Started** → **Enable GuardDuty**
2. Wait for findings (may take 15–30 min)
3. Review sample findings:
   - High severity: Compromised instance
   - Medium: Unusual API calls
   - Low: Port probe
4. **Set up EventBridge rule** to send high-severity findings to SNS

### Lab 2: Create and Use KMS Key

1. KMS → **Create key** → Symmetric
2. Alias: `my-app-key`
3. Key administrators: Your IAM user
4. Key users: Your IAM user + Lambda role
5. Create S3 bucket with SSE-KMS using your key
6. Upload file → Verify it's encrypted with your key

### Lab 3: Store Secrets in Secrets Manager

1. Secrets Manager → **Store a new secret**
2. Type: Other type of secret
3. Key/value:
   - `db_host`: `mydb.abc123.us-east-1.rds.amazonaws.com`
   - `db_user`: `admin`
   - `db_password`: `SuperSecret123!`
4. Secret name: `myapp/database`
5. Retrieve from Lambda (see Python example above)

### Lab 4: AWS Config Rules

1. Config → **Get started**
2. Record all resources
3. Add rules:
   - `s3-bucket-public-read-prohibited`
   - `restricted-ssh` (no SSH from 0.0.0.0/0)
   - `encrypted-volumes` (all EBS encrypted)
4. Review compliance dashboard
5. Set up auto-remediation for non-compliant resources

---

## Security Best Practices Summary

1. ✅ **IAM** — Least privilege, MFA, roles over keys
2. ✅ **Network** — VPC isolation, security groups, NACLs
3. ✅ **Encryption** — At rest (KMS) and in transit (TLS)
4. ✅ **Secrets** — Secrets Manager (never hardcode)
5. ✅ **Monitoring** — GuardDuty, CloudTrail, Config, Security Hub
6. ✅ **Protection** — WAF, Shield, Inspector
7. ✅ **Compliance** — Config rules, Artifact reports
8. ✅ **Incident response** — Automate with Lambda via EventBridge
9. ✅ **Logging** — Enable everything, retain appropriately
10. ✅ **Regular audits** — Review access, rotate credentials

## Common Interview Questions

1. **Q: How to encrypt an existing unencrypted EBS volume?**
   - Create snapshot → Copy snapshot with encryption → Create volume from encrypted snapshot.

2. **Q: KMS vs CloudHSM?**
   - KMS: Multi-tenant, managed keys, integrated with AWS. CloudHSM: Dedicated hardware, full key control, FIPS 140-2 Level 3.

3. **Q: How to detect if an S3 bucket is public?**
   - AWS Config rule `s3-bucket-public-read-prohibited`, Trusted Advisor, Access Analyzer.

4. **Q: What is GuardDuty?**
   - ML-based threat detection that analyzes CloudTrail, VPC Flow Logs, and DNS logs for suspicious activity.

5. **Q: WAF vs Shield?**
   - WAF: Layer 7, application-level rules (SQL injection, XSS). Shield: Layer 3/4, DDoS protection.

6. **Q: How to handle a suspected security breach?**
   - Rotate credentials → Review CloudTrail → Disable compromised users → Check resources → Enable GuardDuty → Incident response.

---

[← Previous: CI/CD Pipeline](15-CI-CD-Pipeline.md) | [Back to Main Guide](README.md) | [Next: Cost Optimization →](17-Cost-Optimization.md)
