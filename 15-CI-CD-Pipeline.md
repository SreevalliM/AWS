# 15. 🔄 CI/CD Pipeline

Automate software delivery process.

## AWS CI/CD Services

### 1. CodeCommit
- Git-based source control
- Fully managed
- Encrypted at rest and transit

### 2. CodeBuild
- Compiles code
- Runs tests
- Produces deployable artifacts

### 3. CodeDeploy
- Automates deployments
- EC2, Lambda, ECS, on-premises
- Blue/green deployments
- Rolling deployments

### 4. CodePipeline
- Orchestrates entire CI/CD workflow
- Integrates all CodeServices

## Pipeline Stages
```
Source (GitHub/CodeCommit)
    ↓
Build (CodeBuild)
    ↓
Test (CodeBuild)
    ↓
Deploy (CodeDeploy)
```

## Example: buildspec.yml
```yaml
version: 0.2

phases:
  install:
    runtime-versions:
      python: 3.11
    commands:
      - pip install -r requirements.txt
      
  pre_build:
    commands:
      - echo "Running tests..."
      - python -m pytest tests/
      
  build:
    commands:
      - echo "Building application..."
      
  post_build:
    commands:
      - echo "Build completed on `date`"

artifacts:
  files:
    - '**/*'
```

---

[← Previous: Infrastructure as Code](14-Infrastructure-as-Code.md) | [Back to Main Guide](README.md) | [Next: Security & Compliance →](16-Security-Compliance.md)
