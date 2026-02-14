# 18. 🏛️ Well-Architected Framework

AWS best practices across five pillars.

## The 5 Pillars

### 1. Operational Excellence
**Focus**: Run and monitor systems

**Principles**:
- Perform operations as code (IaC)
- Annotate documentation
- Make frequent, small, reversible changes
- Refine operations procedures frequently
- Anticipate failure
- Learn from operational failures

**Services**: CloudFormation, CloudWatch, X-Ray

### 2. Security
**Focus**: Protect information and systems

**Principles**:
- Implement strong identity foundation (IAM)
- Enable traceability (CloudTrail)
- Apply security at all layers
- Automate security best practices
- Protect data in transit and at rest
- Keep people away from data
- Prepare for security events

**Services**: IAM, KMS, GuardDuty, WAF, Shield

### 3. Reliability
**Focus**: Ensure correct, consistent performance

**Principles**:
- Automatically recover from failure
- Test recovery procedures
- Scale horizontally
- Stop guessing capacity
- Manage change through automation

**Design**:
- Multi-AZ deployments
- Auto Scaling
- Load balancing
- Automated backups
- Disaster recovery plan

**Services**: Auto Scaling, CloudWatch, Route 53, RDS Multi-AZ

### 4. Performance Efficiency
**Focus**: Use resources efficiently

**Principles**:
- Democratize advanced technologies
- Go global in minutes
- Use serverless architectures
- Experiment more often
- Consider mechanical sympathy

**Strategies**:
- Select right instance types
- Use managed services
- Monitor performance
- Make trade-offs (consistency vs. latency)

**Services**: Lambda, CloudFront, ElastiCache, CloudWatch

### 5. Cost Optimization
**Focus**: Avoid unnecessary costs

**Principles**:
- Adopt consumption model
- Measure overall efficiency
- Stop spending on data center operations
- Analyze and attribute expenditure
- Use managed services

**Strategies**:
- Right-sizing
- Reserved capacity
- Spot instances
- Auto Scaling
- Monitor and analyze

**Services**: Cost Explorer, Trusted Advisor, Auto Scaling

## Well-Architected Review
- Assess workloads against best practices
- Identify high/medium risk issues
- Generate improvement plan
- Use AWS Well-Architected Tool (free)

---

[← Previous: Cost Optimization](17-Cost-Optimization.md) | [Back to Main Guide](README.md) | [Next: Disaster Recovery →](19-Disaster-Recovery.md)
