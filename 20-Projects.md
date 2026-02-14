# 🚀 AWS Hands-on Projects

Practical projects to solidify your AWS knowledge and build your portfolio.

---

## 🎯 Project 1: Static Website with S3 & CloudFront

**Difficulty**: Beginner  
**Duration**: 1-2 hours  
**Cost**: ~$0.50/month

### What You'll Build
Host a static portfolio website on S3 with CloudFront CDN for global delivery.

### Architecture
```
User → CloudFront (CDN) → S3 (Static Website) → Route 53 (DNS)
```

### Step-by-Step

**1. Create S3 Bucket**
- Bucket name: `your-name-portfolio`
- Disable "Block all public access"
- Enable static website hosting

**2. Upload Website Files**
```html
<!-- index.html -->
<!DOCTYPE html>
<html>
<head>
    <title>My AWS Portfolio</title>
    <link rel="stylesheet" href="css/style.css">
</head>
<body>
    <h1>Welcome to My Portfolio</h1>
    <p>Hosted on AWS S3</p>
</body>
</html>
```

```css
/* css/style.css */
body {
    font-family: Arial, sans-serif;
    max-width: 800px;
    margin: 50px auto;
    padding: 20px;
}
```

**3. Configure Bucket Policy**
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Sid": "PublicReadGetObject",
    "Effect": "Allow",
    "Principal": "*",
    "Action": "s3:GetObject",
    "Resource": "arn:aws:s3:::your-name-portfolio/*"
  }]
}
```

**4. Create CloudFront Distribution**
- Origin: S3 website endpoint
- Enable HTTPS
- Set default root object: `index.html`

**5. (Optional) Add Custom Domain**
- Register domain in Route 53
- Create A record pointing to CloudFront
- Add ACM certificate for HTTPS

### What You Learned
✅ S3 static website hosting  
✅ S3 bucket policies  
✅ CloudFront CDN  
✅ HTTPS/SSL certificates

---

## 🎯 Project 2: EC2 Web Server with Load Balancer

**Difficulty**: Intermediate  
**Duration**: 2-3 hours  
**Cost**: Free Tier eligible

### What You'll Build
High-availability web application with multiple EC2 instances behind load balancer.

### Architecture
```
Internet → ALB → Target Group → EC2 (us-east-1a) + EC2 (us-east-1b)
                                        ↓
                                   RDS (MySQL)
```

### Implementation

**1. Create VPC Infrastructure**
```
VPC: 10.0.0.0/16
├── Public Subnet A: 10.0.1.0/24 (us-east-1a)
├── Public Subnet B: 10.0.2.0/24 (us-east-1b)
├── Private Subnet A: 10.0.11.0/24 (us-east-1a)
└── Private Subnet B: 10.0.12.0/24 (us-east-1b)
```

**2. Launch EC2 Instances**

User Data script:
```bash
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd

# Create simple page showing instance info
INSTANCE_ID=$(curl -s http://169.254.169.254/latest/meta-data/instance-id)
AZ=$(curl -s http://169.254.169.254/latest/meta-data/placement/availability-zone)

cat > /var/www/html/index.html <<EOF
<!DOCTYPE html>
<html>
<head>
    <title>AWS Load Balancer Demo</title>
    <style>
        body { font-family: Arial; text-align: center; padding: 50px; }
        .info { background: #f0f0f0; padding: 20px; border-radius: 10px; }
    </style>
</head>
<body>
    <h1>Hello from AWS!</h1>
    <div class="info">
        <h2>Instance ID: $INSTANCE_ID</h2>
        <h3>Availability Zone: $AZ</h3>
    </div>
</body>
</html>
EOF
```

Launch 2 instances in different AZs.

**3. Create Application Load Balancer**
- Type: Application Load Balancer
- Scheme: Internet-facing
- Subnets: Both public subnets
- Security group: HTTP (80) from anywhere

**4. Create Target Group**
- Target type: Instances
- Protocol: HTTP, Port: 80
- Health check: `/index.html`
- Register both EC2 instances

**5. Test High Availability**
- Access ALB DNS name
- Refresh multiple times (see different instances)
- Stop one instance (ALB automatically routes to healthy one)

### What You Learned
✅ VPC with multi-AZ  
✅ Application Load Balancer  
✅ Target groups and health checks  
✅ High availability architecture

---

## 🎯 Project 3: Serverless REST API

**Difficulty**: Intermediate  
**Duration**: 2-3 hours  
**Cost**: Free Tier eligible

### What You'll Build
Complete CRUD API for a todo application using Lambda, API Gateway, and DynamoDB.

### Architecture
```
Client → API Gateway → Lambda Functions → DynamoDB
                            ↓
                       CloudWatch Logs
```

### Implementation

**1. Create DynamoDB Table**
```
Table name: todos
Partition key: todoId (String)
Billing mode: On-demand
```

**2. Create IAM Role for Lambda**
Policy:
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": [
      "dynamodb:PutItem",
      "dynamodb:GetItem",
      "dynamodb:UpdateItem",
      "dynamodb:DeleteItem",
      "dynamodb:Scan"
    ],
    "Resource": "arn:aws:dynamodb:*:*:table/todos"
  },
  {
    "Effect": "Allow",
    "Action": [
      "logs:CreateLogGroup",
      "logs:CreateLogStream",
      "logs:PutLogEvents"
    ],
    "Resource": "*"
  }]
}
```

**3. Create Lambda Function**

```python
import json
import boto3
import uuid
from datetime import datetime
from decimal import Decimal

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('todos')

def lambda_handler(event, context):
    print(f"Event: {json.dumps(event)}")
    
    http_method = event['httpMethod']
    path = event['path']
    
    try:
        if http_method == 'GET' and path == '/todos':
            return get_all_todos()
        elif http_method == 'GET' and '/todos/' in path:
            todo_id = path.split('/')[-1]
            return get_todo(todo_id)
        elif http_method == 'POST' and path == '/todos':
            body = json.loads(event['body'])
            return create_todo(body)
        elif http_method == 'PUT' and '/todos/' in path:
            todo_id = path.split('/')[-1]
            body = json.loads(event['body'])
            return update_todo(todo_id, body)
        elif http_method == 'DELETE' and '/todos/' in path:
            todo_id = path.split('/')[-1]
            return delete_todo(todo_id)
        else:
            return response(404, {'error': 'Not found'})
    except Exception as e:
        print(f"Error: {str(e)}")
        return response(500, {'error': str(e)})

def get_all_todos():
    result = table.scan()
    return response(200, result['Items'])

def get_todo(todo_id):
    result = table.get_item(Key={'todoId': todo_id})
    if 'Item' in result:
        return response(200, result['Item'])
    else:
        return response(404, {'error': 'Todo not found'})

def create_todo(body):
    todo = {
        'todoId': str(uuid.uuid4()),
        'title': body['title'],
        'completed': False,
        'createdAt': datetime.now().isoformat()
    }
    table.put_item(Item=todo)
    return response(201, todo)

def update_todo(todo_id, body):
    table.update_item(
        Key={'todoId': todo_id},
        UpdateExpression='SET title = :title, completed = :completed',
        ExpressionAttributeValues={
            ':title': body.get('title'),
            ':completed': body.get('completed', False)
        }
    )
    return response(200, {'message': 'Updated successfully'})

def delete_todo(todo_id):
    table.delete_item(Key={'todoId': todo_id})
    return response(200, {'message': 'Deleted successfully'})

def response(status_code, body):
    return {
        'statusCode': status_code,
        'headers': {
            'Content-Type': 'application/json',
            'Access-Control-Allow-Origin': '*',
            'Access-Control-Allow-Methods': 'GET,POST,PUT,DELETE,OPTIONS',
           'Access-Control-Allow-Headers': 'Content-Type'
        },
        'body': json.dumps(body, default=str)
    }
```

**4. Create API Gateway**
- Type: REST API
- Create resource: `/todos`
- Create resource: `/todos/{id}`
- Methods with Lambda integration:
  - GET /todos → Lambda
  - POST /todos → Lambda
  - GET /todos/{id} → Lambda
  - PUT /todos/{id} → Lambda
  - DELETE /todos/{id} → Lambda
- Enable CORS
- Deploy API to `prod` stage

**5. Test API**

```bash
# Get all todos
curl https://YOUR-API-ID.execute-api.region.amazonaws.com/prod/todos

# Create todo
curl -X POST https://YOUR-API-ID.execute-api.region.amazonaws.com/prod/todos \
  -H "Content-Type: application/json" \
  -d '{"title": "Learn AWS Lambda"}'

# Get specific todo
curl https://YOUR-API-ID.execute-api.region.amazonaws.com/prod/todos/TODO-ID

# Update todo
curl -X PUT https://YOUR-API-ID.execute-api.region.amazonaws.com/prod/todos/TODO-ID \
  -H "Content-Type: application/json" \
  -d '{"title": "Learn AWS Lambda", "completed": true}'

# Delete todo
curl -X DELETE https://YOUR-API-ID.execute-api.region.amazonaws.com/prod/todos/TODO-ID
```

### What You Learned
✅ AWS Lambda functions  
✅ API Gateway REST API  
✅ DynamoDB operations  
✅ CORS configuration  
✅ Serverless architecture

---

## 🎯 Project 4: CI/CD Pipeline

**Difficulty**: Advanced  
**Duration**: 3-4 hours  
**Cost**: Free Tier eligible

### What You'll Build
Automated deployment pipeline using CodePipeline, CodeBuild, and CodeDeploy.

### Architecture
```
GitHub → CodePipeline → CodeBuild → CodeDeploy → EC2/Auto Scaling Group
                            ↓
                           S3 (Artifacts)
```

### Implementation

**1. Prepare Application Code**

`app.py` (Simple Flask app):
```python
from flask import Flask, jsonify
import os

app = Flask(__name__)

@app.route('/')
def home():
    return jsonify({
        'message': 'Hello from CI/CD Pipeline!',
        'version': os.environ.get('APP_VERSION', '1.0'),
        'environment': os.environ.get('ENVIRONMENT', 'production')
    })

@app.route('/health')
def health():
    return jsonify({'status': 'healthy'})

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=80)
```

`requirements.txt`:
```
Flask==2.3.0
gunicorn==20.1.0
```

`buildspec.yml` (CodeBuild):
```yaml
version: 0.2

phases:
  install:
    runtime-versions:
      python: 3.11
    commands:
      - echo "Installing dependencies..."
      - pip install -r requirements.txt
      
  pre_build:
    commands:
      - echo "Running tests..."
      - python -m pytest tests/ || true
      
  build:
    commands:
      - echo "Building application..."
      - echo "Build completed on `date`"
      
  post_build:
    commands:
      - echo "Creating deployment package..."

artifacts:
  files:
    - '**/*'
  name: my-app-$(date +%Y%m%d-%H%M%S)
```

`appspec.yml` (CodeDeploy):
```yaml
version: 0.0
os: linux

files:
  - source: /
    destination: /var/www/myapp

hooks:
  BeforeInstall:
    - location: scripts/install_dependencies.sh
      timeout: 300
      runas: root
      
  AfterInstall:
    - location: scripts/start_server.sh
      timeout: 300
      runas: root
      
  ApplicationStart:
    - location: scripts/validate_service.sh
      timeout: 300
      runas: root
```

**2. Create scripts/**

`scripts/install_dependencies.sh`:
```bash
#!/bin/bash
yum install -y python3-pip
pip3 install -r /var/www/myapp/requirements.txt
```

`scripts/start_server.sh`:
```bash
#!/bin/bash
cd /var/www/myapp
nohup python3 app.py > /var/log/myapp.log 2>&1 &
```

`scripts/validate_service.sh`:
```bash
#!/bin/bash
sleep 10
curl http://localhost/health || exit 1
```

**3. Setup AWS Resources**

**EC2 Launch Template:**
- Amazon Linux 2
- Instance profile with CodeDeploy permissions
- Security group: HTTP (80)
- User Data: Install CodeDeploy agent

**Auto Scaling Group:**
- Min: 2, Max: 4
- Health check: ELB
- Subnets: Multiple AZs

**Application Load Balancer:**
- Target Group with /health check
- Listener: HTTP:80

**4. Create CodeDeploy Application**
- Compute platform: EC2/On-premises
- Deployment group: Link to Auto Scaling Group
- Deployment configuration: CodeDeployDefault.AllAtOnce

**5. Create CodePipeline**

Stages:
1. **Source**
   - Provider: GitHub
   - Repository: your-repo
   - Branch: main

2. **Build**
   - Provider: CodeBuild
   - Project: Create new with buildspec.yml

3. **Deploy**
   - Provider: CodeDeploy
   - Application: your-app
   - Deployment group: your-deployment-group

**6. Test Pipeline**
- Push code to GitHub
- Watch pipeline execute
- Verify deployment to EC2 instances
- Access ALB to see application

### What You Learned
✅ CI/CD pipeline setup  
✅ CodePipeline orchestration  
✅ CodeBuild for building/testing  
✅ CodeDeploy for deployment  
✅ Blue/green deployments  
✅ Auto Scaling integration

---

## 🎯 Project 5: Three-Tier Web Application

**Difficulty**: Advanced  
**Duration**: 4-5 hours  
**Cost**: ~$5-10/month

### What You'll Build
Production-ready three-tier architecture with web, application, and database layers.

### Architecture
```
Internet → CloudFront → ALB (Public) → Web Tier (EC2)
                                           ↓
                         ALB (Private) → App Tier (EC2)
                                           ↓
                                      RDS (MySQL Multi-AZ)
                                           ↓
                                     ElastiCache (Redis)
```

### Key Components

**Network Layer:**
- VPC with public and private subnets across 2 AZs
- NAT Gateways for outbound internet access
- Security Groups for each tier
- Network ACLs for additional security

**Web Tier:**
- EC2 Auto Scaling Group (2-4 instances)
- Application Load Balancer
- CloudFront distribution
- S3 for static assets

**Application Tier:**
- EC2 Auto Scaling Group (2-6 instances)
- Internal Application Load Balancer
- Lambda for async tasks
- SQS for message queuing

**Database Tier:**
- RDS MySQL Multi-AZ
- Read Replicas for read scaling
- ElastiCache Redis cluster
- S3 for database backups

**Monitoring & Logging:**
- CloudWatch dashboards
- CloudWatch alarms (CPU, memory, requests)
- CloudTrail for API auditing
- VPC Flow Logs

**Security:**
- IAM roles for EC2 instances
- Secrets Manager for credentials
- KMS for encryption
- WAF for application protection

### Implementation Steps (High-Level)

1. **Create VPC Infrastructure** - Subnets, route tables, gateways
2. **Setup Security** - Security groups, NACLs, IAM roles
3. **Deploy Database** - RDS Multi-AZ, read replicas, ElastiCache
4. **Deploy Application Tier** - EC2 Auto Scaling, internal ALB
5. **Deploy Web Tier** - EC2 Auto Scaling, public ALB
6. **Setup CDN** - CloudFront distribution
7. **Configure Monitoring** - CloudWatch, alarms, dashboards
8. **Load Testing** - Verify auto-scaling works
9. **Disaster Recovery** - Cross-region backup, RDS snapshots

### What You Learned
✅ Multi-tier architecture design  
✅ High availability and fault tolerance  
✅ Auto Scaling across tiers  
✅ Database replication strategies  
✅ Caching strategies  
✅ Security best practices  
✅ Monitoring and alerting  
✅ Cost optimization

---

## 📋 Project Checklist

After completing projects, make sure to:

- [ ] Document architecture diagrams
- [ ] Create README with setup instructions
- [ ] Push code to GitHub
- [ ] Add projects to resume/portfolio
- [ ] Include estimated costs
- [ ] List technologies used
- [ ] Explain design decisions
- [ ] Screenshot key components
- [ ] Video demo (optional)
- [ ] Write blog post about learning

---

## 💡 Tips for Projects

1. **Start Simple** - Get basic version working first
2. **Version Control** - Commit changes frequently
3. **Tag Resources** - Use consistent naming and tags
4. **Monitor Costs** - Set billing alerts
5. **Clean Up** - Delete resources when done
6. **Document** - Write clear documentation
7. **Test** - Verify everything works
8. **Iterate** - Improve incrementally
9. **Share** - Show your work (LinkedIn, GitHub)
10. **Learn** - Understand why, not just how

---

## 🎓 Next Steps

After completing these projects:

1. **Add custom features** to make them unique
2. **Combine projects** into larger applications
3. **Explore additional services** (SageMaker, ECS, EKS)
4. **Pursue certification** (Solutions Architect Associate)
5. **Contribute to open source** AWS projects
6. **Build real-world applications** solving actual problems
7. **Share knowledge** through blogs/videos/talks

**Your AWS journey has just begun! Keep building! 🚀**
