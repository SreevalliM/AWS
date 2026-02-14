# 9. ⚖️ Auto Scaling & Load Balancing

**High availability and elasticity for your applications**

## What is Auto Scaling?

Automatically adjust compute capacity to maintain steady, predictable performance at the lowest possible cost.

### Key Benefits
- ✅ **High availability** — Replace unhealthy instances automatically
- ✅ **Cost optimization** — Scale down during low demand
- ✅ **Performance** — Scale up during traffic spikes
- ✅ **Fault tolerance** — Distribute across AZs

## Auto Scaling Components

### 1. Launch Template / Launch Configuration
Defines **what** to launch (instance blueprint):

| Setting | Description |
|---------|-------------|
| AMI | Which OS / image to use |
| Instance Type | Size (e.g., t3.micro) |
| Key Pair | SSH access |
| Security Groups | Firewall rules |
| User Data | Bootstrap script |
| IAM Role | Permissions for the instance |
| Storage | EBS volume config |

> **Note:** Launch Templates are recommended over Launch Configurations (which are now legacy).

### 2. Auto Scaling Group (ASG)
Defines **where** and **how many** to launch:

- **Min size** — Minimum instances (e.g., 2)
- **Max size** — Maximum instances (e.g., 10)
- **Desired capacity** — Target number right now (e.g., 4)
- **VPC / Subnets** — Which AZs to spread across
- **Target Group** — Load balancer attachment
- **Health Check Type** — EC2 (default) or ELB
- **Health Check Grace Period** — Seconds to wait before checking new instance

### 3. Scaling Policies

**Target Tracking** (most common):
- Maintain a specific CloudWatch metric value
- Example: Keep average CPU at 50%
- ASG adds/removes instances automatically

**Step Scaling**:
- Define step adjustments based on alarm thresholds
- Example: Add 2 instances when CPU > 70%, add 4 when CPU > 90%

**Simple Scaling**:
- Add or remove a fixed number when alarm triggers
- Waits for cooldown period before next action

**Scheduled Scaling**:
- Scale at specific times (e.g., scale up Mon–Fri 9 AM)
- Use cron expressions or date/time

**Predictive Scaling**:
- Uses machine learning to forecast traffic
- Proactively scales before demand arrives

### 4. Cooldown Period
- Default: 300 seconds (5 minutes)
- Prevents ASG from launching/terminating instances too quickly
- Gives new instances time to start handling traffic

## Auto Scaling Lifecycle

```
Pending → InService → (Scaling Event) → Terminating → Terminated
                ↓
        Standby (manual)
```

### Lifecycle Hooks
Run custom actions during scale-out or scale-in:
- **Launch hook** — Run setup scripts, register with config management
- **Terminate hook** — Drain connections, save logs, deregister

## Example: CLI Configuration

```bash
# Create launch template
aws ec2 create-launch-template \
  --launch-template-name my-web-template \
  --version-description "v1" \
  --launch-template-data '{
    "ImageId": "ami-0c55b159cbfafe1f0",
    "InstanceType": "t3.micro",
    "KeyName": "my-key",
    "SecurityGroupIds": ["sg-12345678"],
    "UserData": "IyEvYmluL2Jhc2gKeXVtIHVwZGF0ZSAteQ=="
  }'

# Create Auto Scaling group
aws autoscaling create-auto-scaling-group \
  --auto-scaling-group-name my-asg \
  --launch-template LaunchTemplateName=my-web-template,Version='$Latest' \
  --min-size 2 \
  --max-size 10 \
  --desired-capacity 4 \
  --vpc-zone-identifier "subnet-1,subnet-2" \
  --target-group-arns arn:aws:elasticloadbalancing:region:account:targetgroup/my-targets/1234567890 \
  --health-check-type ELB \
  --health-check-grace-period 300

# Create target tracking policy
aws autoscaling put-scaling-policy \
  --auto-scaling-group-name my-asg \
  --policy-name cpu-target-tracking \
  --policy-type TargetTrackingScaling \
  --target-tracking-configuration '{
    "PredefinedMetricSpecification": {
      "PredefinedMetricType": "ASGAverageCPUUtilization"
    },
    "TargetValue": 50.0
  }'
```

