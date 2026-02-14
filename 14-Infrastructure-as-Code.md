# 14. 🏗️ Infrastructure as Code

Define infrastructure using declarative code.

## AWS CloudFormation

### Stack Components
- **Template**: JSON/YAML file defining resources
- **Stack**: Collection of AWS resources
- **Change Sets**: Preview changes before applying

### Example Template (YAML)
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: Simple EC2 instance

Parameters:
  KeyName:
    Type: AWS::EC2::KeyPair::KeyName
    Description: EC2 Key Pair

Resources:
  MyEC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-0c55b159cbfafe1f0
      InstanceType: t2.micro
      KeyName: !Ref KeyName
      Tags:
        - Key: Name
          Value: MyServer

Outputs:
  InstanceId:
    Description: Instance ID
    Value: !Ref MyEC2Instance
  PublicIP:
    Description: Public IP
    Value: !GetAtt MyEC2Instance.PublicIp
```

### Deploy Stack
```bash
aws cloudformation create-stack \
  --stack-name my-stack \
  --template-body file://template.yaml \
  --parameters ParameterKey=KeyName,ParameterValue=mykey
```

## Terraform (Alternative)

```hcl
# main.tf
provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"

  tags = {
    Name = "MyServer"
  }
}

output "instance_ip" {
  value = aws_instance.web.public_ip
}
```

```bash
terraform init
terraform plan
terraform apply
```

---

[← Previous: API Gateway](13-API-Gateway.md) | [Back to Main Guide](README.md) | [Next: CI/CD Pipeline →](15-CI-CD-Pipeline.md)
