# 12. ⚡ Lambda (Serverless Computing)

**Run code without managing servers**

![Lambda Storage](https://d2908q01vomqb2.cloudfront.net/da4b9237bacccdf19c0760cab7aec4a8359010b0/2022/01/28/2022-aws-lambda-ephemeral-storage-2-new.png)

![Lambda Code Editor](https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2024/10/21/Viewing-function-code-in-the-Lambda-Code-Editor.png)

![Lambda S3 Trigger](https://docs.aws.amazon.com/images/lambda/latest/dg/images/services-s3-example/s3trigger_tut_steps2.png)

![Lambda S3 Configuration](https://docs.aws.amazon.com/images/lambda/latest/dg/images/services-s3-example/s3_tut_config.png)

## What is AWS Lambda?

AWS Lambda is a **serverless compute service** that runs your code in response to events and automatically manages the computing resources. You only pay for compute time consumed.

### Key Features
- ✅ **No servers to manage**
- ✅ **Auto-scaling**
- ✅ **Pay per request** and compute time
- ✅ **Event-driven**
- ✅ **Supports multiple languages**
- ✅ **Built-in fault tolerance**

### Benefits
- **Cost-effective**: Pay only when code runs
- **Zero administration**: No infrastructure management
- **Automatic scaling**: From few requests to thousands
- **Fast deployment**: Deploy code in seconds
- **Integrated security**: IAM-based permissions

## How Lambda Works

```
Event Source → Triggers → Lambda Function → Output/Action
```

**Example Flow:**
1. User uploads image to S3
2. S3 triggers Lambda function
3. Lambda resizes image
4. Lambda saves thumbnail to S3
5. Lambda records metadata in DynamoDB

## Supported Languages

### Native Support
- **Python** 3.8, 3.9, 3.10, 3.11, 3.12
- **Node.js** 16.x, 18.x, 20.x
- **Java** 8, 11, 17, 21
- **Go** 1.x
- **.NET** Core 3.1, 6, 7
- **Ruby** 2.7, 3.2

### Custom Runtimes
- Use Lambda Runtime API
- Run any language (Rust, Elixir, etc.)
- Package as Lambda Layer

## Lambda Function Structure

### Python Example
```python
def lambda_handler(event, context):
    """
    event: Input data (JSON)
    context: Runtime information
    """
    print(f"Received event: {event}")
    
    # Your logic here
    name = event.get('name', 'World')
    message = f"Hello, {name}!"
    
    return {
        'statusCode': 200,
        'body': message
    }
```

### Node.js Example
```javascript
exports.handler = async (event) => {
    console.log('Received event:', event);
    
    const name = event.name || 'World';
    const message = `Hello, ${name}!`;
    
    return {
        statusCode: 200,
        body: JSON.stringify({ message })
    };
};
```

## Lambda Components

### 1. Function Code
Your application logic

### 2. Event Source
What triggers the function:
- API Gateway (REST API)
- S3 (file upload/delete)
- DynamoDB Streams
- SNS/SQS
- CloudWatch Events/EventBridge
- ALB (Application Load Balancer)
- Cognito
- Kinesis
- And 20+ more...

### 3. Execution Role
IAM role with permissions:
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": [
      "logs:CreateLogGroup",
      "logs:CreateLogStream",
      "logs:PutLogEvents",
      "s3:GetObject",
      "s3:PutObject"
    ],
    "Resource": "*"
  }]
}
```

### 4. Configuration

**Memory**: 128 MB to 10,240 MB (10 GB)
- More memory = More CPU
- More memory = Higher cost

**Timeout**: 1 second to 15 minutes (900 seconds)
- Default: 3 seconds
- Set based on expected execution time

**Ephemeral Storage**: 512 MB to 10,240 MB
- `/tmp` directory
- Temporary file storage
- Cleared between invocations

**Environment Variables**:
```
DB_HOST=mydb.amazonaws.com
API_KEY=secret123
ENVIRONMENT=production
```

## Lambda Limits

### Per Function
- **Memory**: 128 MB - 10 GB
- **Timeout**: 15 minutes max
- **Deployment package**: 50 MB (zipped), 250 MB (unzipped)
- **Environment variables**: 4 KB
- **Ephemeral storage (/tmp)**: 512 MB - 10 GB
- **Concurrent executions**: 1000 (soft limit, can increase)

### Per Region
- **Concurrent executions**: 1000 (default)
- **Function storage**: 75 GB

## Lambda Pricing

### Free Tier (per month)
- 1 million requests
- 400,000 GB-seconds of compute time

### After Free Tier
- **Requests**: $0.20 per 1 million requests
- **Compute**: $0.0000166667 per GB-second

### Cost Examples

**Example 1**: 1 million requests, 128 MB, 100ms each
```
Compute: 1M * 0.128 GB * 0.1s = 12,800 GB-seconds
Cost: 12,800 * $0.0000166667 = $0.21
Requests: FREE (within 1M free tier)
Total: $0.21/month
```

**Example 2**: 10 million requests, 512 MB, 500ms each
```
Compute: 10M * 0.512 GB * 0.5s = 2,560,000 GB-seconds
Cost: 2,560,000 * $0.0000166667 = $42.67
Requests: 9M (after 1M free) * $0.20/1M = $1.80
Total: $44.47/month
```

## Lambda Event Sources

### Synchronous (Wait for response)
- **API Gateway**: REST API requests
- **ALB**: Load balancer forwarding
- **Cognito**: User authentication
- **Lex**: Chatbot interactions
- **Alexa**: Voice commands
- **CloudFront**: Lambda@Edge

### Asynchronous (Fire and forget)
- **S3**: Object created/deleted
- **SNS**: Message published
- **EventBridge**: Scheduled/custom events
- **CodeCommit**: Git push
- **CloudFormation**: Custom resources

### Stream-based (Process records)
- **DynamoDB Streams**: Table changes
- **Kinesis Data Streams**: Real-time data
- **SQS**: Queue messages

## Lambda Layers

Reusable components shared across functions.

### Use Cases
- Common dependencies
- Custom runtimes
- Shared configuration
- Libraries

### Benefits
- ✅ Reduce deployment package size
- ✅ Share code across functions
- ✅ Separate dependencies from code

### Example Structure
```
my-lambda-function/
├── lambda_function.py      (your code)
└── layers/
    └── my-layer/
        └── python/
            └── requests/   (shared library)
