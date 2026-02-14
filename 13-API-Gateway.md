# 13. 🌐 API Gateway

Create, publish, and manage APIs at any scale.

## API Types

### 1. REST API
- RESTful HTTP API
- Full API management features
- Custom domain names
- Request/response transformations

### 2. HTTP API
- Simpler, faster, cheaper
- 70% cheaper than REST API
- OIDC and OAuth 2.0 support
- Basic features only

### 3. WebSocket API
- Two-way communication
- Real-time applications
- Chat, gaming, trading

## Key Features
- **Throttling**: Rate limiting (10,000 req/sec default)
- **Caching**: Reduce backend calls
- **Authorization**: IAM, Cognito, Lambda authorizer
- **Stages**: dev, test, prod environments
- **API Keys**: Identify third-party developers
- **Usage Plans**: Throttle and quota limits per customer

## Integration Types

**Lambda Integration**:
```
Client → API Gateway → Lambda → Response
```

**HTTP Integration**:
```
Client → API Gateway → HTTP Endpoint → Response
```

**AWS Service Integration**:
```
Client → API Gateway → DynamoDB/S3/SNS → Response
```

## Example: Create REST API
```bash
# Create REST API
aws apigateway create-rest-api \
  --name "My API" \
  --description "My REST API"

# Create Lambda integration
aws apigateway put-integration \
  --rest-api-id abc123 \
  --resource-id xyz789 \
  --http-method GET \
  --type AWS_PROXY \
  --integration-http-method POST \
  --uri arn:aws:apigateway:region:lambda:path/2015-03-31/functions/arn:aws:lambda:region:account:function:my-function/invocations
```

---

[← Previous: Lambda](12-Lambda.md) | [Back to Main Guide](README.md) | [Next: Infrastructure as Code →](14-Infrastructure-as-Code.md)
