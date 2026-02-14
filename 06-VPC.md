# 6. 🌐 VPC (Virtual Private Cloud)

**Your own private network in AWS**

![VPC Peering](https://docs.aws.amazon.com/images/prescriptive-guidance/latest/integrate-third-party-services/images/p2_vpc-peering.png)

![VPC Private Subnets](https://docs.aws.amazon.com/images/vpc/latest/userguide/images/vpc-example-private-subnets.png)

![VPC Architecture](https://miro.medium.com/v2/resize%3Afit%3A1200/1%2Agftv4LSqU_12kRqNwYISJw.png)

![Internet Gateway](https://docs.aws.amazon.com/images/vpc/latest/userguide/images/internet-gateway-basics.png)

## What is VPC?

Amazon Virtual Private Cloud lets you provision a logically isolated section of AWS where you can launch resources in a virtual network that you define.

### Key Features
- ✅ Complete control over network environment
- ✅ IP address range selection (CIDR)
- ✅ Subnets creation
- ✅ Route tables and gateways
- ✅ Network security control
- ✅ Hardware VPN connection

## VPC Components

### 1. CIDR Block (IP Range)
Define IP address range for your VPC.

**Examples:**
- `10.0.0.0/16` - 65,536 IPs (typical)
- `172.16.0.0/16` - 65,536 IPs
- `192.168.0.0/24` - 256 IPs

**Private IP Ranges (RFC 1918):**
- `10.0.0.0/8` - 10.0.0.0 to 10.255.255.255
- `172.16.0.0/12` - 172.16.0.0 to 172.31.255.255
- `192.168.0.0/16` - 192.168.0.0 to 192.168.255.255

### 2. Subnets
Subdivide VPC into smaller networks.

**Public Subnet:**
- Has route to Internet Gateway
- Resources get public IP
- Use for: Web servers, load balancers

**Private Subnet:**
- No direct internet access
- Resources only have private IP
- Use for: Databases, application servers

**Example Layout:**
```
VPC: 10.0.0.0/16
├── Public Subnet A: 10.0.1.0/24 (us-east-1a)
├── Public Subnet B: 10.0.2.0/24 (us-east-1b)
├── Private Subnet A: 10.0.11.0/24 (us-east-1a)
└── Private Subnet B: 10.0.12.0/24 (us-east-1b)
```

### 3. Route Tables
Control traffic routing within VPC.

**Main Route Table** (default):
```
Destination: 10.0.0.0/16 → Target: local
```

**Public Route Table:**
```
Destination: 10.0.0.0/16 → Target: local
Destination: 0.0.0.0/0 → Target: igw-xxxxx
```

**Private Route Table:**
```
Destination: 10.0.0.0/16 → Target: local
Destination: 0.0.0.0/0 → Target: nat-xxxxx
```

### 4. Internet Gateway (IGW)
Enables communication between VPC and internet.

- One IGW per VPC
- Horizontally scaled, redundant
- No bandwidth constraints
- Supports IPv4 and IPv6

### 5. NAT Gateway
Enables private subnet resources to access internet (outbound only).

**Use Cases:**
- Software updates
- API calls to external services
- Download dependencies

**NAT Gateway vs NAT Instance:**

| Feature | NAT Gateway | NAT Instance |
|---------|-------------|--------------|
| **Managed** | Yes (AWS) | No (you manage) |
| **Availability** | HA within AZ | Manual HA setup |
| **Bandwidth** | Up to 45 Gbps | Depends on instance type |
| **Cost** | Per hour + data | EC2 instance cost |
| **Security Groups** | No | Yes |

### 6. Security Groups
Virtual firewalls for EC2 instances (stateful).

**Characteristics:**
- Apply at instance level
- Stateful (return traffic automatically allowed)
- Support allow rules only
- Evaluate all rules before deciding

**Example:**
```
Name: web-server-sg
Inbound:
- HTTP (80) from 0.0.0.0/0
- HTTPS (443) from 0.0.0.0/0
- SSH (22) from MyIP

Outbound:
- All traffic (default)
```

### 7. Network ACLs (NACLs)
Stateless firewall for subnets.

**Characteristics:**
- Apply at subnet level
- Stateless (must allow return traffic explicitly)
- Support allow and deny rules
- Rules processed in order

**Security Groups vs NACLs:**

| Feature | Security Group | NACL |
|---------|----------------|------|
| **Level** | Instance | Subnet |
| **State** | Stateful | Stateless |
| **Rules** | Allow only | Allow & Deny |
| **Processing** | All rules | In order (rule #) |
| **Default** | Deny all inbound | Allow all |

### 8. VPC Peering
Connect two VPCs privately.

**Features:**
- Same or different regions
- Same or different accounts
- Non-overlapping CIDR blocks required
- No transitive peering

**Example:**
```
VPC A (10.0.0.0/16) ←peer→ VPC B (172.16.0.0/16)
```

### 9. VPC Endpoints
Private connection to AWS services without internet.

**Types:**

**Interface Endpoint (PrivateLink):**
- ENI with private IP
- Most AWS services
- Cost per hour

**Gateway Endpoint:**
- Route table target
- S3 and DynamoDB only
- Free!

## VPC Flow Logs

Capture IP traffic information.

**Levels:**
- VPC level
- Subnet level
- Network interface level

**Destinations:**
- CloudWatch Logs
- S3
- Kinesis Data Firehose

**Use Cases:**
- Troubleshoot connectivity
- Security analysis
- Cost optimization

## ✅ Hands-on Lab: Create Custom VPC

### Lab: Build Complete VPC Infrastructure

**Step 1: Create VPC**
1. Go to **VPC Console** → **Create VPC**
2. Choose **VPC and more**
3. Settings:
   - Name: `my-app-vpc`
   - IPv4 CIDR: `10.0.0.0/16`
   - AZs: 2
   - Public subnets: 2
   - Private subnets: 2
   - NAT gateways: 1 per AZ
   - VPC endpoints: S3 Gateway
4. Click **Create VPC**

This creates:
- 1 VPC
- 4 Subnets (2 public, 2 private)
- 1 Internet Gateway
- 2 NAT Gateways
- 4 Route Tables
- 1 S3 Gateway Endpoint

**Step 2: Manual VPC Creation (Alternative)**

**2.1: Create VPC**
1. **VPC Console** → **Your VPCs** → **Create VPC**
2. Name: `my-custom-vpc`
3. IPv4 CIDR: `10.0.0.0/16`
4. Click **Create VPC**

**2.2: Create Subnets**
1. **Subnets** → **Create subnet**
2. VPC: `my-custom-vpc`
3. Create 4 subnets:

   **Public Subnet 1:**
   - Name: `public-subnet-1a`
   - AZ: us-east-1a
   - CIDR: `10.0.1.0/24`

   **Public Subnet 2:**
   - Name: `public-subnet-1b`
   - AZ: us-east-1b
   - CIDR: `10.0.2.0/24`

   **Private Subnet 1:**
   - Name: `private-subnet-1a`
   - AZ: us-east-1a
   - CIDR: `10.0.11.0/24`

   **Private Subnet 2:**
   - Name: `private-subnet-1b`
   - AZ: us-east-1b
   - CIDR: `10.0.12.0/24`

**2.3: Create Internet Gateway**
1. **Internet Gateways** → **Create**
2. Name: `my-igw`
3. Click **Create**
4. Select IGW → **Actions** → **Attach to VPC**
5. Choose `my-custom-vpc`

**2.4: Create NAT Gateway**
1. **NAT Gateways** → **Create**
2. Name: `my-nat-gateway`
3. Subnet: `public-subnet-1a`
4. **Allocate Elastic IP**
5. Click **Create**

**2.5: Configure Route Tables**

**Public Route Table:**
1. **Route Tables** → **Create**
2. Name: `public-rt`
3. VPC: `my-custom-vpc`
4. Click **Create**
5. Select → **Routes** → **Edit routes**
6. Add route:
   - Destination: `0.0.0.0/0`
   - Target: Internet Gateway (my-igw)
7. **Subnet Associations** → **Edit**
8. Associate both public subnets

**Private Route Table:**
1. Create route table: `private-rt`
2. Add route:
   - Destination: `0.0.0.0/0`
   - Target: NAT Gateway (my-nat-gateway)
3. Associate both private subnets

**Step 3: Create Security Groups**

**Web Server Security Group:**
1. **Security Groups** → **Create**
2. Name: `web-sg`
3. VPC: `my-custom-vpc`
4. Inbound rules:
   - HTTP (80) from 0.0.0.0/0
   - HTTPS (443) from 0.0.0.0/0
   - SSH (22) from My IP
5. Create

**Database Security Group:**
1. Name: `db-sg`
2. VPC: `my-custom-vpc`
3. Inbound rules:
   - MySQL/Aurora (3306) from web-sg
4. Create

**Step 4: Launch EC2 in Custom VPC**
1. Launch EC2 instance
2. Network: Select `my-custom-vpc`
3. Subnet: Choose `public-subnet-1a`
4. Auto-assign Public IP: Enable
5. Security Group: `web-sg`
6. Launch

**Step 5: Test Connectivity**
```bash
# SSH into instance
ssh -i mykey.pem ec2-user@<public-ip>

# Test internet access
ping -c 3 google.com

# Check routing
ip route

# View network interfaces
ifconfig
```

### Lab: Configure VPC Flow Logs

1. **VPC** → Select your VPC
2. **Flow logs** → **Create flow log**
3. Settings:
   - Filter: All
   - Destination: Send to CloudWatch Logs
   - IAM role: Create new role
4. **Create**

5. View logs:
   - CloudWatch → Log groups
   - Search for traffic patterns

## Common VPC Scenarios

### Scenario 1: Public Web Application
```
Internet → IGW → Public Subnet → EC2 (web server)
```

### Scenario 2: Private Database
```
EC2 (web) in Public Subnet → Private Subnet → RDS
```

### Scenario 3: Outbound Internet from Private
```
Private Subnet → NAT Gateway → IGW → Internet
```

### Scenario 4: Cross-VPC Communication
```
VPC A → VPC Peering → VPC B
```

## VPC Best Practices

1. ✅ **Plan CIDR carefully** - Can't change after creation
2. ✅ **Use multiple AZs** - High availability
3. ✅ **Public/private separation** - Security layers
4. ✅ **Security group naming** - Clear, descriptive names
5. ✅ **VPC Flow Logs** - Enable for troubleshooting
6. ✅ **NAT Gateway redundancy** - One per AZ
7. ✅ **Use VPC endpoints** - Save costs, improve security
8. ✅ **Tag everything** - Resource organization

## Common VPC Interview Questions

1. **Q: Difference between Security Group and NACL?**
   - SG: Instance-level, stateful, allow only
   - NACL: Subnet-level, stateless, allow/deny

2. **Q: How to provide internet access to private subnet?**
   - NAT Gateway in public subnet + route in private RT

3. **Q: What is VPC peering?**
   - Private connection between two VPCs using AWS network

4. **Q: Can you change VPC CIDR block?**
   - Can add additional CIDR blocks, but can't modify primary

---

[← Previous: S3](05-S3.md) | [Back to Main Guide](README.md) | [Next: RDS →](07-RDS.md)