---

## Load Balancing

Distribute incoming traffic across multiple targets for high availability.

### Load Balancer Types

| Feature | ALB | NLB | GWLB | CLB (Legacy) |
|---------|-----|-----|------|-------------|
| **Layer** | 7 (HTTP/HTTPS) | 4 (TCP/UDP) | 3 (Network) | 4 & 7 |
| **Latency** | ~400ms | ~100μs | Low | Medium |
| **Static IP** | No (use Global Accelerator) | Yes | Yes | No |
| **WebSocket** | ✅ | ✅ | ❌ | ❌ |
| **Path routing** | ✅ | ❌ | ❌ | ❌ |
| **Host routing** | ✅ | ❌ | ❌ | ❌ |
| **Target types** | EC2, IP, Lambda | EC2, IP, ALB | EC2, IP | EC2 |
| **Use case** | Web apps, microservices | Extreme perf, TCP/UDP | Virtual appliances | Legacy only |

### 1. Application Load Balancer (ALB)

**Layer 7** — HTTP/HTTPS traffic with advanced routing.

**Routing Rules:**
- **Path-based**: `/api/*` → API servers, `/images/*` → Media servers
- **Host-based**: `api.example.com` → API target group
- **Query string/header**: Route based on `?platform=mobile`
- **HTTP method**: GET vs POST to different targets

**Sticky Sessions:**
- Route user to same target for session persistence
- Cookie-based (application or LB-generated)
- Duration: 1 sec to 7 days

### 2. Network Load Balancer (NLB)

**Layer 4** — TCP/UDP/TLS with ultra-high performance.

- Handles millions of requests per second
- Static IP per AZ (or Elastic IP)
- Preserves source IP address
- Best for: Gaming, IoT, financial trading, TCP/UDP protocols

### 3. Gateway Load Balancer (GWLB)

**Layer 3** — Deploy and scale third-party virtual appliances.

- Transparent to the application
- Single entry/exit for all traffic
- Best for: Firewalls, IDS/IPS, deep packet inspection

### ALB Architecture

```
Internet
    ↓
Application Load Balancer
    ↓
Listener (Port 80) ──────────── Listener (Port 443)
    ↓                                 ↓
Rule: /api/*  → API Target Group     Default → Web Target Group
Rule: default → Web Target Group
    ↓                                 ↓
EC2 (us-east-1a)                 EC2 (us-east-1b)
EC2 (us-east-1b)                 EC2 (us-east-1a)
```

### Target Group Health Checks

| Setting | Description | Default |
|---------|-------------|---------|
| Protocol | HTTP, HTTPS, TCP | HTTP |
| Path | URL path for check | `/` |
| Healthy threshold | Consecutive successes | 5 |
| Unhealthy threshold | Consecutive failures | 2 |
| Interval | Seconds between checks | 30 |
| Timeout | Seconds to wait | 5 |

### Connection Draining / Deregistration Delay
- Time to complete in-flight requests before removing target
- Default: 300 seconds
- Set to 0 for short-lived requests

### Cross-Zone Load Balancing

| Setting | ALB | NLB |
|---------|-----|-----|
| **Default** | Enabled (free) | Disabled |
| **Effect** | Traffic distributed evenly across all AZ targets | Traffic stays within AZ |

---

## ✅ Hands-on Lab: Auto Scaling with ALB

### Lab 1: Create Auto Scaling Web Application

**Step 1: Create Launch Template**
1. EC2 → **Launch Templates** → **Create launch template**
2. Name: `web-server-template`
3. AMI: Amazon Linux 2023
4. Instance type: t2.micro
5. Key pair: Select your key
6. Security group: Allow HTTP (80), SSH (22)
7. User Data:
```bash
#!/bin/bash
yum update -y
yum install -y httpd stress
systemctl start httpd
systemctl enable httpd
INSTANCE_ID=$(curl -s http://169.254.169.254/latest/meta-data/instance-id)
AZ=$(curl -s http://169.254.169.254/latest/meta-data/placement/availability-zone)
echo "<h1>Instance: $INSTANCE_ID</h1><h2>AZ: $AZ</h2>" > /var/www/html/index.html
```
8. **Create launch template**

