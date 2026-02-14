# 17. 💰 Cost Optimization

## Cost Optimization Strategies

### 1. Right-Sizing
- Match instance types to workload
- Use AWS Compute Optimizer
- Monitor CloudWatch metrics

### 2. Purchasing Options
- **Reserved Instances**: Up to 75% savings (1-3 years)
- **Savings Plans**: Flexible commitment
- **Spot Instances**: Up to 90% savings (interruptible)

### 3. Storage Optimization
- **S3 Lifecycle policies**: Move to cheaper storage classes
- **EBS snapshots**: Delete old snapshots
- **S3 Intelligent-Tiering**: Automatic cost optimization

### 4. Scaling
- **Auto Scaling**: Scale down during low usage
- **Lambda**: Pay only for execution time
- **Fargate**: No idle instance costs

### 5. Data Transfer
- **VPC Endpoints**: Free S3/DynamoDB access (no internet)
- **CloudFront**: Reduce origin requests
- **Direct Connect**: Lower data transfer costs (high volume)

## Cost Management Tools

### AWS Cost Explorer
- Visualize spending patterns
- Forecast future costs
- Filter by service, tag, region

### AWS Budgets
- Set custom cost/usage budgets
- Alerts when exceeding thresholds
- Action-based budgets (automated responses)

### AWS Trusted Advisor
- Cost optimization recommendations
- Idle resources
- Reserved Instance optimization

### Cost Allocation Tags
```
Project: WebApp
Environment: Production
Team: Engineering
CostCenter: 1234
```

## Quick Wins
1. ✅ Delete unused resources (EC2, EBS, EIPs)
2. ✅ Stop non-production instances overnight
3. ✅ Enable S3 lifecycle policies
4. ✅ Use Reserved Instances for steady workloads
5. ✅ Implement Auto Scaling
6. ✅ Review CloudWatch metrics for underutilized resources

---

[← Previous: Security & Compliance](16-Security-Compliance.md) | [Back to Main Guide](README.md) | [Next: Well-Architected Framework →](18-Well-Architected-Framework.md)
