# 1. 🌥 Cloud Computing Basics

## What is Cloud Computing?

Cloud computing is the delivery of computing services over the internet ("the cloud"), including servers, storage, databases, networking, software, analytics, and intelligence.

### Key Benefits:
- **Cost Savings**: Pay only for what you use
- **Scalability**: Scale up or down based on demand
- **Performance**: Access to latest technology
- **Speed**: Deploy resources in minutes
- **Global Reach**: Deploy worldwide instantly
- **Reliability**: Data backup and disaster recovery

## Service Models

### IaaS (Infrastructure as a Service)
- **What**: Virtualized computing resources over the internet
- **Examples**: EC2, VPC, EBS
- **Use Case**: Host applications, store data, run workloads
- **Control**: You manage OS, middleware, runtime, data, applications

### PaaS (Platform as a Service)
- **What**: Platform allowing developers to build applications
- **Examples**: Elastic Beanstalk, RDS, Lambda
- **Use Case**: Application development and deployment
- **Control**: You manage data and applications only

### SaaS (Software as a Service)
- **What**: Complete software solution delivered over internet
- **Examples**: Gmail, Salesforce, Office 365
- **Use Case**: End-user applications
- **Control**: Just use the application

## AWS Global Infrastructure

### Regions
- Geographic locations worldwide
- Each region is completely independent
- Currently 30+ regions globally
- Choose based on: latency, cost, compliance, services availability

### Availability Zones (AZs)
- One or more discrete data centers within a region
- Isolated from failures in other AZs
- Connected through high-bandwidth, low-latency networking
- Typically 2-6 AZs per region

### Edge Locations
- Content Delivery Network (CDN) endpoints
- Used by CloudFront for caching
- 400+ edge locations globally

## Shared Responsibility Model

### AWS Responsibility (Security OF the Cloud)
- Physical infrastructure
- Hardware and facilities
- Network infrastructure
- Virtualization layer
- Managed services

### Customer Responsibility (Security IN the Cloud)
- Customer data
- Platform, applications, IAM
- Operating system, network & firewall
- Client-side data encryption
- Server-side encryption
- Network traffic protection

---

[← Back to Main Guide](README.md) | [Next: AWS Account Setup →](02-AWS-Account-Setup.md)