**Step 2: Create Target Group**
1. EC2 → **Target Groups** → **Create target group**
2. Target type: Instances
3. Name: `web-target-group`
4. Protocol: HTTP, Port: 80
5. Health check path: `/index.html`
6. **Create** (don't register targets yet)

**Step 3: Create Application Load Balancer**
1. EC2 → **Load Balancers** → **Create**
2. Type: Application Load Balancer
3. Name: `web-alb`
4. Scheme: Internet-facing
5. Subnets: Select 2+ public subnets (different AZs)
6. Security group: Allow HTTP (80)
7. Listener: HTTP:80 → Forward to `web-target-group`
8. **Create**

**Step 4: Create Auto Scaling Group**
1. EC2 → **Auto Scaling Groups** → **Create**
2. Name: `web-asg`
3. Launch template: `web-server-template`
4. Subnets: Same as ALB (2+ AZs)
5. Attach to: `web-target-group`
6. Health check type: **ELB**
7. Group size: Min=2, Max=6, Desired=2
8. Scaling policy: **Target tracking**
   - Metric: Average CPU utilization
   - Target value: 50%
9. **Create**

**Step 5: Test Load Balancing**
1. Copy ALB DNS name
2. Open in browser — see Instance ID
3. Refresh — see different Instance ID (round-robin)

**Step 6: Test Auto Scaling**
```bash
# SSH into one instance
ssh -i key.pem ec2-user@<instance-ip>

# Generate CPU load
stress --cpu 4 --timeout 300

# Watch ASG in console — new instances will launch
# After stopping stress, instances will terminate (scale in)
```

### Lab 2: Scheduled Scaling

```bash
# Scale up at 9 AM (Mon-Fri)
aws autoscaling put-scheduled-update-group-action \
  --auto-scaling-group-name web-asg \
  --scheduled-action-name scale-up-morning \
  --recurrence "0 9 * * 1-5" \
  --desired-capacity 4

# Scale down at 6 PM (Mon-Fri)
aws autoscaling put-scheduled-update-group-action \
  --auto-scaling-group-name web-asg \
  --scheduled-action-name scale-down-evening \
  --recurrence "0 18 * * 1-5" \
  --desired-capacity 2
```

---

## Auto Scaling Best Practices

1. ✅ **Use ELB health checks** — More accurate than EC2 status checks
2. ✅ **Spread across AZs** — At least 2 AZs for fault tolerance
3. ✅ **Set appropriate cooldown** — Avoid thrashing
4. ✅ **Use target tracking** — Simpler and more effective than step/simple
5. ✅ **Tag instances** — Propagate tags from ASG for cost tracking
6. ✅ **Use lifecycle hooks** — Gracefully handle scale events
7. ✅ **Warm pool** — Pre-initialize instances for faster scaling
8. ✅ **Instance refresh** — Rolling update instances with new AMI

## Common Interview Questions

1. **Q: Difference between ALB and NLB?**
   - ALB: Layer 7, HTTP/HTTPS, path/host routing, WebSocket
   - NLB: Layer 4, TCP/UDP, ultra-low latency, static IP

2. **Q: What is connection draining?**
   - Time to complete in-flight requests before deregistering a target

3. **Q: How does Auto Scaling decide which instance to terminate?**
   - Default termination policy: AZ with most instances → Oldest launch config → Closest to billing hour

4. **Q: Target tracking vs step scaling?**
   - Target tracking: Set target metric (simpler, recommended)
   - Step scaling: Define thresholds and step adjustments (more control)

5. **Q: How to handle stateful applications with Auto Scaling?**
   - Use sticky sessions on ALB, externalize state to ElastiCache/DynamoDB

6. **Q: What is a warm pool?**
   - Pre-initialized instances in stopped/running state for faster scaling

---

[← Previous: DynamoDB](08-DynamoDB.md) | [Back to Main Guide](README.md) | [Next: CloudWatch →](10-CloudWatch.md)
