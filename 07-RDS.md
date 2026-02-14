# 7. 🗄 RDS (Relational Database Service)

**Managed relational database service**

![RDS Dashboard](https://docs.aws.amazon.com/images/AmazonRDS/latest/UserGuide/images/preview-environment-dashboard.png)

![RDS Architecture](https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2024/01/11/RDS_Blog_1-Page-2.png)

![RDS Read Replicas](https://docs.aws.amazon.com/images/AmazonRDS/latest/UserGuide/images/read-and-standby-replica.png)

![RDS Multi-AZ](https://miro.medium.com/1%2ANMt7vXbqH34hNJLHVU2d-A.png)

## What is RDS?

Amazon Relational Database Service makes it easy to set up, operate, and scale a relational database in the cloud. AWS handles routine tasks like provisioning, patching, backup, recovery,failure detection, and repair.

### Supported Database Engines
- ✅ **Amazon Aurora** (MySQL/PostgreSQL compatible)
- ✅ **MySQL**
- ✅ **PostgreSQL**
- ✅ **MariaDB**
- ✅ **Oracle**
- ✅ **Microsoft SQL Server**

### Key Features
- Automated backups
- Software patching
- Automatic failure detection
- Point-in-time recovery
- Monitoring and metrics
- Read replicas for scaling

## RDS vs EC2 Database

| Feature | RDS | Self-managed on EC2 |
|---------|-----|---------------------|
| **Setup** | Automated | Manual |
| **Patching** | Automatic | You handle |
| **Backups** | Automatic | Manual setup |
| **Scaling** | Easy | Manual |
| **HA/DR** | Built-in options | Manual setup |
| **Cost** | Higher per hour | Lower, but more effort |
| **Control** | Limited | Full control |

## Multi-AZ Deployment

**High Availability** solution for production databases.

### How it Works
1. Primary DB in one AZ
2. Synchronous replication to standby in different AZ
3. Automatic failover on failure (60-120 seconds)
4. Same endpoint (DNS automatically switches)

### Benefits
- ✅ Automatic failover
- ✅ No manual intervention
- ✅ Enhanced durability
- ✅ Zero data loss (synchronous replication)

### Use Cases
- Production databases
- Applications requiring high availability
- Compliance requirements

**Important:** Standby is NOT used for read traffic (only for failover)

## Read Replicas

**Scale read operations** by creating read-only copies.

### How it Works
1. Master DB handles writes
2. Asynchronous replication to read replicas
3. Read replicas handle read queries
4. Can promote to standalone DB

### Benefits
- ✅ Improve read performance
- ✅ Separate read/write workloads
- ✅ Disaster recovery (cross-region)
- ✅ Reporting/analytics workload

### Configurations
- Up to 15 read replicas
- Same region or cross-region
- Cross-AZ within region (free replication)
- Can have read replicas of read replicas

### Multi-AZ vs Read Replicas

| Feature | Multi-AZ | Read Replicas |
|---------|----------|---------------|
| **Purpose** | High Availability | Read Scaling |
| **Replication** | Synchronous | Asynchronous |
| **Endpoints** | One (automatic failover) | Multiple |
| **Writes** | Primary only | Primary only |
| **Reads** | Primary only | Primary + replicas |
| **Failover** | Automatic | Manual promotion |
| **Cost** | ~2x instance cost | Per replica |

## RDS Backups

### Automated Backups
- **Daily** full backup
- **Transaction logs** backed up every 5 minutes
- **Retention**: 0-35 days (0 = disabled)
- **Point-in-time recovery** (PITR) to any second
- **Storage**: Stored in S3 (you don't see it)
- **Performance**: Brief I/O suspension (Multi-AZ: from standby)

### Manual Snapshots
- **User-initiated** backups
- **No retention limit** (kept until deleted)
- **Use case**: Before major changes, long-term archival
- **Can share** with other AWS accounts
- **Can copy** to other regions

### Backup Best Practices
1. ✅ Enable automated backups (7-35 days retention)
2. ✅ Take manual snapshot before major changes
3. ✅ Copy snapshots to another region (DR)
4. ✅ Test restore procedures regularly
5. ✅ Use Multi-AZ for production

## RDS Storage

### Storage Types

| Type | Description | IOPS | Use Case |
|------|-------------|------|----------|
| **General Purpose (gp3)** | Balanced price/performance | 3,000-16,000 | Most workloads |
| **General Purpose (gp2)** | Legacy SSD | 3-16,000 | Legacy apps |
| **Provisioned IOPS (io1)** | High performance | 40,000+ | I/O intensive |
| **Magnetic** | Legacy HDD | Low | Compatibility only |

### Storage Auto-Scaling
- Automatically increases storage when:
  - Free space < 10%
  - Low storage lasts 5 minutes
  - 6 hours since last modification
- Set **Maximum storage threshold**
- No downtime

## RDS Security

### Network Security
- **VPC**: Launch in private subnet
- **Security Groups**: Control access
- **No SSH access** (managed service)

### Encryption
**At-rest encryption:**
- Uses AWS KMS
- Must enable at creation
- Includes data, automated backups, snapshots, replicas
- **Cannot enable on existing  DB** (must create encrypted copy)

**In-transit encryption:**
- SSL/TLS certificates
- Force SSL for PostgreSQL:
  ```sql
  ALTER DATABASE mydb SET rds.force_ssl = 1;
  ```

### Access Control
- **IAM Policies**: Control who can manage RDS
- **Database users**: Standard DB authentication
- **IAM Database Authentication**: MySQL/PostgreSQL only
  - No passwords, use auth token
  - Token valid for 15 minutes

### Auditing
- **CloudWatch Logs**: Error logs, slow query logs
- **CloudTrail**: Management API calls
- **Database Audit Logs**: Engine-specific

## RDS Performance Insights

Monitor database performance and analyze queries.

### Features
- Dashboard showing database load
- Top queries consuming resources
- Wait events analysis
- Historical data (up to 2 years)
- Free for 7 days retention

### Metrics
- **DB Load**: Number of active sessions
- **Top SQL**: Resource-consuming queries
- **Wait Events**: What's blocking queries

## ✅ Hands-on Lab: Create RDS Database

### Lab 1: Launch MySQL Database

**Step 1: Create DB Subnet Group**
1. **RDS Console** → **Subnet groups** → **Create**
2. Name: `my-db-subnet-group`
3. VPC: Select your VPC
4. Add subnets: Select private subnets from 2+ AZs
5. **Create**

**Step 2: Create Security Group**
1. EC2 → Security Groups → Create
2. Name: `rds-mysql-sg`
3. VPC: Your VPC
4. Inbound: MySQL/Aurora (3306) from web-server-sg
5. **Create**

**Step 3: Create RDS Instance**
1. **RDS Console** → **Create database**
2. **Creation method**: Standard create
3. **Engine**: MySQL
4. **Version**: Latest
5. **Templates**: Free tier (for learning)

**Settings:**
- DB instance identifier: `my-mysql-db`
- Master username: `admin`
- Master password: Create strong password

**Instance configuration:**
- Burstable classes: db.t3.micro (Free Tier)

**Storage:**
- gp3, 20 GB
- Disable storage autoscaling (for Free Tier)

**Connectivity:**
- VPC: Your VPC
- Subnet group: `my-db-subnet-group`
- Public access: No
- VPC security group: `rds-mysql-sg`
- Availability Zone: No preference

**Database authentication:**
- Password authentication

**Additional configuration:**
- Initial database name: `myappdb`
- Backup retention: 7 days
- Enable encryption
- Enable CloudWatch logs: Error log, Slow query log

6. **Create database**
7. Wait ~10 minutes for available status

**Step 4: Connect from EC2**

SSH into EC2 instance in same VPC:

```bash
# Install MySQL client
sudo yum install -y mysql

# Connect to RDS
mysql -h <rds-endpoint> -u admin -p

# After entering password:
SHOW DATABASES;
USE myappdb;

# Create table
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

# Insert data
INSERT INTO users (name, email) VALUES ('John Doe', 'john@example.com');

# Query data
SELECT * FROM users;

# Exit
EXIT;
```

### Lab 2: Create Read Replica

1. **RDS Console** → Select your DB
2. **Actions** → **Create read replica**
3. DB instance identifier: `my-mysql-db-replica`
4. Destination region: Same region
5. Availability Zone: Different from primary
6. **Create**

**Test read replica:**
```bash
# Connect to read replica
mysql -h <replica-endpoint> -u admin -p

# Can read
SELECT * FROM users;

# Cannot write (error)
INSERT INTO users (name, email) VALUES ('Test', 'test@test.com');
```

### Lab 3: Enable Multi-AZ

1. **RDS Console** → Select your DB
2. **Modify**
3. **Multi-AZ deployment**: Yes
4. **Continue** → **Modify DB instance**
5. Apply: During the next maintenance window (or immediately)

### Lab 4: Create Manual Snapshot

1. Select database
2. **Actions** → **Take snapshot**
3. Snapshot name: `my-mysql-db-snapshot-2026-02-14`
4. **Take snapshot**

**Restore from snapshot:**
1. **Snapshots** → Select snapshot
2. **Actions** → **Restore snapshot**
3. New DB identifier: `my-mysql-db-restored`
4. Configure settings
5. **Restore**

### Lab 5: Point-in-Time Recovery

1. Select database
2. **Actions** → **Restore to point in time**
3. Choose: Latest restorable time (or custom)
4. DB instance identifier: `my-mysql-db-pitr`
5. **Restore**

## Amazon Aurora

**AWS's high-performance MySQL/PostgreSQL-compatible database**

### Key Features
- 5x faster than MySQL, 3x faster than PostgreSQL
- Up to 64TB auto-scaling storage
- 15 read replicas
- Sub-10ms replica lag
- Point-in-time recovery(continuous backup to S3)
- Multi-master option

### Aurora Serverless
- **Auto-scaling** based on demand
- **Pay per second** (no minimum)
- **Use case**: Infrequent, intermittent workloads
- **ACU**: Aurora Capacity Units (compute + memory)

### Aurora Global Database
- **Cross-region** replication
- **< 1 second** replication lag
- **Up to 5 secondary regions**
- **Use case**: DR, global applications

## RDS Proxy

Connection pooling service for RDS/Aurora.

### Benefits
- ✅ Improve app scalability (connection pooling)
- ✅ Reduce failover time by 66%
- ✅ IAM authentication support
- ✅ No code changes required

### Use Cases
- Lambda functions connecting to RDS
- Applications opening many connections
- Failover time reduction

## Common RDS Interview Questions

1. **Q: RDS vs DynamoDB?**
   - RDS: Relational, ACID, complex queries, vertical scaling
   - DynamoDB: NoSQL, key-value, simple queries, horizontal scaling

2. **Q: Multi-AZ vs Read Replicas?**
   - Multi-AZ: HA/failover, synchronous, automated failover
   - Read Replicas: Read scaling, asynchronous, manual promotion

3. **Q: Can you SSH into RDS?**
   - No, it's a managed service (no OS access)

4. **Q: How to encrypt existing unencrypted RDS?**
   - Create encrypted snapshot → Restore to new encrypted DB → Switch app

5. **Q: Maximum backup retention?**
   - Automated: 35 days
   - Manual snapshots: Indefinite

---

[← Previous: VPC](06-VPC.md) | [Back to Main Guide](README.md) | [Next: DynamoDB →](08-DynamoDB.md)
