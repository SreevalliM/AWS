# 15. 🔄 CI/CD Pipeline

**Automate building, testing, and deploying software**

## What is CI/CD?

| Term | Full Form | Meaning |
|------|-----------|---------|
| **CI** | Continuous Integration | Automatically build & test on every code commit |
| **CD** | Continuous Delivery | Automatically prepare releases (manual deploy approval) |
| **CD** | Continuous Deployment | Automatically deploy every passing build to production |

### Why CI/CD?
- ✅ **Faster releases** — Deploy in minutes, not days
- ✅ **Fewer bugs** — Catch issues early with automated tests
- ✅ **Consistency** — Same process every time
- ✅ **Developer productivity** — Focus on code, not deployments
- ✅ **Rollback** — Quickly revert bad deployments

---

## AWS CI/CD Services

### 1. CodeCommit
Fully managed **Git-based source control**.

- Private Git repositories
- Encrypted at rest and in transit
- IAM-based access control
- Integrates with CodePipeline, CodeBuild
- Pull requests and code review
- Similar to GitHub/GitLab (AWS-native)

> **Note:** AWS announced CodeCommit is no longer accepting new customers (2024). Consider GitHub, GitLab, or Bitbucket as alternatives.

### 2. CodeBuild
Fully managed **build service**.

- Compiles code, runs tests, produces artifacts
- Scales automatically (no build queue)
- Pay-per-minute pricing
- Docker support (custom build environments)
- Caches dependencies for faster builds

**Build Environment:** Managed images for common runtimes (Python, Node.js, Java, Go, .NET, Docker)

### 3. CodeDeploy
Fully managed **deployment service**.

**Deployment Targets:**
- EC2 instances
- On-premises servers
- Lambda functions
- ECS services

**Deployment Strategies:**

| Strategy | How It Works | Downtime | Rollback |
|----------|-------------|----------|----------|
| **In-Place** (Rolling) | Updates instances one by one | Minimal | Redeploy previous |
| **Blue/Green** | Create new set, switch traffic | Zero | Switch back instantly |
| **Canary** | Route % of traffic to new version | Zero | Route back |
| **Linear** | Gradually shift traffic over time | Zero | Route back |

### 4. CodePipeline
Fully managed **CI/CD orchestrator**.

- Connects Source → Build → Test → Deploy
- Visual pipeline editor
- Manual approval stages
- Parallel actions
- Cross-region/cross-account deployments

---

## Pipeline Architecture

```
┌────────────┐     ┌────────────┐     ┌────────────┐     ┌────────────┐
│   Source    │ ──→ │   Build    │ ──→ │    Test    │ ──→ │   Deploy   │
│            │     │            │     │            │     │            │
│ GitHub     │     │ CodeBuild  │     │ CodeBuild  │     │ CodeDeploy │
│ CodeCommit │     │ Jenkins    │     │ (test stage)│     │ ECS        │
│ S3         │     │            │     │            │     │ Lambda     │
│ Bitbucket  │     │            │     │            │     │ S3         │
└────────────┘     └────────────┘     └────────────┘     └────────────┘
                                            ↓
                                    ┌────────────┐
                                    │  Approval  │
                                    │  (Manual)  │
                                    └────────────┘
```

---

## buildspec.yml (CodeBuild)

Controls what CodeBuild does during each build phase.

```yaml
version: 0.2

env:
  variables:
    ENV: "production"
  parameter-store:
    DB_PASSWORD: "/myapp/db/password"
  secrets-manager:
    API_KEY: "myapp/api-key:api_key"

phases:
  install:
    runtime-versions:
      python: 3.11
      nodejs: 18
    commands:
      - echo "Installing dependencies..."
      - pip install -r requirements.txt
      - npm install

  pre_build:
    commands:
      - echo "Running lint..."
      - python -m flake8 src/
      - echo "Running unit tests..."
      - python -m pytest tests/unit/ --junitxml=reports/unit.xml

  build:
    commands:
      - echo "Building application..."
      - python -m build
      - echo "Running integration tests..."
      - python -m pytest tests/integration/ --junitxml=reports/integration.xml

  post_build:
    commands:
      - echo "Build completed on $(date)"
      - echo "Creating deployment package..."

reports:
  pytest_reports:
    files:
      - 'reports/*.xml'
    file-format: JUNITXML

artifacts:
  files:
    - '**/*'
  discard-paths: no
  name: build-$(date +%Y%m%d-%H%M%S)

cache:
  paths:
    - '/root/.cache/pip/**/*'
    - 'node_modules/**/*'
```

---

## appspec.yml (CodeDeploy)

Controls what CodeDeploy does during deployment to EC2.

```yaml
version: 0.0
os: linux

files:
  - source: /
    destination: /var/www/myapp
    overwrite: true

permissions:
  - object: /var/www/myapp
    owner: ec2-user
    group: ec2-user
    mode: 755

hooks:
  BeforeInstall:
    - location: scripts/stop_server.sh
      timeout: 120
      runas: root

  AfterInstall:
    - location: scripts/install_dependencies.sh
      timeout: 300
      runas: root

  ApplicationStart:
    - location: scripts/start_server.sh
      timeout: 120
      runas: root

  ValidateService:
    - location: scripts/health_check.sh
      timeout: 60
      runas: root
```

**Lifecycle event order (EC2):**
```
ApplicationStop → DownloadBundle → BeforeInstall → Install →
AfterInstall → ApplicationStart → ValidateService
```

