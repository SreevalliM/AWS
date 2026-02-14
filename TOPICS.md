# 📚 AWS Learning Guide — Complete Topic Index

A detailed index of all 21 topics covered in this learning repository, organized by module and week.

---

## Part 1: Foundation (Week 1)

| # | Topic | File | Key Concepts |
|---|-------|------|--------------|
| 01 | ☁️ Cloud Computing Basics | [01-Cloud-Computing-Basics.md](01-Cloud-Computing-Basics.md) | IaaS / PaaS / SaaS, Regions & AZs, Shared Responsibility Model |
| 02 | 🔧 AWS Account Setup | [02-AWS-Account-Setup.md](02-AWS-Account-Setup.md) | Free Tier, Console tour, Billing alerts, Root account security |
| 03 | 🔐 IAM | [03-IAM.md](03-IAM.md) | Users, Groups, Roles, Policies, MFA, Least Privilege |
| 04 | 💻 EC2 | [04-EC2.md](04-EC2.md) | Instance types, Pricing models, Security Groups, Key Pairs, EBS, User Data |
| 05 | 📦 S3 | [05-S3.md](05-S3.md) | Storage classes, Versioning, Lifecycle rules, Static website hosting, Replication |
| 06 | 🌐 VPC | [06-VPC.md](06-VPC.md) | Subnets, Route Tables, IGW, NAT Gateway, NACLs, VPC Peering, Endpoints |

## Part 2: Database Services (Week 2)

| # | Topic | File | Key Concepts |
|---|-------|------|--------------|
| 07 | 🗄 RDS | [07-RDS.md](07-RDS.md) | Multi-AZ, Read Replicas, Backups, Aurora, RDS Proxy |
| 08 | ⚡ DynamoDB | [08-DynamoDB.md](08-DynamoDB.md) | Partition/Sort keys, GSI/LSI, Capacity modes, Streams, DAX, Global Tables |

## Part 3: Scaling & Monitoring (Week 2)

| # | Topic | File | Key Concepts |
|---|-------|------|--------------|
| 09 | ⚖️ Auto Scaling & Load Balancing | [09-Auto-Scaling-Load-Balancing.md](09-Auto-Scaling-Load-Balancing.md) | ASG, Scaling policies, ALB / NLB / GWLB, Target Groups, Health Checks |
| 10 | 📈 CloudWatch | [10-CloudWatch.md](10-CloudWatch.md) | Metrics, Logs, Alarms, Dashboards, EventBridge, Custom Metrics |
| 11 | 🔍 CloudTrail | [11-CloudTrail.md](11-CloudTrail.md) | API auditing, Management/Data events, Insights, Compliance |

## Part 4: Serverless & DevOps (Week 3)

| # | Topic | File | Key Concepts |
|---|-------|------|--------------|
| 12 | ⚡ Lambda | [12-Lambda.md](12-Lambda.md) | Event-driven, Layers, Versions/Aliases, Cold starts, Pricing |
| 13 | 🌐 API Gateway | [13-API-Gateway.md](13-API-Gateway.md) | REST / HTTP / WebSocket APIs, Throttling, Caching, Authorization |
| 14 | 🏗️ Infrastructure as Code | [14-Infrastructure-as-Code.md](14-Infrastructure-as-Code.md) | CloudFormation, Terraform, Stacks, Change Sets |
| 15 | 🔄 CI/CD Pipeline | [15-CI-CD-Pipeline.md](15-CI-CD-Pipeline.md) | CodeCommit, CodeBuild, CodeDeploy, CodePipeline |

## Part 5: Advanced Topics (Week 4)

| # | Topic | File | Key Concepts |
|---|-------|------|--------------|
| 16 | 🔒 Security & Compliance | [16-Security-Compliance.md](16-Security-Compliance.md) | KMS, Secrets Manager, WAF, Shield, GuardDuty, Inspector, Config |
| 17 | 💰 Cost Optimization | [17-Cost-Optimization.md](17-Cost-Optimization.md) | Right-sizing, Reserved/Spot, Cost Explorer, Budgets, Trusted Advisor |
| 18 | 🏛️ Well-Architected Framework | [18-Well-Architected-Framework.md](18-Well-Architected-Framework.md) | 5 Pillars: Operational Excellence, Security, Reliability, Performance, Cost |
| 19 | 🆘 Disaster Recovery | [19-Disaster-Recovery.md](19-Disaster-Recovery.md) | RTO/RPO, Backup & Restore, Pilot Light, Warm Standby, Active/Active |