```

**Create Layer:**
```bash
# Package layer
cd python
pip install requests -t .
cd ..
zip -r layer.zip python/

# Create layer
aws lambda publish-layer-version \
  --layer-name my-dependencies \
  --zip-file fileb://layer.zip \
  --compatible-runtimes python3.11
```

## Lambda Versions & Aliases

### Versions
- **Immutable** snapshots of function
- Each version has unique ARN
- $LATEST = current editable version

### Aliases
- **Pointers** to specific versions
- Mutable (can update)
- Enable blue/green deployments

**Example:**
```
Function: my-lambda
├── $LATEST (version 5)
├── Version 1 ← Alias: PROD (80%)
└── Version 2 ← Alias: PROD (20%)  [Blue/Green deployment]
    Alias: DEV → Version 2
```

## Lambda Environment Variables

Pass configuration without hardcoding.

### Set Variables
```python
import os

def lambda_handler(event, context):
    db_host = os.environ['DB_HOST']
    api_key = os.environ['API_KEY']
    
    # Use variables
    return f"Connecting to {db_host}"
```

### Encryption
- Encrypted at rest by default (AWS KMS)
- Use AWS Secrets Manager for sensitive data

## Lambda Best Practices

### Performance
1. ✅ **Minimize package size** - Faster cold starts
2. ✅ **Reuse connections** - Database connections, HTTP clients
3. ✅ **Use environment variables** - Avoid hardcoding
4. ✅ **Optimize memory** - More memory = more CPU
5. ✅ **Take advantage of /tmp** - Cache data between invocations

### Security
1. ✅ **Least privilege IAM role** - Minimum permissions
2. ✅ **Secrets Manager** - Store sensitive data
3. ✅ **VPC configuration** - Access private resources securely
4. ✅ **Environment variable encryption** - Use KMS
5. ✅ **Code signing** - Verify code integrity

### Cost Optimization  
1. ✅ **Right-size memory** - Balance performance/cost
2. ✅ **Reduce execution time** - Optimize code
3. ✅ **Use appropriate timeout** - Don't over-allocate
4. ✅ **Monitor invocations** - Track unused functions
5. ✅ **Consider Compute Savings Plans** - For consistent workloads

### Code Structure
```python
# ❌ Bad: Initialize inside handler
def lambda_handler(event, context):
    dynamodb = boto3.resource('dynamodb')  # Called every invocation
    table = dynamodb.Table('Users')
    # ...