---

## ✅ Hands-on Lab: Build CI/CD Pipeline

### Lab 1: CodeBuild Project

**Step 1: Create S3 Bucket for Artifacts**
```bash
aws s3 mb s3://my-build-artifacts-$(date +%s)
```

**Step 2: Create CodeBuild Project**
1. CodeBuild → **Create build project**
2. Project name: `my-app-build`
3. Source: GitHub (connect your repo)
4. Environment:
   - Managed image → Amazon Linux 2 → Standard → Latest
   - Runtime: Standard
   - Service role: New role
5. Buildspec: Use buildspec.yml in source
6. Artifacts: S3 bucket
7. **Create build project**

**Step 3: Start Build**
1. Click **Start build**
2. Watch build phases: INSTALL → PRE_BUILD → BUILD → POST_BUILD
3. Check **Build logs** for output
4. Check S3 for artifacts

### Lab 2: Full CodePipeline

**Step 1: Prepare Application**

Create a simple app with these files:
- `app.py` (Flask app)
- `requirements.txt`
- `buildspec.yml`
- `appspec.yml`
- `scripts/install_dependencies.sh`
- `scripts/start_server.sh`
- `scripts/health_check.sh`

**Step 2: Create Pipeline**
1. CodePipeline → **Create pipeline**
2. Pipeline name: `my-app-pipeline`

**Stage 1 — Source:**
- Provider: GitHub (v2)
- Repository: your-repo
- Branch: main
- Detection: CodeStarConnections

**Stage 2 — Build:**
- Provider: CodeBuild
- Project: `my-app-build`

**Stage 3 — Approval (optional):**
- Provider: Manual approval
- SNS topic for notification

**Stage 4 — Deploy:**
- Provider: CodeDeploy
- Application: `my-app`
- Deployment group: `my-app-prod`

3. **Create pipeline**

**Step 3: Test Pipeline**
```bash
# Push code change
git add . && git commit -m "trigger pipeline" && git push

# Watch pipeline progress in console:
# Source ✅ → Build ✅ → Approval (waiting) → Deploy ✅
```

### Lab 3: Blue/Green Deployment

1. CodeDeploy → Create application
2. Create deployment group:
   - Type: Blue/Green
   - Environment: Auto Scaling Group
   - Load balancer: your ALB
   - Reroute traffic: Immediately
   - Original instances: Terminate after 15 minutes
3. Deploy — old instances (blue) remain until new ones (green) pass health checks
4. Traffic switches automatically

---

## CI/CD with Docker & ECR

```yaml
# buildspec.yml for Docker builds
version: 0.2

env:
  variables:
    REPO_URI: "123456789012.dkr.ecr.us-east-1.amazonaws.com/my-app"

phases:
  pre_build:
    commands:
      - echo "Logging in to ECR..."
      - aws ecr get-login-password | docker login --username AWS --password-stdin $REPO_URI
      - IMAGE_TAG=$(echo $CODEBUILD_RESOLVED_SOURCE_VERSION | cut -c 1-7)

  build:
    commands:
      - echo "Building Docker image..."
      - docker build -t $REPO_URI:$IMAGE_TAG .
      - docker tag $REPO_URI:$IMAGE_TAG $REPO_URI:latest

  post_build:
    commands:
      - echo "Pushing Docker image..."
      - docker push $REPO_URI:$IMAGE_TAG
      - docker push $REPO_URI:latest
      - printf '[{"name":"my-app","imageUri":"%s"}]' $REPO_URI:$IMAGE_TAG > imagedefinitions.json

artifacts:
  files:
    - imagedefinitions.json
```

---

## CI/CD Best Practices

1. ✅ **Automate everything** — Build, test, deploy, rollback
2. ✅ **Run tests early** — Fail fast in the pipeline
3. ✅ **Use environment variables** — No hardcoded secrets
4. ✅ **Store secrets securely** — Parameter Store / Secrets Manager
5. ✅ **Blue/Green for production** — Zero-downtime deployments
6. ✅ **Manual approval** for production — Prevent accidental deploys
7. ✅ **Monitor deployments** — CloudWatch alarms during deploy
8. ✅ **Cache dependencies** — Faster builds
9. ✅ **Version artifacts** — Track what's deployed where
10. ✅ **Rollback plan** — Always have a way back

## Common Interview Questions

1. **Q: CI vs CD vs CD?**
   - CI: Auto build & test. CD (Delivery): Auto prepare release. CD (Deployment): Auto deploy to prod.

2. **Q: Blue/Green vs Rolling deployment?**
   - Blue/Green: Create full new environment, switch traffic (zero downtime, instant rollback)
   - Rolling: Update instances one by one (minimal downtime, slower rollback)

3. **Q: What does buildspec.yml do?**
   - Defines build commands for CodeBuild (install, test, build, artifact phases).

4. **Q: How to handle secrets in a pipeline?**
   - SSM Parameter Store or Secrets Manager, referenced in buildspec env section; never in code.

5. **Q: How to rollback a deployment?**
   - CodeDeploy: Auto-rollback on failure, or manually redeploy previous revision. Blue/Green: switch traffic back.

---

[← Previous: Infrastructure as Code](14-Infrastructure-as-Code.md) | [Back to Main Guide](README.md) | [Next: Security & Compliance →](16-Security-Compliance.md)