## Part 6: Practice & Career

| # | Topic | File | Key Concepts |
|---|-------|------|--------------|
| 20 | 🚀 Hands-on Projects | [20-Projects.md](20-Projects.md) | 5 real-world projects (Beginner → Advanced) |
| 21 | 🎯 Interview Preparation | [21-Interview-Prep.md](21-Interview-Prep.md) | Core Q&A, Scenarios, Architecture design, Behavioral questions |

---

## 🎯 Day-by-Day Learning Path

### Week 1: Foundations (Topics 1–6)
| Day | Topics | Activities |
|-----|--------|------------|
| 1–2 | Cloud Basics + Account Setup | Create AWS account, enable billing alerts, explore console |
| 3–4 | IAM + EC2 | Create IAM users/roles/policies, launch EC2, SSH into instance |
| 5–6 | S3 + VPC | Host static website on S3, build custom VPC with public/private subnets |
| 7 | **Mini Project 1** | Deploy a web server on EC2 inside a custom VPC |

### Week 2: Databases & Monitoring (Topics 7–11)
| Day | Topics | Activities |
|-----|--------|------------|
| 8–9 | RDS + DynamoDB | Launch MySQL RDS, create DynamoDB table with GSI, test CRUD |
| 10–11 | Auto Scaling + CloudWatch | Create ASG with ALB, set up CloudWatch alarms and dashboards |
| 12–13 | CloudTrail + Review | Enable CloudTrail, review logs, revisit weak areas |
| 14 | **Mini Project 2** | Build a load-balanced, auto-scaling web app with monitoring |

### Week 3: Serverless & DevOps (Topics 12–15)
| Day | Topics | Activities |
|-----|--------|------------|
| 15–16 | Lambda + API Gateway | Create Lambda functions, build REST API with API Gateway |
| 17 | **Mini Project 3** | Serverless CRUD API (Lambda + API Gateway + DynamoDB) |
| 18–19 | IaC + CI/CD | Write CloudFormation template, set up CodePipeline |
| 20–21 | **Mini Project 4** | Deploy app via CI/CD pipeline |

### Week 4: Advanced & Career (Topics 16–21)
| Day | Topics | Activities |
|-----|--------|------------|
| 22–23 | Security + Cost Optimization | Review KMS, WAF, GuardDuty; analyze costs |
| 24–25 | Well-Architected + DR | Study 5 pillars, design DR strategy |
| 26–27 | Interview Prep + Architecture Design | Practice scenario questions, draw architecture diagrams |
| 28–30 | **Capstone Project** | Three-tier application (Project 5 in [20-Projects.md](20-Projects.md)) |

---

## 💡 Study Tips

- ✅ **Hands-on practice** is crucial — don't just read
- ✅ **Use Free Tier** to avoid costs
- ✅ **Set billing alerts** to monitor spending
- ✅ **Take notes** and create your own documentation
- ✅ **Join AWS communities** (Reddit, Discord, forums)
- ✅ **Watch re:Invent sessions** on YouTube
- ✅ **Build projects** to demonstrate skills

---

## 🔗 Useful Resources

### Official AWS
- [AWS Documentation](https://docs.aws.amazon.com/)
- [AWS Free Tier](https://aws.amazon.com/free/)
- [AWS Training & Certification](https://aws.amazon.com/training/)
- [AWS Skill Builder](https://skillbuilder.aws/)
- [AWS Well-Architected Labs](https://wellarchitectedlabs.com/)
- [AWS Workshops](https://workshops.aws/)

### Community
- [AWS SubReddit](https://reddit.com/r/aws)
- [AWS Discord Communities](https://discord.gg/aws)
- [A Cloud Guru](https://acloudguru.com/)
- [AWS Architecture Center](https://aws.amazon.com/architecture/)

### Certifications (Recommended Order)
1. **AWS Certified Cloud Practitioner** — Entry level
2. **AWS Certified Solutions Architect Associate** — Most popular
3. **AWS Certified Developer Associate** — For developers
4. **AWS Certified SysOps Administrator Associate** — For operations

---

## 🎓 After Completing This Guide

You'll be ready to:
- ✅ Build production-ready AWS infrastructure
- ✅ Design scalable and highly available architectures
- ✅ Pass AWS certification exams
- ✅ Work on cloud projects professionally
- ✅ Understand AWS billing and cost optimization
- ✅ Implement security best practices
- ✅ Create Infrastructure as Code
- ✅ Set up CI/CD pipelines

**Good luck on your AWS journey! ☁️**