# ✅ Good: Initialize outside handler
import boto3
dynamodb = boto3.resource('dynamodb')  # Reused across invocations
table = dynamodb.Table('Users')

def lambda_handler(event, context):
    # Use pre-initialized resources
    # ...
```

## ✅ Hands-on Labs

### Lab 1: Create Simple Lambda

**Step 1: Create Function**
1. **Lambda Console** → **Create function**
2. Choose **Author from scratch**
3. Function name: `HelloWorldFunction`
4. Runtime: **Python 3.11**
5. **Create function**

**Step 2: Write Code**
```python
import json

def lambda_handler(event, context):
    print(f"Event: {json.dumps(event)}")
    
    name = event.get('name', 'World')
    
    return {
        'statusCode': 200,
        'body': json.dumps({
            'message': f'Hello, {name}!',
            'event_received': event
        })
    }
```

**Step 3: Test**
1. Click **Test**
2. Event name: `TestEvent`
3. Event JSON:
```json
{
  "name": "AWS Lambda"
}
```
4. **Save** and **Test**

**Step 4: View Logs**
- Monitor tab → View CloudWatch Logs

### Lab 2: Lambda with S3 Trigger

**Step 1: Create Lambda**
```python
import json
import boto3

s3 = boto3.client('s3')

def lambda_handler(event, context):
    # Get S3 event details
    bucket = event['Records'][0]['s3']['bucket']['name']
    key = event['Records'][0]['s3']['object']['key']
    
    print(f"File uploaded: s3://{bucket}/{key}")
    
    # Get object metadata
    response = s3.head_object(Bucket=bucket, Key=key)
    size = response['ContentLength']
    
    print(f"File size: {size} bytes")
    
    return {
        'statusCode': 200,
        'body': json.dumps(f'Processed {key}')
    }
```

**Step 2: Add S3 Permission to Lambda Role**
1. Configuration → Permissions
2. Click role name
3. Add policy: `AmazonS3ReadOnlyAccess`

**Step 3: Add S3 Trigger**
1. Add trigger → S3
2. Bucket: Select your bucket
3. Event type: `All object create events`
4. **Add**

**Step 4: Test**
1. Upload file to S3 bucket
2. Check Lambda CloudWatch Logs

### Lab 3: Lambda with API Gateway

**Step 1: Create Lambda**
```python
import json

def lambda_handler(event, context):
    # Get HTTP method and path
    method = event['httpMethod']
    path = event['path']
    
    # Get query parameters
    params = event.get('queryStringParameters', {})
    name = params.get('name', 'Guest') if params else 'Guest'
    
    # Response
    response = {
        'statusCode': 200,
        'headers': {
            'Content-Type': 'application/json',
            'Access-Control-Allow-Origin': '*'
        },
        'body': json.dumps({
            'message': f'Hello, {name}!',
            'method': method,
            'path': path
        })
    }
    
    return response
```

**Step 2: Create API Gateway**
1. Add trigger → API Gateway
2. Create API: REST API
3. Security: Open
4. **Add**
5. Note API endpoint URL

**Step 3: Test**
```bash
curl "https://YOUR-API-ID.execute-api.region.amazonaws.com/default/HelloWorldFunction?name=John"
```

### Lab 4: Lambda with DynamoDB

**Step 1: Create DynamoDB Table**
```
Table: users
Partition Key: userId (String)
```

**Step 2: Create Lambda CRUD Functions**

```python
import json
import boto3
from decimal import Decimal

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('users')

