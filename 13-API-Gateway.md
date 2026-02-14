# 13. 🌐 API Gateway

**Create, publish, and manage APIs at any scale**

## What is API Gateway?

Amazon API Gateway is a fully managed service for creating, deploying, and managing RESTful, HTTP, and WebSocket APIs. It acts as the "front door" for applications to access backend services.

### Key Benefits
- ✅ **Fully managed** — No servers to provision
- ✅ **Scalable** — Handles any traffic level
- ✅ **Secure** — Built-in auth and throttling
- ✅ **Cost-effective** — Pay per API call
- ✅ **Integrated** — Works with Lambda, EC2, any HTTP endpoint

---

## API Types

### 1. REST API
Full-featured API management:

| Feature | Details |
|---------|---------|
| Protocol | HTTP/HTTPS |
| Auth | IAM, Cognito, Lambda Authorizer, API keys |
| Caching | Built-in response caching |
| Throttling | Per-method, per-client |
| Transformations | Request/response model mapping |
| Stages | dev, staging, prod |
| Custom domains | ✅ |
| WAF integration | ✅ |
| Usage plans | ✅ |

### 2. HTTP API
Simpler, faster, cheaper (ideal for most use cases):

| Feature | Details |
|---------|---------|
| Protocol | HTTP/HTTPS |
| Auth | OIDC, OAuth 2.0, JWT, IAM |
| Cost | ~70% cheaper than REST API |
| Latency | ~60% lower latency |
| Auto-deploy | ✅ |
| CORS | Built-in support |

### 3. WebSocket API
Two-way real-time communication:

