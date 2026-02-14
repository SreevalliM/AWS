# 9. ⚖️ Auto Scaling & Load Balancing

## Auto Scaling

Automatically adjust compute capacity to maintain steady, predictable performance.

### Components
1. **Launch Template/Configuration**: Defines instance configuration
2. **Auto Scaling Group**: Manages collection of EC2 instances
3. **Scaling Policies**: Rules for when to scale
4. **CloudWatch Alarms**: Trigger scaling actions

### Scaling Policies
- **Target Tracking**: Maintain specific metric (e.g., 70% CPU)
- **Step Scaling**: Scale based on CloudWatch alarm thresholds
- **Simple Scaling**: Add/remove specific number of instances
- **Scheduled Scaling**: Scale at specific times

### Example Configuration
```bash
# Create Auto Scaling group
aws autoscaling create-auto-scaling-group \
  --auto-scaling-group-name my-asg \
  --launch-template LaunchTemplateName=my-template \
  --min-size 2 \
  --max-size 10 \
  --desired-capacity 4 \
  --vpc-zone-identifier "subnet-1,subnet-2" \
  --target-group-arns arn:aws:elasticloadbalancing:region:account:targetgroup/my-targets/1234567890
```

## Load Balancing

Distribute incoming traffic across multiple targets.

### Load Balancer Types

**1. Application Load Balancer (ALB)**
- **Layer**: 7 (HTTP/HTTPS)
- **Features**: Path-based routing, host-based routing, WebSocket
- **Use Case**: Web applications, microservices
- **Target Types**: EC2, IP, Lambda

**2. Network Load Balancer (NLB)**
- **Layer**: 4 (TCP/UDP)
- **Features**: Ultra-low latency, static IP, TLS termination
- **Use Case**: Extreme performance, TCP/UDP traffic
- **Target Types**: EC2, IP, ALB

**3. Gateway Load Balancer (GWLB)**
- **Layer**: 3 (Network)
- **Features**: Deploy/scale virtual appliances
- **Use Case**: Firewalls, intrusion detection

**4. Classic Load Balancer (CLB)**
- **Layer**: 4 & 7
- **Status**: Legacy (use ALB/NLB instead)

### ALB Components
```
Internet → ALB → Listener (Port 80/443)
                     ↓
               Target Groups
                     ↓
            EC2 Instances (Targets)
```

### Target Group Health Checks
- Protocol: HTTP, HTTPS, TCP
- Path: `/health`
- Healthy threshold: 2 consecutive successes
- Unhealthy threshold: 2 consecutive failures
- Interval: 30 seconds
- Timeout: 5 seconds

### Example: Create ALB
```bash
# Create Application Load Balancer
aws elbv2 create-load-balancer \
  --name my-alb \
  --subnets subnet-1 subnet-2 \
  --security-groups sg-12345678 \
  --scheme internet-facing \
  --type application \
  --ip-address-type ipv4

# Create target group
aws elbv2 create-target-group \
  --name my-targets \
  --protocol HTTP \
  --port 80 \
  --vpc-id vpc-12345678 \
  --health-check-path /health
```

---

[← Previous: DynamoDB](08-DynamoDB.md) | [Back to Main Guide](README.md) | [Next: CloudWatch →](10-CloudWatch.md)