def lambda_handler(event, context):
    operation = event['operation']
    
    if operation == 'create':
        table.put_item(Item=event['payload'])
        return {'statusCode': 200, 'body': 'User created'}
        
    elif operation == 'read':
        response = table.get_item(Key={'userId': event['userId']})
        return {
            'statusCode': 200,
            'body': json.dumps(response.get('Item'), default=str)
        }
        
    elif operation == 'update':
        table.update_item(
            Key={'userId': event['userId']},
            UpdateExpression='SET #n = :name, email = :email',
            ExpressionAttributeNames={'#n': 'name'},
            ExpressionAttributeValues={
                ':name': event['name'],
                ':email': event['email']
            }
        )
        return {'statusCode': 200, 'body': 'User updated'}
        
    elif operation == 'delete':
        table.delete_item(Key={'userId': event['userId']})
        return {'statusCode': 200, 'body': 'User deleted'}
        
    elif operation == 'list':
        response = table.scan()
        return {
            'statusCode': 200,
            'body': json.dumps(response['Items'], default=str)
        }
```

**Step 3: Add DynamoDB Permissions**
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": [
      "dynamodb:GetItem",
      "dynamodb:PutItem",
      "dynamodb:UpdateItem",
      "dynamodb:DeleteItem",
      "dynamodb:Scan"
    ],
    "Resource": "arn:aws:dynamodb:region:account-id:table/users"
  }]
}
```

**Step 4: Test Events**

Create:
```json
{
  "operation": "create",
  "payload": {
    "userId": "user001",
    "name": "John Doe",
    "email": "john@example.com"
  }
}
```

Read:
```json
{
  "operation": "read",
  "userId": "user001"
}
```

### Lab 5: Scheduled Lambda (CloudWatch Events)

**Step 1: Create Lambda**
```python
import json
from datetime import datetime

def lambda_handler(event, context):
    current_time = datetime.now().strftime('%Y-%m-%d %H:%M:%S')
    print(f"Lambda executed at: {current_time}")
    
    # Your scheduled task here
    # E.g., cleanup old data, send reports, etc.
    
    return {
        'statusCode': 200,
        'body': json.dumps(f'Task completed at {current_time}')
    }
```

**Step 2: Add EventBridge Trigger**
1. Add trigger → EventBridge
2. Create new rule
3. Rule name: `DailyTask`
4. Schedule expression: `rate(1 day)` or `cron(0 9 * * ? *)`  # 9 AM daily
5. **Add**

## Lambda Cold Starts

### What is Cold Start?
First invocation or after idle period requires initialization:
1. Download code
2. Start execution environment
3. Initialize runtime
4. Run initialization code

### Cold Start Latency
- **Python**: ~100-300ms
- **Node.js**: ~100-300ms
- **Java**: ~1-3 seconds
- **.NET**: ~1-3 seconds

### Reduce Cold Starts
1. ✅ **Provisioned Concurrency** - Keep instances warm
2. ✅ **Minimize package size** - Faster download
3. ✅ **Choose Python/Node** - Faster runtime init
4. ✅ **Keep functions warm** - CloudWatch Events ping
5. ✅ **Optimize initialization** - Lazy load libraries

## Lambda vs EC2 vs Fargate

| Feature | Lambda | EC2 | Fargate |
|---------|--------|-----|---------|
| **Management** | Serverless | You manage | Managed containers |
| **Scaling** | Automatic | Manual/Auto Scaling | Automatic |
| **Pricing** | Per request | Per hour | Per task-hour |
| **Duration** | 15 min max | Unlimited | Unlimited |
| **Use Case** | Event-driven, short tasks | Long-running, custom | Containerized apps |
| **Cold Start** | Yes | No | Yes |

## Common Interview Questions

1. **Q: What is AWS Lambda?**
   - Serverless compute service, runs code in response to events, no server management

2. **Q: Lambda limitations?**
   - 15 min timeout, 10 GB memory, 250 MB deployment package

3. **Q: How to handle long-running tasks?**
   - Use Step Functions, ECS, EC2 for tasks > 15 minutes

4. **Q: What is cold start?**
   - Initialization delay for first invocation or after idle period

5. **Q: Lambda pricing model?**
   - Pay per request and GB-second of compute time

6. **Q: How to secure Lambda?**
   - IAM roles, VPC, encryption, Secrets Manager, least privilege

7. **Q: Lambda vs EC2?**
   - Lambda: Serverless, auto-scale, short tasks, event-driven
   - EC2: Full control, long tasks, persistent connections

8. **Q: Can Lambda access VPC resources?**
   - Yes, configure VPC settings (needs ENI setup)

---

[← Previous: CloudTrail](11-CloudTrail.md) | [Back to Main Guide](README.md) | [Next: API Gateway →](13-API-Gateway.md)
