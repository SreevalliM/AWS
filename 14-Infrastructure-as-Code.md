# 14. 🏗️ Infrastructure as Code (IaC)

**Define, provision, and manage infrastructure using declarative code**

## What is Infrastructure as Code?

IaC treats infrastructure the same way developers treat application code — version-controlled, tested, repeatable, and automated. Instead of clicking through the console, you describe your desired state in a file and let the tool create it.

### Key Benefits
- ✅ **Repeatability** — Deploy identical environments every time
- ✅ **Version control** — Track changes with Git
- ✅ **Speed** — Spin up entire stacks in minutes
- ✅ **Consistency** — Eliminate configuration drift
- ✅ **Documentation** — Code IS the documentation
- ✅ **Cost** — Automate teardown of unused resources

---

## AWS CloudFormation

AWS's native IaC service. Define resources in JSON or YAML templates.

### Core Concepts

| Concept | Description |
|---------|-------------|
| **Template** | JSON/YAML file describing resources |
| **Stack** | A collection of AWS resources created from a template |
| **Change Set** | Preview changes before applying |
| **Drift Detection** | Detect manual changes to stack resources |
| **Nested Stacks** | Templates referencing other templates |
| **StackSets** | Deploy stacks across multiple accounts/regions |

### Template Structure

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: My application infrastructure

# Input parameters
Parameters:
  Environment:
    Type: String
    Default: dev
    AllowedValues: [dev, staging, prod]
  InstanceType:
    Type: String
    Default: t3.micro

# Conditional resource creation
Conditions:
  IsProd: !Equals [!Ref Environment, prod]

# Reusable values
Mappings:
  RegionAMI:
    us-east-1:
      AMI: ami-0c55b159cbfafe1f0
    us-west-2:
      AMI: ami-0d6621c01e8c2de2c

# AWS resources to create
Resources:
  WebSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Allow HTTP and SSH
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0

  WebServer:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: !FindInMap [RegionAMI, !Ref 'AWS::Region', AMI]
      InstanceType: !Ref InstanceType
      SecurityGroupIds:
        - !Ref WebSecurityGroup
      Tags:
        - Key: Name
          Value: !Sub '${Environment}-web-server'
        - Key: Environment
          Value: !Ref Environment
      UserData:
        Fn::Base64: !Sub |
          #!/bin/bash
          yum update -y
          yum install -y httpd
          systemctl start httpd
          systemctl enable httpd
          echo "<h1>${Environment} Server</h1>" > /var/www/html/index.html

  # Only create in production
  WebServerAlarm:
    Type: AWS::CloudWatch::Alarm
    Condition: IsProd
    Properties:
      AlarmDescription: High CPU for web server
      MetricName: CPUUtilization
      Namespace: AWS/EC2
      Statistic: Average
      Period: 300
      EvaluationPeriods: 2
      Threshold: 80
      ComparisonOperator: GreaterThanThreshold
      Dimensions:
        - Name: InstanceId
          Value: !Ref WebServer

# Output values
Outputs:
  WebServerPublicIP:
    Description: Public IP of web server
    Value: !GetAtt WebServer.PublicIp
    Export:
      Name: !Sub '${Environment}-WebServerIP'
  WebServerURL:
    Description: URL of web server
    Value: !Sub 'http://${WebServer.PublicDnsName}'
```

### Intrinsic Functions

| Function | Purpose | Example |
|----------|---------|---------|
| `!Ref` | Reference parameter or resource | `!Ref MyInstance` |
| `!GetAtt` | Get resource attribute | `!GetAtt MyInstance.PublicIp` |
| `!Sub` | String substitution | `!Sub '${Environment}-bucket'` |
| `!Join` | Join strings | `!Join ['-', [my, app]]` |
| `!Select` | Select from list | `!Select [0, [a, b, c]]` |
| `!FindInMap` | Lookup value from Mappings | `!FindInMap [Map, Key, Value]` |
| `!If` | Conditional value | `!If [IsProd, t3.large, t3.micro]` |
| `!ImportValue` | Import from another stack | `!ImportValue SharedVPC-ID` |

### Stack Operations

```bash
# Validate template
aws cloudformation validate-template \
  --template-body file://template.yaml