| Feature | Details |
|---------|---------|
| Protocol | WebSocket (wss://) |
| Use cases | Chat apps, real-time dashboards, gaming |
| Routes | `$connect`, `$disconnect`, `$default`, custom |
| Backend | Lambda, HTTP, AWS services |

### REST API vs HTTP API

| Feature | REST API | HTTP API |
|---------|----------|----------|
| **Cost** | ~$3.50/million | ~$1.00/million |
| **Latency** | Higher | Lower |
| **Caching** | ✅ | ❌ |
| **Request transform** | ✅ | ❌ |
| **Usage plans/API keys** | ✅ | ❌ |
| **WAF** | ✅ | ❌ |
| **Lambda authorizer** | ✅ | ✅ (v2 only) |
| **Private integrations** | ✅ | ✅ |

**Rule of thumb:** Use HTTP API unless you need caching, WAF, transformation, or API keys.

---

## Key Features

### Throttling & Rate Limiting
- **Account default**: 10,000 requests/second, 5,000 burst
- **Stage-level**: Override per stage
- **Method-level**: Override per endpoint
- **Usage plans**: Per-customer throttling with API keys
- Returns `429 Too Many Requests` when exceeded

### Response Caching (REST API only)
- Cache backend responses at the stage level
- TTL: 0–3,600 seconds (default: 300)
- Cache size: 0.5 GB – 237 GB
- Reduces backend calls and improves latency
- Per-method cache invalidation

### Stages & Deployment
```
API Definition
    ↓
Deploy to Stage
    ↓
├── dev   → https://abc123.execute-api.region.amazonaws.com/dev
├── test  → https://abc123.execute-api.region.amazonaws.com/test
└── prod  → https://abc123.execute-api.region.amazonaws.com/prod
```

- **Stage variables**: Environment-specific configuration
- **Canary deployments**: Route percentage of traffic to new version

### Authorization Methods

| Method | How It Works | Use Case |
|--------|-------------|----------|
| **IAM** | AWS SigV4 signatures | AWS users/roles, service-to-service |
| **Cognito** | JWT tokens from User Pool | Mobile/web app users |
| **Lambda Authorizer** | Custom auth logic in Lambda | Custom tokens, OAuth, SAML |
| **API Keys** | x-api-key header | Third-party developer access |

### CORS (Cross-Origin Resource Sharing)
Enable browser-based clients from different domains:

```json
{
  "Access-Control-Allow-Origin": "*",
  "Access-Control-Allow-Methods": "GET, POST, PUT, DELETE, OPTIONS",
  "Access-Control-Allow-Headers": "Content-Type, Authorization"
}
```

---

## Integration Types

### 1. Lambda Proxy (most common)
```
Client → API Gateway → Lambda Function → Response
```
- API Gateway passes entire request to Lambda
- Lambda returns statusCode, headers, body
- No mapping templates needed

### 2. Lambda Custom Integration
- API Gateway transforms request before Lambda
- Mapping templates (Velocity Template Language)
- More control, more complexity

### 3. HTTP / HTTP Proxy
```
Client → API Gateway → External HTTP Endpoint → Response
```
- Proxy existing REST APIs
- Add throttling, auth, caching

### 4. AWS Service Integration
```
Client → API Gateway → DynamoDB / S3 / SQS / SNS → Response
```
- Direct integration without Lambda
- Reduces cost and latency

### 5. Mock Integration
- Return hardcoded response (no backend)
- Useful for testing and API prototyping

---

## ✅ Hands-on Lab: Build REST API

### Lab 1: Simple Lambda-Backed API

**Step 1: Create Lambda Function**
```python
import json

def lambda_handler(event, context):
    method = event['httpMethod']
    path = event['path']
    params = event.get('queryStringParameters') or {}
    body = json.loads(event['body']) if event.get('body') else {}

    name = params.get('name', body.get('name', 'World'))

    return {
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
```

**Step 2: Create REST API**
1. API Gateway Console → **Create API** → **REST API** → Build
2. API name: `HelloAPI`
3. Endpoint type: Regional

**Step 3: Create Resources & Methods**
1. Actions → **Create Resource**: `/hello`
2. Actions → **Create Method**: GET
3. Integration type: **Lambda Function**
4. Lambda Proxy integration: ✅
5. Select your Lambda function

**Step 4: Enable CORS**
1. Select `/hello` resource
2. Actions → **Enable CORS**
3. Confirm defaults → **Enable**

**Step 5: Deploy**
1. Actions → **Deploy API**
2. Stage: **New Stage** → `prod`
3. Copy the **Invoke URL**

**Step 6: Test**
```bash
# GET request
curl "https://YOUR-API-ID.execute-api.region.amazonaws.com/prod/hello?name=AWS"

# POST request
curl -X POST "https://YOUR-API-ID.execute-api.region.amazonaws.com/prod/hello" \
  -H "Content-Type: application/json" \
  -d '{"name": "API Gateway"}'
```

### Lab 2: API with Usage Plan & API Key

**Step 1: Create Usage Plan**
1. API Gateway → **Usage Plans** → **Create**
2. Name: `BasicPlan`
3. Throttling: 100 requests/second, 50 burst
4. Quota: 10,000 requests/month
5. Add API stage: `HelloAPI` / `prod`

**Step 2: Create API Key**
1. API Gateway → **API Keys** → **Create**
2. Name: `developer-key-1`
3. Add to usage plan: `BasicPlan`

**Step 3: Require API Key on Method**
1. Select GET method on `/hello`
2. **Method Request** → **API Key Required**: true
3. Re-deploy API

**Step 4: Test**
```bash
# Without key (403 Forbidden)
curl "https://YOUR-API-ID.execute-api.region.amazonaws.com/prod/hello"

# With key (200 OK)
curl -H "x-api-key: YOUR_API_KEY" \
  "https://YOUR-API-ID.execute-api.region.amazonaws.com/prod/hello"
```

### Lab 3: Custom Domain Name

1. Request ACM certificate for `api.yourdomain.com`
2. API Gateway → **Custom domain names** → **Create**
3. Domain: `api.yourdomain.com`
4. Certificate: Select ACM cert
5. Create API mapping: HelloAPI → prod → /v1
6. Create Route 53 alias record pointing to API Gateway domain
7. Access: `https://api.yourdomain.com/v1/hello`

---

## API Gateway Pricing

| API Type | Cost |
|----------|------|
| REST API | $3.50 per million requests + data transfer |
| HTTP API | $1.00 per million requests + data transfer |
| WebSocket | $1.00 per million messages + connection minutes |
| Caching | $0.02–$3.80/hour depending on cache size |

**Free Tier:** 1 million REST API calls or 1 million HTTP API calls per month (12 months).

---

## API Gateway Best Practices

1. ✅ **Use HTTP API** when possible — cheaper and faster
2. ✅ **Enable caching** for read-heavy APIs (REST API)
3. ✅ **Set throttling** to protect backends
4. ✅ **Use Lambda Proxy** integration for simplicity
5. ✅ **Enable CloudWatch logging** for debugging
6. ✅ **Use stage variables** for environment config
7. ✅ **Validate requests** with request validation models
8. ✅ **Use custom domains** with ACM certificates

## Common Interview Questions

1. **Q: REST API vs HTTP API?**
   - REST: Full features (caching, WAF, transforms), more expensive
   - HTTP: Simpler, 70% cheaper, lower latency, ideal for Lambda proxy

2. **Q: How to secure an API?**
   - IAM auth, Cognito, Lambda authorizer, API keys, WAF, throttling, HTTPS

3. **Q: What is Lambda Proxy integration?**
   - API Gateway passes the entire raw request to Lambda; Lambda returns statusCode, headers, body

4. **Q: How to handle CORS?**
   - Enable CORS on the resource; ensure Lambda returns `Access-Control-Allow-Origin` header

5. **Q: How to version APIs?**
   - Use stages (v1, v2), path-based (`/v1/resource`), or custom domain mappings

---

[← Previous: Lambda](12-Lambda.md) | [Back to Main Guide](README.md) | [Next: Infrastructure as Code →](14-Infrastructure-as-Code.md)
