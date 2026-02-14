# 16. 🔒 Security & Compliance

## Key Security Services

### 1. AWS KMS (Key Management Service)
- Create and manage encryption keys
- Integrated with AWS services
- Hardware Security Modules (HSMs)

### 2. AWS Secrets Manager
- Store secrets (passwords, API keys)
- Automatic rotation
- Integration with RDS, Redshift

### 3. AWS WAF (Web Application Firewall)
- Protect against web exploits
- SQL injection, XSS protection
- Rate limiting

### 4. AWS Shield
- **Standard**: Free DDoS protection
- **Advanced**: $3,000/month, enhanced DDoS protection

### 5. AWS GuardDuty
- Intelligent threat detection
- Machine learning-based
- Monitors CloudTrail, VPC Flow Logs, DNS logs

### 6. AWS Inspector
- Automated security assessments
- Vulnerabilities and deviations
- Network and host assessments

### 7. AWS Config
- Resource configuration history
- Compliance auditing
- Configuration change notifications

## Encryption

### At Rest
- **S3**: SSE-S3, SSE-KMS, SSE-C
- **EBS**: Encrypted volumes
- **RDS**: Transparent Data Encryption (TDE)

### In Transit
- **TLS/SSL**: HTTPS, SSL certificates
- **VPN**: Site-to-site, client VPN
- **Direct Connect**: Private connection (+ VPN for encryption)

## Compliance Programs
- PCI-DSS
- HIPAA
- SOC 1, 2, 3
- ISO 27001
- GDPR
- FedRAMP

---

[← Previous: CI/CD Pipeline](15-CI-CD-Pipeline.md) | [Back to Main Guide](README.md) | [Next: Cost Optimization →](17-Cost-Optimization.md)