# Create stack
aws cloudformation create-stack \
  --stack-name my-app-stack \
  --template-body file://template.yaml \
  --parameters ParameterKey=Environment,ParameterValue=prod \
  --capabilities CAPABILITY_IAM

# Create change set (preview changes)
aws cloudformation create-change-set \
  --stack-name my-app-stack \
  --change-set-name my-changes \
  --template-body file://template-v2.yaml

# Execute change set
aws cloudformation execute-change-set \
  --change-set-name my-changes \
  --stack-name my-app-stack

# Update stack
aws cloudformation update-stack \
  --stack-name my-app-stack \
  --template-body file://template-v2.yaml

# Delete stack (removes all resources)
aws cloudformation delete-stack --stack-name my-app-stack

# Detect drift
aws cloudformation detect-stack-drift --stack-name my-app-stack
```

### Rollback Behaviour
- **Create fails** → Automatic rollback (delete all resources)
- **Update fails** → Automatic rollback to previous state
- **Disable rollback** → Keep failed resources for debugging

---

## Terraform (Alternative IaC Tool)

Open-source, multi-cloud IaC by HashiCorp. Uses HCL (HashiCorp Configuration Language).

### Why Terraform?
- ✅ **Multi-cloud** — AWS, Azure, GCP, etc.
- ✅ **State management** — Tracks what exists
- ✅ **Plan before apply** — Preview changes
- ✅ **Modules** — Reusable components
- ✅ **Large community** — Thousands of providers and modules

### Terraform vs CloudFormation

| Feature | CloudFormation | Terraform |
|---------|---------------|-----------|
| **Cloud support** | AWS only | Multi-cloud |
| **Language** | JSON / YAML | HCL |
| **State** | Managed by AWS | You manage (.tfstate) |
| **Drift detection** | Built-in | `terraform plan` |
| **Rollback** | Automatic | Manual (apply previous) |
| **Cost** | Free | Free (open-source) |
| **Modules** | Nested stacks | Terraform Registry |
| **Learning curve** | Moderate | Moderate |

### Terraform Workflow

```
terraform init    → Download providers and modules
terraform plan    → Preview what will change
terraform apply   → Create/update infrastructure
terraform destroy → Delete all resources
```

### Terraform Example

```hcl
# providers.tf
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }

  backend "s3" {
    bucket = "my-terraform-state"
    key    = "prod/terraform.tfstate"
    region = "us-east-1"
  }
}

provider "aws" {
  region = var.aws_region
}

# variables.tf
variable "aws_region" {
  default = "us-east-1"
}

variable "environment" {
  type    = string
  default = "dev"
}

variable "instance_type" {
  type    = string
  default = "t3.micro"
}

# main.tf
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
  tags = {
    Name        = "${var.environment}-vpc"
    Environment = var.environment
  }
}

resource "aws_subnet" "public" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.1.0/24"
  map_public_ip_on_launch = true
  tags = {
    Name = "${var.environment}-public-subnet"
  }
}

resource "aws_instance" "web" {
  ami           = data.aws_ami.amazon_linux.id
  instance_type = var.instance_type
  subnet_id     = aws_subnet.public.id

  user_data = <<-EOF
    #!/bin/bash
    yum update -y
    yum install -y httpd
    systemctl start httpd
    echo "<h1>Hello from Terraform!</h1>" > /var/www/html/index.html
  EOF

  tags = {
    Name        = "${var.environment}-web-server"
    Environment = var.environment
  }
}

data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]
  filter {
    name   = "name"
    values = ["al2023-ami-*-x86_64"]
  }
}

# outputs.tf
output "instance_ip" {
  value = aws_instance.web.public_ip
}

output "instance_url" {
  value = "http://${aws_instance.web.public_dns}"
}
```

```bash
terraform init
terraform plan -var="environment=prod"
terraform apply -var="environment=prod" -auto-approve
terraform destroy -auto-approve
```

---

## AWS CDK (Cloud Development Kit)

Define infrastructure using **programming languages** (TypeScript, Python, Java, C#, Go).

```typescript
import * as cdk from 'aws-cdk-lib';
import * as ec2 from 'aws-cdk-lib/aws-ec2';

