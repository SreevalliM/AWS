# 19. 🆘 Disaster Recovery

## DR Concepts

### Recovery Metrics
- **RTO** (Recovery Time Objective): How long to recover
- **RPO** (Recovery Point Objective): How much data loss acceptable

**Example**:
- RTO = 4 hours: System must be back online within 4 hours
- RPO = 1 hour: Can afford to lose up to 1 hour of data

### DR Strategies (Fastest to Slowest)

#### 1. Multi-Site Active/Active
- **RTO**: Minutes
- **RPO**: Near zero
- **Cost**: Highest
- **Setup**: Full production in multiple regions, both active
- **Use Case**: Mission-critical applications

#### 2. Hot Standby (Warm Standby)
- **RTO**: Minutes to hours
- **RPO**: Minutes
- **Cost**: High
- **Setup**: Scaled-down version running, scale up when needed
- **Use Case**: Important production workloads

#### 3. Pilot Light
- **RTO**: Hours
- **RPO**: Minutes (with continuous replication)
- **Cost**: Medium
- **Setup**: Core systems replicated, scale up during DR
- **Use Case**: Less critical workloads

#### 4. Backup & Restore
- **RTO**: Hours to days
- **RPO**: Hours
- **Cost**: Lowest
- **Setup**: Regular backups, restore when needed
- **Use Case**: Non-critical systems

## Implementation

### Backup Strategies
- **Automated Backups**: RDS, EBS snapshots
- **Cross-Region Replication**: S3, RDS
- **AWS Backup**: Centralized backup service
- **Lifecycle Policies**: Retention and deletion

### High Availability Architecture
```
Region 1 (Primary)           Region 2 (DR)
    ↓                             ↓
Route 53 (DNS Failover)
    ↓                             ↓
ALB                           ALB (Standby)
    ↓                             ↓
EC2 Auto Scaling             EC2 (Minimal/None)
    ↓                             ↓
RDS Multi-AZ     →→→→→    RDS Read Replica
    ↓                             ↓
S3 Replication   →→→→→    S3 (Cross-Region)
```

### Failover Process
1. **Detection**: Route 53 health checks
2. **Notification**: SNS, CloudWatch alarms
3. **Activation**: Manual or automated
4. **Scaling**: Increase capacity in DR region
5. **Traffic Shift**: Route 53 failover
6. **Validation**: Test all services

### Best Practices
1. ✅ **Document** DR plan
2. ✅ **Test** regularly (quarterly)
3. ✅ **Automate** failover when possible
4. ✅ **Monitor** continuously
5. ✅ **Multi-region** for critical workloads
6. ✅ **Version control** IaC templates
7. ✅ **Train team** on DR procedures

---

[← Previous: Well-Architected Framework](18-Well-Architected-Framework.md) | [Back to Main Guide](README.md) | [Next: Hands-on Projects →](20-Projects.md)