export class MyStack extends cdk.Stack {
  constructor(scope: cdk.App, id: string) {
    super(scope, id);

    const vpc = new ec2.Vpc(this, 'MyVpc', {
      maxAzs: 2,
      natGateways: 1,
    });

    new ec2.Instance(this, 'WebServer', {
      vpc,
      instanceType: ec2.InstanceType.of(ec2.InstanceClass.T3, ec2.InstanceSize.MICRO),
      machineImage: ec2.MachineImage.latestAmazonLinux2023(),
    });
  }
}
```

```bash
cdk init app --language typescript
cdk synth     # Generates CloudFormation template
cdk diff      # Preview changes
cdk deploy    # Deploy stack
cdk destroy   # Delete stack
```

---

## ✅ Hands-on Lab: IaC

### Lab 1: Deploy VPC + EC2 with CloudFormation

**Step 1: Create Template** — Save as `web-stack.yaml`
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: Web server with VPC

Parameters:
  KeyName:
    Type: AWS::EC2::KeyPair::KeyName

Resources:
  VPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
      Tags:
        - Key: Name
          Value: cf-vpc

  PublicSubnet:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: 10.0.1.0/24
      MapPublicIpOnLaunch: true

  InternetGateway:
    Type: AWS::EC2::InternetGateway

  AttachGateway:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      VpcId: !Ref VPC
      InternetGatewayId: !Ref InternetGateway

  RouteTable:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref VPC

  PublicRoute:
    Type: AWS::EC2::Route
    DependsOn: AttachGateway
    Properties:
      RouteTableId: !Ref RouteTable
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId: !Ref InternetGateway

  SubnetRouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref PublicSubnet
      RouteTableId: !Ref RouteTable

  WebSG:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Allow HTTP
      VpcId: !Ref VPC
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0

  WebServer:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: t2.micro
      ImageId: ami-0c55b159cbfafe1f0
      SubnetId: !Ref PublicSubnet
      SecurityGroupIds: [!Ref WebSG]
      KeyName: !Ref KeyName

Outputs:
  URL:
    Value: !Sub 'http://${WebServer.PublicDnsName}'
```

**Step 2: Deploy**
```bash
aws cloudformation create-stack \
  --stack-name web-stack \
  --template-body file://web-stack.yaml \
  --parameters ParameterKey=KeyName,ParameterValue=my-key
```

**Step 3: Verify** in CloudFormation console → Outputs tab

**Step 4: Clean Up**
```bash
aws cloudformation delete-stack --stack-name web-stack
```

### Lab 2: Drift Detection

1. Deploy a stack with CloudFormation
2. Manually change a resource in the console (e.g., modify Security Group)
3. CloudFormation → Select stack → **Detect drift**
4. View **Drift results** — see differences

---

## IaC Best Practices

1. ✅ **Version control everything** — Git for all templates
2. ✅ **Use parameters** — Don't hardcode values
3. ✅ **Small, composable stacks** — Easier to manage
4. ✅ **Use change sets** — Always preview before updating
5. ✅ **Tag all resources** — Cost tracking and organization
6. ✅ **Store state remotely** (Terraform) — S3 + DynamoDB locking
7. ✅ **Use modules** — DRY (Don't Repeat Yourself)
8. ✅ **Lint & validate** — Catch errors before deploying

## Common Interview Questions

1. **Q: CloudFormation vs Terraform?**
   - CF: AWS-native, free, automatic rollback, managed state
   - TF: Multi-cloud, HCL syntax, you manage state, larger ecosystem

2. **Q: What happens if a CloudFormation stack creation fails?**
   - All resources created so far are rolled back (deleted) by default.

3. **Q: What is a Change Set?**
   - A preview of what CloudFormation will change before you apply it.

4. **Q: How to handle secrets in IaC?**
   - Use SSM Parameter Store or Secrets Manager; reference via `!Ref` or `dynamic` blocks; never hardcode.

5. **Q: What is drift detection?**
   - Identifies resources that were changed manually outside of CloudFormation.

---

[← Previous: API Gateway](13-API-Gateway.md) | [Back to Main Guide](README.md) | [Next: CI/CD Pipeline →](15-CI-CD-Pipeline.md)
