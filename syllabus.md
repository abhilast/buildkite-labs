<!-- cSpell:words pytest taskflow pyenv virtualenv PYTHONPATH conftest addopts junitxml htmlcov etree getroot collectstatic noinput gunicorn wsgi buildkit pipefail eksctl serviceaccount configmap kustomization pipenv xdist loadscope makemigrations myorg pylint mypy isort pyproject testpaths cutover envsubst cwagentconfig boto utcnow Chargeback timedelta runbook showmigrations -->

# Buildkite Mastery Syllabus

## Table of Contents

- [Buildkite Mastery Syllabus](#buildkite-mastery-syllabus)
  - [Table of Contents](#table-of-contents)
  - [Project Context](#project-context)
    - [**Module 1: Buildkite Fundamentals \& Architecture**](#module-1-buildkite-fundamentals--architecture)
    - [**Module 2: First Pipeline \& Local Agent**](#module-2-first-pipeline--local-agent)
    - [**Module 3: Cloud Agents on AWS EC2**](#module-3-cloud-agents-on-aws-ec2)
    - [**Module 4: Parallelism \& Test Distribution**](#module-4-parallelism--test-distribution)
    - [**Module 5: Artifacts \& Build Metadata**](#module-5-artifacts--build-metadata)
    - [**Module 6: Docker Plugin \& Containerized Builds**](#module-6-docker-plugin--containerized-builds)
    - [**Module 7: Building \& Publishing Docker Images to ECR**](#module-7-building--publishing-docker-images-to-ecr)
    - [**Module 8: Secrets Management with AWS**](#module-8-secrets-management-with-aws)
    - [**Module 9: EKS-Based Agent Fleet**](#module-9-eks-based-agent-fleet)
    - [**Module 10: Performance Optimization**](#module-10-performance-optimization)
    - [**Module 11: Dynamic Pipelines \& API Automation**](#module-11-dynamic-pipelines--api-automation)
    - [**Module 12: Testing Strategies \& Quality Gates**](#module-12-testing-strategies--quality-gates)
    - [**Module 13: Deployment Patterns to EKS**](#module-13-deployment-patterns-to-eks)
    - [**Module 14: Observability \& Monitoring**](#module-14-observability--monitoring)
    - [**Module 15: Cost Optimization \& Resource Management**](#module-15-cost-optimization--resource-management)
    - [**Module 16: Troubleshooting \& Debugging**](#module-16-troubleshooting--debugging)
    - [**Module 17: Security \& Compliance**](#module-17-security--compliance)
    - [**Module 18: Capstone Project - Production-Ready CI/CD Platform**](#module-18-capstone-project---production-ready-cicd-platform)

## Project Context

Throughout this syllabus, we'll build and evolve a **Django-based API application** called "TaskFlow" - a task management system. Each module will add CI/CD capabilities to this application, progressively building toward a production-ready deployment pipeline.

**Technology Stack:**

- **Application**: Python 3.11+, Django 4.2+, Django REST Framework, PostgreSQL
- **Testing**: pytest, pytest-django, pytest-cov, factory_boy
- **Local Development**: macOS with Docker Desktop
- **Cloud Infrastructure**: AWS (EC2, EKS, ECR, RDS, S3, Secrets Manager, Parameter Store)
- **Container Orchestration**: Amazon EKS for Buildkite agents
- **Version Control**: GitHub
- **Infrastructure as Code**: Terraform (for AWS resources)

---

### **Module 1: Buildkite Fundamentals & Architecture**

**Objective**: Understand Buildkite's hybrid architecture and set up the foundation

- **1.1** The Buildkite Hybrid Model
  - Control Plane (Buildkite SaaS) vs. Data Plane (your agents in AWS)
  - Why this matters for security, compliance, and cost
  - Comparison with GitHub Actions (hosted runners) and Jenkins (self-hosted)
  - Data flow: GitHub webhook → Buildkite → Agent (AWS) → Results
- **1.2** Core Concepts
  - Organizations, Teams, Pipelines hierarchy
  - Builds, Jobs, Steps - the execution model
  - Agents and Queues - the worker model
  - Build lifecycle: queued → assigned → running → passed/failed
- **1.3** Agent Architecture Fundamentals
  - Agent polling mechanism (long-polling over HTTPS)
  - Job claim and execution flow
  - Bootstrap process and working directory
  - Exit codes and build status propagation

**Hands-on**:

- Create Buildkite organization and connect GitHub account
- Fork/create "TaskFlow" Django starter repo on GitHub
- Explore Buildkite UI: pipelines, builds, agents dashboard
- Review GitHub webhook integration settings

**Knowledge Checkpoint**:

- Why does Buildkite use agents instead of hosted runners like GitHub Actions?
- What are the security implications of running agents in your own AWS account?
- Trace the path of a Git push from GitHub to Buildkite to agent execution
- Where does your code execute and where is build metadata stored?

**Example Project State**: Django starter with basic models (Task, User), requirements.txt, basic tests

---

### **Module 2: First Pipeline & Local Agent**

**Objective**: Run your first Buildkite pipeline with a local macOS agent

- **2.1** Installing Your First Agent (macOS)
  - Install Buildkite agent via Homebrew: `brew install buildkite/buildkite/buildkite-agent`
  - Configure agent token and tags
  - Understanding buildkite-agent.cfg location: `$(brew --prefix)/etc/buildkite-agent/buildkite-agent.cfg`
  - Start agent: `buildkite-agent start`
- **2.2** Your First Pipeline - Basic Python Tests
  - Create `.buildkite/pipeline.yml` in TaskFlow repo
  - Pipeline YAML structure: steps array, command steps
  - Step attributes: label, command, key
  - Running pytest: `pytest tests/ -v`
- **2.3** Python Environment Setup in Pipeline
  - Using pyenv/virtualenv in agent bootstrap
  - Installing dependencies: `pip install -r requirements.txt`
  - Setting PYTHONPATH and Django settings
  - Environment variables for local testing
- **2.4** Basic Pipeline Flow
  - Sequential steps: install deps → run tests → report results
  - Exit codes and build success/failure
  - Viewing build logs in Buildkite UI
  - Git commit hooks triggering builds

**Hands-on**:

- Install Buildkite agent on your Mac
- Create `.buildkite/pipeline.yml` with 2 steps:
  1. Setup: install Python dependencies
  2. Test: run pytest
- Trigger build via Git push to GitHub
- Watch logs in real-time from macOS terminal and Buildkite UI

**Knowledge Checkpoint**:

- How does the agent know which jobs to pick up?
- What happens if pytest fails? How does Buildkite know?
- Where do the build commands actually execute?
- How would you run multiple test commands sequentially?

**Example Project State**:

```
taskflow/
├── .buildkite/
│   └── pipeline.yml          # Basic 2-step pipeline
├── taskflow/                  # Django app
├── tests/
│   └── test_tasks.py         # Basic pytest tests
├── requirements.txt
└── pytest.ini
```

---

### **Module 3: Cloud Agents on AWS EC2**

**Objective**: Move from local macOS agent to cloud-hosted Linux agents on AWS

- **3.1** AWS Infrastructure Setup
  - VPC, subnets, security groups for Buildkite agents
  - IAM role for EC2 instances (S3, ECR, Secrets Manager access)
  - Launch template for agent EC2 instances
  - Amazon Linux 2023 AMI selection
- **3.2** Agent Installation on EC2
  - User data script for agent bootstrap
  - Installing Docker, Git, Python on Amazon Linux
  - Buildkite agent installation via RPM
  - Agent configuration: queue tags, instance metadata
- **3.3** Agent Queues & Targeting
  - Creating named queues: `default`, `linux`, `docker`
  - Queue assignment in pipeline: `agents: { queue: "linux" }`
  - Agent tags for routing: `os=linux`, `instance-type=t3.medium`
  - Priority and concurrency limits
- **3.4** Testing the Cloud Agent
  - Updating pipeline to target Linux queue
  - Python/pip setup on Linux (vs macOS differences)
  - Verifying builds run on EC2 instead of Mac
  - Logs and troubleshooting remote execution

**Hands-on**:

- Use Terraform to provision:
  - VPC with public subnet
  - Security group allowing outbound HTTPS (agent→Buildkite)
  - IAM role with S3/ECR permissions
  - EC2 instance with Buildkite agent installed
- Create `linux` queue in Buildkite
- Update pipeline to use `agents: { queue: "linux" }`
- Push code and verify build runs on EC2
- SSH into instance and observe agent logs

**Knowledge Checkpoint**:

- Why does the agent need outbound HTTPS but not inbound access?
- What IAM permissions does your agent need minimally?
- How does queue targeting work when multiple agents are available?
- What happens if no agent matches the queue specification?

**Example Project State**:

```
taskflow/
├── .buildkite/
│   └── pipeline.yml          # Updated with queue targeting
├── terraform/
│   ├── main.tf               # EC2 instance for agent
│   ├── iam.tf                # Agent IAM role
│   └── variables.tf
└── scripts/
    └── agent-userdata.sh     # EC2 bootstrap script
```

---

### **Module 4: Parallelism & Test Distribution**

**Objective**: Run tests in parallel and understand dependency orchestration

- **4.1** Parallel Test Execution
  - Running multiple pytest steps concurrently
  - Using step `key` for identification
  - Organizing tests: unit, integration, e2e
  - pytest markers for test categorization (`@pytest.mark.unit`)
- **4.2** Fan-out Pattern
  - Multiple parallel steps from single setup
  - Shared dependency step (install deps once)
  - Independent test suites running simultaneously
  - Resource isolation per test step
- **4.3** Fan-in Pattern with `depends_on`
  - Multiple test steps → single reporting step
  - Using `depends_on: ["test-unit", "test-integration"]`
  - Wait points for synchronization
  - Aggregate reporting from parallel executions
- **4.4** Conditional Steps
  - Running steps based on branch: `if: build.branch == "main"`
  - Skip steps based on commit message
  - Conditional linting/security scans
  - Environment-specific validation

**Hands-on**:

- Split TaskFlow tests into categories:
  - Unit tests (models, serializers) - fast
  - Integration tests (API endpoints) - medium
  - Database tests (PostgreSQL required) - slower
- Create pipeline with parallel test execution:

  ```yaml
  - label: ":python: Setup"
    key: "setup"
    command: "pip install -r requirements.txt"

  - wait

  - label: ":pytest: Unit Tests"
    key: "test-unit"
    command: "pytest tests/unit -v"
    depends_on: "setup"

  - label: ":pytest: Integration Tests"
    key: "test-integration"
    command: "pytest tests/integration -v"
    depends_on: "setup"

  - wait

  - label: ":bar_chart: Coverage Report"
    command: "pytest --cov=taskflow --cov-report=term"
    depends_on: ["test-unit", "test-integration"]
  ```

- Observe parallel execution in Buildkite timeline
- Measure build time improvement

**Knowledge Checkpoint**:

- How does `wait` differ from `depends_on`?
- What happens if a parallel test step fails? Do others continue?
- How would you ensure all tests pass before deployment step?
- Design a fan-out/fan-in pattern for 10 test suites

**Example Project State**:

```
taskflow/
├── .buildkite/
│   └── pipeline.yml          # Parallel test execution
├── tests/
│   ├── unit/
│   │   ├── test_models.py
│   │   └── test_serializers.py
│   ├── integration/
│   │   ├── test_api_tasks.py
│   │   └── test_api_users.py
│   └── conftest.py           # pytest fixtures
└── pytest.ini                # Markers configuration
```

---

### **Module 5: Artifacts & Build Metadata**

**Objective**: Store and share data between pipeline steps using artifacts and metadata

- **5.1** Artifact System for Test Results
  - Uploading pytest results: `buildkite-agent artifact upload`
  - Coverage reports (HTML, XML, JSON)
  - Test result artifacts (JUnit XML for annotations)
  - Artifact storage backends (Buildkite-hosted vs. S3)
  - Glob patterns: `coverage/**/*`, `test-results/*.xml`
- **5.2** Downloading Artifacts
  - `buildkite-agent artifact download` in dependent steps
  - Download patterns and filtering
  - Artifact availability across steps
  - Using artifacts for deployment packages
- **5.3** Build Metadata
  - Storing test counts: `buildkite-agent meta-data set "test-count" "143"`
  - Coverage percentage metadata
  - Passing data between steps without file storage
  - Metadata scope: build-wide vs. job-specific
  - Reading metadata: `buildkite-agent meta-data get "test-count"`
- **5.4** Annotations for Test Reports
  - JUnit XML → Buildkite annotations
  - Coverage report display in UI
  - Failure summaries and test output
  - Styling annotations: error, warning, info, success
  - Markdown formatting in annotations

**Hands-on**:

- Configure pytest to generate JUnit XML:
  ```ini
  # pytest.ini
  [pytest]
  addopts = --junitxml=test-results/junit.xml --cov=taskflow --cov-report=html --cov-report=xml
  ```
- Update pipeline to upload artifacts:
  ```yaml
  - label: ":pytest: Run Tests"
    command: |
      pytest tests/ -v
      buildkite-agent artifact upload "test-results/junit.xml"
      buildkite-agent artifact upload "htmlcov/**/*"
      buildkite-agent artifact upload "coverage.xml"

  - label: ":bar_chart: Annotate Coverage"
    command: |
      buildkite-agent artifact download "coverage.xml" .
      coverage=$(python -c "import xml.etree.ElementTree as ET; print(ET.parse('coverage.xml').getroot().get('line-rate'))")
      buildkite-agent annotate "Coverage: ${coverage}%" --style info
  ```
- Configure S3 bucket for artifact storage
- View artifacts in Buildkite UI
- Download coverage HTML report

**Knowledge Checkpoint**:

- When would you use artifacts vs. metadata?
- How long are artifacts retained?
- What's the cost implication of storing large artifacts?
- How would you implement a coverage threshold gate?

**Example Project State**:

```
taskflow/
├── .buildkite/
│   └── pipeline.yml          # Artifact upload/download
├── test-results/             # Generated by pytest (gitignored)
│   └── junit.xml
├── htmlcov/                  # Generated coverage report
│   └── index.html
└── scripts/
    └── annotate-coverage.py  # Parse coverage.xml for annotations
```

---

### **Module 6: Docker Plugin & Containerized Builds**

**Objective**: Run builds in Docker containers for consistency and isolation

- **6.1** Why Docker for CI/CD
  - Consistent build environment across agents
  - Dependency isolation (Python versions, system packages)
  - Faster agent startup (no pip install on bare metal)
  - Image versioning for reproducibility
- **6.2** Docker Plugin Basics
  - Plugin configuration in pipeline YAML
  - Using official Python images: `python:3.11-slim`
  - Volume mounts and workspace access
  - Environment variable propagation
  - Container cleanup
- **6.3** Docker Compose Plugin for Integration Tests
  - Running Django with PostgreSQL
  - Service dependencies in docker-compose.yml
  - Waiting for service readiness
  - Network configuration for inter-service communication
  - Database migrations in containerized environment
- **6.4** Common Buildkite Plugins
  - `docker-buildkite-plugin`: Run steps in containers
  - `docker-compose-buildkite-plugin`: Multi-service tests
  - `ecr-buildkite-plugin`: AWS ECR authentication
  - `artifacts-buildkite-plugin`: Advanced artifact handling
  - `junit-annotate-buildkite-plugin`: Test result annotations
  - Plugin versioning and pinning: `plugins#v4.0.0`

**Hands-on**:

- Create Dockerfile for TaskFlow:
  ```dockerfile
  FROM python:3.11-slim
  WORKDIR /app
  COPY requirements.txt .
  RUN pip install --no-cache-dir -r requirements.txt
  COPY . .
  ```
- Update pipeline to use Docker plugin:
  ```yaml
  - label: ":pytest: Tests in Docker"
    plugins:
      - docker#v5.8.0:
          image: "python:3.11-slim"
          command: |
            pip install -r requirements.txt
            pytest tests/ -v
          volumes:
            - "./:/app"
          workdir: "/app"
  ```
- Create docker-compose.yml for integration tests:
  ```yaml
  version: "3.8"
  services:
    db:
      image: postgres:15
      environment:
        POSTGRES_DB: taskflow_test
        POSTGRES_PASSWORD: test
    app:
      build: .
      depends_on:
        - db
      environment:
        DATABASE_URL: postgresql://postgres:test@db:5432/taskflow_test
  ```
- Use docker-compose plugin in pipeline
- Verify tests run with PostgreSQL

**Knowledge Checkpoint**:

- How does the Docker plugin mount your workspace?
- What happens to containers after the build completes?
- When would you use docker vs. docker-compose plugin?
- How would you cache pip dependencies in Docker builds?

**Example Project State**:

```
taskflow/
├── .buildkite/
│   └── pipeline.yml          # Docker and docker-compose plugins
├── Dockerfile
├── docker-compose.yml
├── docker-compose.buildkite.yml  # CI-specific compose file
└── .dockerignore
```

---

### **Module 7: Building & Publishing Docker Images to ECR**

**Objective**: Build application Docker images and push to Amazon ECR

- **7.1** AWS ECR Setup
  - Creating ECR repository: `taskflow-app`
  - Repository policies and lifecycle rules
  - IAM permissions for agent (ecr:GetAuthorizationToken, ecr:PushImage)
  - Image tagging strategy: git SHA, branch, semver
- **7.2** Docker Build in Pipeline
  - Multi-stage builds for optimization
  - Build arguments for versioning
  - Layer caching with BuildKit
  - Build context and .dockerignore
  - Security: running as non-root user
- **7.3** ECR Authentication & Push
  - Using ecr-buildkite-plugin for authentication
  - Alternative: manual `aws ecr get-login-password`
  - Pushing multiple tags
  - Image scanning for vulnerabilities
  - Verifying image in ECR console
- **7.4** Image Caching Strategies
  - Docker BuildKit cache mounts
  - ECR as cache backend
  - Layer reuse across builds
  - Cache invalidation strategies
  - Measuring cache hit rates

**Hands-on**:

- Create multi-stage Dockerfile:

  ```dockerfile
  # Build stage
  FROM python:3.11-slim as builder
  WORKDIR /app
  COPY requirements.txt .
  RUN pip install --user --no-cache-dir -r requirements.txt

  # Runtime stage
  FROM python:3.11-slim
  WORKDIR /app
  COPY --from=builder /root/.local /root/.local
  COPY . .
  ENV PATH=/root/.local/bin:$PATH
  RUN python manage.py collectstatic --noinput
  CMD ["gunicorn", "taskflow.wsgi:application", "--bind", "0.0.0.0:8000"]
  ```

- Create ECR repository via Terraform
- Update pipeline to build and push:
  ```yaml
  - label: ":docker: Build & Push to ECR"
    plugins:
      - ecr#v2.7.0:
          login: true
          account-ids: "123456789012"
          region: us-east-1
      - docker-buildkite-plugin#v5.8.0:
          image: "123456789012.dkr.ecr.us-east-1.amazonaws.com/taskflow-app"
          tags:
            - "${BUILDKITE_COMMIT:0:7}"
            - "${BUILDKITE_BRANCH}"
          push-only: true
          buildkit: true
          cache-from:
            - "123456789012.dkr.ecr.us-east-1.amazonaws.com/taskflow-app:main"
  ```
- Verify image in ECR
- Test image locally: `docker run -p 8000:8000 <image>`

**Knowledge Checkpoint**:

- Why use multi-stage builds?
- How does ECR authentication work with the agent's IAM role?
- What's the benefit of BuildKit over classic Docker build?
- Design an image tagging strategy for dev/staging/prod

**Example Project State**:

```
taskflow/
├── .buildkite/
│   └── pipeline.yml          # Build + push to ECR
├── Dockerfile                # Multi-stage production image
├── .dockerignore
├── terraform/
│   └── ecr.tf                # ECR repository
└── scripts/
    └── tag-image.sh          # Semver tagging helper
```

---

### **Module 8: Secrets Management with AWS**

**Objective**: Securely manage Django secrets using AWS Secrets Manager and Parameter Store

- **8.1** Django Secrets Landscape
  - DATABASE_URL, SECRET_KEY, API keys
  - Environment-specific secrets (dev, staging, prod)
  - Third-party service credentials
  - Why secrets don't belong in environment variables or code
- **8.2** AWS Secrets Manager Integration
  - Storing secrets: database passwords, Django SECRET_KEY
  - IAM permissions for agent to read secrets
  - Retrieving secrets in pipeline using AWS CLI
  - Secret rotation and versioning
  - Cost considerations: Secrets Manager vs. Parameter Store
- **8.3** AWS Systems Manager Parameter Store
  - Storing non-sensitive config (feature flags, URLs)
  - Parameter hierarchies: `/taskflow/dev/`, `/taskflow/prod/`
  - Parameter types: String, StringList, SecureString
  - Retrieving parameters in pipeline
  - Parameter Store advantages: free tier, integrated with AWS
- **8.4** Environment Hooks for Secret Injection
  - Agent environment hook: `~/.buildkite-agent/hooks/environment`
  - Fetching secrets before job execution
  - Exporting as environment variables
  - Secret redaction in build logs
  - Securing hook execution

**Hands-on**:

- Create secrets in AWS Secrets Manager:
  ```bash
  aws secretsmanager create-secret \
    --name taskflow/dev/database-url \
    --secret-string "postgresql://user:pass@host:5432/db"

  aws secretsmanager create-secret \
    --name taskflow/dev/django-secret-key \
    --secret-string "your-secret-key-here"
  ```
- Create parameters in Parameter Store:
  ```bash
  aws ssm put-parameter \
    --name /taskflow/dev/debug \
    --value "True" \
    --type String
  ```
- Create environment hook script:

  ```bash
  #!/bin/bash
  # hooks/environment
  set -euo pipefail

  if [[ "${BUILDKITE_PIPELINE_SLUG}" == "taskflow" ]]; then
    export DATABASE_URL=$(aws secretsmanager get-secret-value \
      --secret-id taskflow/dev/database-url \
      --query SecretString --output text)

    export DJANGO_SECRET_KEY=$(aws secretsmanager get-secret-value \
      --secret-id taskflow/dev/django-secret-key \
      --query SecretString --output text)
  fi
  ```

- Update IAM role for agents
- Test secret injection in pipeline
- Verify secrets are redacted in logs

**Knowledge Checkpoint**:

- When would you use Secrets Manager vs. Parameter Store?
- How do environment hooks execute? What's the lifecycle?
- How would you implement secret rotation for database passwords?
- Design a secret management strategy for multi-environment deployments

**Example Project State**:

```
taskflow/
├── .buildkite/
│   ├── pipeline.yml
│   └── hooks/
│       └── environment           # Secret injection hook
├── terraform/
│   ├── secrets.tf                # Secrets Manager resources
│   └── iam.tf                    # Updated agent permissions
└── scripts/
    ├── rotate-secrets.sh
    └── bootstrap-secrets.sh      # Initial secret setup
```

---

### **Module 9: EKS-Based Agent Fleet**

**Objective**: Deploy Buildkite agents to Amazon EKS with autoscaling

- **9.1** EKS Cluster Setup for Agents
  - Creating EKS cluster via Terraform/eksctl
  - Node groups: on-demand vs. spot instances
  - VPC CNI and networking considerations
  - IAM roles for service accounts (IRSA)
  - Cluster autoscaler configuration
- **9.2** Buildkite Agent Kubernetes Deployment
  - Agent deployment manifest with pod anti-affinity
  - ConfigMap for agent configuration
  - Secret for agent token
  - Resource requests and limits (CPU, memory)
  - Multiple queues: `kubernetes`, `docker`, `heavy-compute`
- **9.3** Autoscaling Agents
  - Horizontal Pod Autoscaler (HPA) based on CPU/memory
  - Queue-depth based scaling with metrics server
  - Cluster Autoscaler for node scaling
  - Spot instance interruption handling
  - Graceful agent shutdown on pod termination
- **9.4** Agent Pod Design Patterns
  - DinD (Docker-in-Docker) for building images
  - Privileged vs. rootless containers
  - Persistent volume claims for workspace caching
  - Network policies for security
  - Pod lifecycle: preStop hooks for cleanup

**Hands-on**:

- Create EKS cluster with Terraform:
  ```hcl
  module "eks" {
    source  = "terraform-aws-modules/eks/aws"
    version = "~> 19.0"

    cluster_name    = "buildkite-agents"
    cluster_version = "1.28"

    vpc_id     = module.vpc.vpc_id
    subnet_ids = module.vpc.private_subnets

    eks_managed_node_groups = {
      agents = {
        instance_types = ["t3.medium", "t3a.medium"]
        capacity_type  = "SPOT"
        min_size       = 1
        max_size       = 10
        desired_size   = 2
      }
    }
  }
  ```
- Create Buildkite agent deployment:
  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: buildkite-agent
  spec:
    replicas: 2
    selector:
      matchLabels:
        app: buildkite-agent
    template:
      metadata:
        labels:
          app: buildkite-agent
      spec:
        serviceAccountName: buildkite-agent
        containers:
          - name: agent
            image: buildkite/agent:3
            env:
              - name: BUILDKITE_AGENT_TOKEN
                valueFrom:
                  secretKeyRef:
                    name: buildkite-agent
                    key: token
              - name: BUILDKITE_AGENT_TAGS
                value: "queue=kubernetes,os=linux,docker=true"
            resources:
              requests:
                cpu: 1000m
                memory: 2Gi
              limits:
                cpu: 2000m
                memory: 4Gi
            volumeMounts:
              - name: docker-sock
                mountPath: /var/run/docker.sock
        volumes:
          - name: docker-sock
            hostPath:
              path: /var/run/docker.sock
  ```
- Configure IRSA for AWS access
- Deploy agent to EKS
- Test builds running on Kubernetes pods
- Configure HPA for autoscaling
- Monitor agent pods with kubectl

**Knowledge Checkpoint**:

- Why run agents on EKS vs. EC2?
- What are the security implications of mounting Docker socket?
- How does IRSA work for AWS credentials?
- Design an autoscaling strategy for 1000+ builds/day

**Example Project State**:

```
taskflow/
├── .buildkite/
│   └── pipeline.yml          # Updated queue: kubernetes
├── terraform/
│   ├── eks.tf                # EKS cluster
│   ├── irsa.tf               # IAM roles for service accounts
│   └── vpc.tf
├── kubernetes/
│   ├── buildkite-agent/
│   │   ├── deployment.yaml
│   │   ├── serviceaccount.yaml
│   │   ├── secret.yaml
│   │   ├── configmap.yaml
│   │   └── hpa.yaml
│   └── kustomization.yaml
└── scripts/
    └── deploy-agents.sh
```

---

### **Module 10: Performance Optimization**

**Objective**: Maximize pipeline speed through caching, parallelization, and optimization

- **10.1** Python Dependency Caching
  - pip cache directory mounting
  - Using Docker cache mounts for /root/.cache/pip
  - Requirements.txt layering strategy
  - poetry/pipenv cache optimization
  - Comparing cached vs. uncached build times
- **10.2** Test Parallelization with pytest-xdist
  - Installing pytest-xdist
  - Running tests across multiple CPUs: `pytest -n auto`
  - Test distribution strategies
  - Database isolation for parallel tests
  - Measuring speedup with parallel execution
- **10.3** Docker Layer Caching
  - BuildKit inline cache: `--cache-from`
  - Registry caching with ECR
  - Optimizing Dockerfile layer order
  - Multi-stage build cache optimization
  - Cache invalidation strategies
- **10.4** Build Matrix for Python Versions
  - Testing against Python 3.10, 3.11, 3.12
  - Matrix build pattern in pipeline
  - Parallel version testing
  - Conditional steps based on Python version
- **10.5** Workspace Optimization
  - Git shallow clones: `--depth=1`
  - Sparse checkout for monorepos
  - .gitattributes for LFS files
  - Incremental builds vs. clean builds
  - Agent workspace persistence

**Hands-on**:

- Implement pip caching in Dockerfile:

  ```dockerfile
  FROM python:3.11-slim
  WORKDIR /app

  # Cache pip packages
  RUN --mount=type=cache,target=/root/.cache/pip \
      pip install --upgrade pip

  COPY requirements.txt .
  RUN --mount=type=cache,target=/root/.cache/pip \
      pip install -r requirements.txt

  COPY . .
  ```

- Add pytest-xdist to requirements
- Update pipeline to use parallel tests:
  ```yaml
  - label: ":pytest: Tests (Parallel)"
    command: |
      pip install -r requirements.txt
      pytest -n auto --dist loadscope -v
    plugins:
      - docker#v5.8.0:
          image: python:3.11-slim
          volumes:
            - pip-cache:/root/.cache/pip
  ```
- Create build matrix for Python versions:
  ```yaml
  - label: ":python: Test Python {{ matrix.version }}"
    command: pytest tests/ -v
    matrix:
      setup:
        version:
          - "3.10"
          - "3.11"
          - "3.12"
    plugins:
      - docker#v5.8.0:
          image: "python:{{ matrix.version }}-slim"
  ```
- Measure and compare build times
- Implement ECR layer caching
- Configure shallow git clones

**Knowledge Checkpoint**:

- What's the ROI of dependency caching for different project sizes?
- How does pytest-xdist distribute tests?
- When would shallow clones be problematic?
- Design a caching strategy for a 200-test Django app

**Example Project State**:

```
taskflow/
├── .buildkite/
│   └── pipeline.yml          # Matrix builds, caching
├── Dockerfile                # BuildKit cache mounts
├── requirements.txt
├── requirements-dev.txt
├── pytest.ini                # pytest-xdist config
└── scripts/
    ├── benchmark-builds.sh
    └── clear-cache.sh
```

---

### **Module 11: Dynamic Pipelines & API Automation**

**Objective**: Generate pipelines programmatically and automate via Buildkite API

- **11.1** Dynamic Pipeline Generation
  - `buildkite-agent pipeline upload` command
  - Generating YAML with Python scripts
  - Conditional step generation based on changed files
  - Git diff analysis for selective testing
  - Multi-stage pipeline uploads
- **11.2** Detecting Changed Files
  - Git diff between HEAD and base branch
  - Detecting changed Python modules
  - Running tests only for affected apps
  - Migration-specific checks
  - Documentation vs. code changes
- **11.3** Python-Based Pipeline Generator
  - Using PyYAML for pipeline generation
  - Jinja2 templates for pipeline YAML
  - Logic for test selection
  - Environment-based pipeline variation
  - DRY principles for pipeline code
- **11.4** Buildkite REST API
  - Authentication with API tokens
  - Triggering builds programmatically
  - Querying build status
  - Managing pipelines via API
  - Webhooks for event-driven automation
- **11.5** Automation Use Cases
  - Slack notifications via Python script
  - Auto-retry flaky tests
  - Deployment approval workflows
  - Collecting metrics for dashboards

**Hands-on**:

- Create Python pipeline generator:

  ```python
  # scripts/generate_pipeline.py
  import yaml
  import subprocess

  def get_changed_files():
      """Get files changed compared to main branch"""
      result = subprocess.run(
          ['git', 'diff', '--name-only', 'origin/main...HEAD'],
          capture_output=True, text=True
      )
      return result.stdout.strip().split('\n')

  def generate_test_steps(changed_files):
      """Generate test steps based on changed files"""
      steps = []

      # Check if models changed
      if any('models.py' in f for f in changed_files):
          steps.append({
              'label': ':django: Migration Check',
              'command': 'python manage.py makemigrations --check --dry-run'
          })

      # Check if API changed
      if any('api/' in f for f in changed_files):
          steps.append({
              'label': ':pytest: API Tests',
              'command': 'pytest tests/api/ -v'
          })

      return steps

  def main():
      changed_files = get_changed_files()
      steps = generate_test_steps(changed_files)

      pipeline = {'steps': steps}
      print(yaml.dump(pipeline))

  if __name__ == '__main__':
      main()
  ```

- Create initial pipeline that uploads dynamic pipeline:
  ```yaml
  # .buildkite/pipeline.yml
  steps:
    - label: ":pipeline: Generate Pipeline"
      command: |
        python scripts/generate_pipeline.py | buildkite-agent pipeline upload
  ```
- Use Buildkite API to trigger builds:

  ```python
  import requests

  API_TOKEN = os.environ['BUILDKITE_API_TOKEN']
  ORG_SLUG = 'myorg'
  PIPELINE_SLUG = 'taskflow'

  response = requests.post(
      f'https://api.buildkite.com/v2/organizations/{ORG_SLUG}/pipelines/{PIPELINE_SLUG}/builds',
      headers={'Authorization': f'Bearer {API_TOKEN}'},
      json={
          'commit': 'HEAD',
          'branch': 'main',
          'message': 'Triggered via API'
      }
  )
  ```

- Implement Slack notifications
- Create dashboard using GraphQL API

**Knowledge Checkpoint**:

- When should you use dynamic vs. static pipelines?
- How would you test a pipeline generator?
- Design a selective testing strategy for a Django monolith
- What are the security implications of dynamic pipelines?

**Example Project State**:

```
taskflow/
├── .buildkite/
│   ├── pipeline.yml          # Initial upload step
│   └── templates/
│       └── test-template.yml # Jinja2 template
├── scripts/
│   ├── generate_pipeline.py
│   ├── trigger_build.py
│   ├── notify_slack.py
│   └── check_migrations.py
└── tests/
    └── test_pipeline_generator.py
```

---

### **Module 12: Testing Strategies & Quality Gates**

**Objective**: Implement comprehensive Django testing with quality enforcement

- **12.1** Django Test Organization
  - Unit tests: models, serializers, utilities
  - Integration tests: API endpoints, views
  - E2E tests: user workflows with Selenium/Playwright
  - Test fixtures with factory_boy
  - Database setup: pytest-django and --reuse-db
- **12.2** Code Quality Checks
  - Linting: ruff, flake8, pylint
  - Type checking: mypy for Django models and views
  - Code formatting: black, isort
  - Security: bandit for security issues
  - Import sorting and organization
- **12.3** Coverage Enforcement
  - pytest-cov for coverage reporting
  - Coverage thresholds: `--cov-fail-under=80`
  - Branch coverage vs. line coverage
  - Excluding migrations and settings from coverage
  - Coverage reports as artifacts
- **12.4** Quality Gates in Pipeline
  - Mandatory checks that must pass
  - Coverage threshold gates
  - Security scan failures block merge
  - Migration safety checks
  - Django system checks: `python manage.py check`
- **12.5** Test Result Reporting
  - JUnit XML for test annotations
  - HTML coverage reports
  - Flaky test detection and retry
  - Test duration tracking
  - Historical test analytics

**Hands-on**:

- Organize tests by type:
  ```
  tests/
  ├── unit/
  │   ├── test_models.py
  │   ├── test_serializers.py
  │   └── test_forms.py
  ├── integration/
  │   ├── test_api_tasks.py
  │   └── test_api_auth.py
  ├── e2e/
  │   └── test_task_workflow.py
  ├── conftest.py
  └── factories.py
  ```
- Create comprehensive quality gate pipeline:
  ```yaml
  steps:
    - label: ":snake: Lint & Format"
      command: |
        ruff check .
        black --check .
        isort --check-only .
        mypy taskflow/

    - label: ":lock: Security Scan"
      command: bandit -r taskflow/ -f json -o bandit-report.json

    - label: ":django: Django Checks"
      command: |
        python manage.py check
        python manage.py makemigrations --check --dry-run

    - wait

    - label: ":pytest: Unit Tests"
      command: |
        pytest tests/unit/ -v \
          --cov=taskflow \
          --cov-report=html \
          --cov-report=xml \
          --junitxml=test-results/unit.xml
      artifact_paths:
        - "test-results/*.xml"
        - "htmlcov/**/*"

    - label: ":pytest: Integration Tests"
      command: |
        pytest tests/integration/ -v \
          --junitxml=test-results/integration.xml
      plugins:
        - docker-compose#v4.14.0:
            run: app
            config: docker-compose.buildkite.yml

    - wait

    - label: ":bar_chart: Coverage Gate"
      command: |
        pytest tests/ --cov=taskflow --cov-fail-under=80
  ```
- Configure tool settings:

  ```ini
  # pyproject.toml
  [tool.pytest.ini_options]
  DJANGO_SETTINGS_MODULE = "taskflow.settings.test"
  python_files = ["test_*.py"]
  testpaths = ["tests"]

  [tool.coverage.run]
  source = ["taskflow"]
  omit = ["*/migrations/*", "*/tests/*", "*/settings/*"]

  [tool.black]
  line-length = 88
  target-version = ['py311']

  [tool.ruff]
  select = ["E", "F", "I", "N", "W"]
  line-length = 88
  ```

- Implement flaky test retry
- Create coverage trend dashboard

**Knowledge Checkpoint**:

- Why separate unit, integration, and e2e tests?
- What coverage threshold is appropriate for production Django apps?
- How would you handle flaky integration tests?
- Design a quality gate strategy for a regulated environment

**Example Project State**:

```
taskflow/
├── .buildkite/
│   └── pipeline.yml          # Quality gates
├── tests/
│   ├── unit/
│   ├── integration/
│   ├── e2e/
│   ├── conftest.py
│   └── factories.py
├── pyproject.toml            # Tool configuration
├── .ruff.toml
└── scripts/
    ├── check-coverage.py
    └── retry-flaky-tests.sh
```

---

### **Module 13: Deployment Patterns to EKS**

**Objective**: Deploy Django application to Amazon EKS with progressive delivery

- **13.1** EKS Deployment Fundamentals
  - Kubernetes manifests for Django
  - Deployment, Service, Ingress resources
  - ConfigMap for Django settings
  - Secret for sensitive configuration
  - Database connection: RDS PostgreSQL
  - Static files: S3 + CloudFront
- **13.2** Blue-Green Deployment
  - Two identical environments (blue=current, green=new)
  - Service selector switching
  - Zero-downtime cutover
  - Quick rollback capability
  - Testing green before cutover
- **13.3** Canary Deployment with Flagger
  - Installing Flagger on EKS
  - Canary deployment spec
  - Progressive traffic shifting: 10% → 50% → 100%
  - Automated rollback on metrics
  - Integration with Prometheus for metrics
- **13.4** Rolling Updates
  - Kubernetes rolling update strategy
  - maxSurge and maxUnavailable settings
  - Health checks: readiness and liveness probes
  - Pre-stop hooks for graceful shutdown
  - Database migrations in rolling updates
- **13.5** Environment Progression Pipeline
  - Dev → Staging → Production
  - Manual approval gates with block steps
  - Environment-specific configuration
  - Smoke tests after deployment
  - Rollback procedures

**Hands-on**:

- Create Kubernetes manifests:
  ```yaml
  # kubernetes/taskflow/deployment.yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: taskflow
  spec:
    replicas: 3
    strategy:
      type: RollingUpdate
      rollingUpdate:
        maxSurge: 1
        maxUnavailable: 0
    selector:
      matchLabels:
        app: taskflow
    template:
      metadata:
        labels:
          app: taskflow
      spec:
        serviceAccountName: taskflow
        containers:
          - name: app
            image: ${ECR_IMAGE}
            ports:
              - containerPort: 8000
            env:
              - name: DATABASE_URL
                valueFrom:
                  secretKeyRef:
                    name: taskflow-secrets
                    key: database-url
            livenessProbe:
              httpGet:
                path: /health/
                port: 8000
              initialDelaySeconds: 30
            readinessProbe:
              httpGet:
                path: /ready/
                port: 8000
              initialDelaySeconds: 10
  ```
- Create deployment pipeline:
  ```yaml
  steps:
    # ... build and test steps ...

    - wait

    - label: ":kubernetes: Deploy to Dev"
      command: |
        export ECR_IMAGE="${ECR_REGISTRY}:${BUILDKITE_COMMIT:0:7}"
        envsubst < kubernetes/taskflow/deployment.yaml | kubectl apply -f -
        kubectl rollout status deployment/taskflow -n dev
      agents:
        queue: kubernetes

    - block: ":rocket: Deploy to Staging?"

    - label: ":kubernetes: Deploy to Staging"
      command: |
        export ECR_IMAGE="${ECR_REGISTRY}:${BUILDKITE_COMMIT:0:7}"
        envsubst < kubernetes/taskflow/deployment.yaml | kubectl apply -f - -n staging
        kubectl rollout status deployment/taskflow -n staging
        # Run smoke tests
        python scripts/smoke_tests.py --env staging

    - block: ":rocket: Deploy to Production?"
      fields:
        - select: "Deployment Strategy"
          key: strategy
          options:
            - label: "Rolling Update"
              value: "rolling"
            - label: "Canary"
              value: "canary"

    - label: ":kubernetes: Deploy to Production"
      command: |
        if [ "${STRATEGY}" == "canary" ]; then
          kubectl apply -f kubernetes/taskflow/canary.yaml -n prod
        else
          export ECR_IMAGE="${ECR_REGISTRY}:${BUILDKITE_COMMIT:0:7}"
          envsubst < kubernetes/taskflow/deployment.yaml | kubectl apply -f - -n prod
          kubectl rollout status deployment/taskflow -n prod
        fi
  ```
- Set up RDS PostgreSQL for production
- Configure Django health check endpoints
- Implement Flagger for canary deployments
- Create rollback pipeline

**Knowledge Checkpoint**:

- When would you use canary vs. blue-green deployment?
- How do you handle database migrations in zero-downtime deployments?
- What metrics would trigger an automatic canary rollback?
- Design a deployment strategy for a 99.99% SLA requirement

**Example Project State**:

```
taskflow/
├── .buildkite/
│   ├── pipeline.yml          # Multi-environment deployment
│   └── rollback-pipeline.yml
├── kubernetes/
│   └── taskflow/
│       ├── deployment.yaml
│       ├── service.yaml
│       ├── ingress.yaml
│       ├── configmap.yaml
│       ├── secret.yaml
│       ├── canary.yaml       # Flagger canary resource
│       └── hpa.yaml
├── taskflow/
│   └── views/
│       └── health.py         # Health check endpoints
├── terraform/
│   ├── rds.tf                # Production database
│   └── s3-static.tf
└── scripts/
    ├── smoke_tests.py
    ├── rollback.sh
    └── database-migrate.sh
```

---

### **Module 14: Observability & Monitoring**

**Objective**: Implement comprehensive monitoring for CI/CD pipeline and application health

- **14.1** Pipeline Metrics & Analytics
  - Build duration tracking
  - Test execution time trends
  - Failure rate analysis
  - Queue wait time monitoring
  - Cost per build calculation
- **14.2** Buildkite Analytics Integration
  - Test Analytics for pytest results
  - Flaky test identification
  - Test suite performance over time
  - Slowest tests reporting
  - Historical trend analysis
- **14.3** CloudWatch for Agent Monitoring
  - Agent CPU/memory utilization
  - EKS pod metrics
  - Custom metrics from agents
  - Log aggregation with CloudWatch Logs
  - Alarms for agent availability
- **14.4** Application Observability in Pipeline
  - Django logging configuration
  - Structured logging with JSON formatter
  - APM integration (AWS X-Ray, DataDog)
  - Error tracking (Sentry) in CI
  - Performance profiling in tests
- **14.5** Alerting & Dashboards
  - Slack/PagerDuty integration
  - Build failure notifications
  - Deployment success/failure alerts
  - CloudWatch dashboards for agents
  - Grafana for custom metrics

**Hands-on**:

- Configure Buildkite Test Analytics:
  ```yaml
  # .buildkite/pipeline.yml
  steps:
    - label: ":pytest: Tests with Analytics"
      command: |
        pip install buildkite-test-collector
        pytest tests/ -v \
          --junitxml=test-results/junit.xml
      env:
        BUILDKITE_ANALYTICS_TOKEN: ${ANALYTICS_TOKEN}
      artifact_paths:
        - "test-results/*.xml"
  ```
- Set up CloudWatch agent on EKS nodes:
  ```yaml
  # kubernetes/cloudwatch-agent.yaml
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: cwagentconfig
  data:
    cwagentconfig.json: |
      {
        "metrics": {
          "namespace": "Buildkite/Agents",
          "metrics_collected": {
            "cpu": {
              "measurement": [
                "cpu_usage_idle",
                "cpu_usage_iowait"
              ]
            },
            "mem": {
              "measurement": [
                "mem_used_percent"
              ]
            }
          }
        }
      }
  ```
- Create custom metrics script:

  ```python
  # scripts/track_build_metrics.py
  import boto3
  import os
  from datetime import datetime

  cloudwatch = boto3.client('cloudwatch')

  def track_build_duration(duration_seconds):
      cloudwatch.put_metric_data(
          Namespace='Buildkite/Builds',
          MetricData=[{
              'MetricName': 'BuildDuration',
              'Value': duration_seconds,
              'Unit': 'Seconds',
              'Timestamp': datetime.utcnow(),
              'Dimensions': [
                  {'Name': 'Pipeline', 'Value': os.environ['BUILDKITE_PIPELINE_SLUG']},
                  {'Name': 'Branch', 'Value': os.environ['BUILDKITE_BRANCH']}
              ]
          }]
      )
  ```

- Configure Slack notifications:

  ```bash
  # hooks/post-command
  #!/bin/bash

  if [[ $BUILDKITE_COMMAND_EXIT_STATUS -ne 0 ]]; then
    curl -X POST https://hooks.slack.com/services/YOUR/WEBHOOK/URL \
      -H 'Content-Type: application/json' \
      -d "{
        \"text\": \"Build Failed: ${BUILDKITE_PIPELINE_NAME} - ${BUILDKITE_MESSAGE}\",
        \"blocks\": [{
          \"type\": \"section\",
          \"text\": {
            \"type\": \"mrkdwn\",
            \"text\": \"*Build Failed*\n*Pipeline:* ${BUILDKITE_PIPELINE_NAME}\n*Branch:* ${BUILDKITE_BRANCH}\n*Commit:* ${BUILDKITE_COMMIT:0:7}\"
          }
        }]
      }"
  fi
  ```

- Create Grafana dashboard for build metrics
- Set up PagerDuty for production deployment failures
- Configure Sentry in Django for error tracking

**Knowledge Checkpoint**:

- What metrics indicate an unhealthy CI/CD system?
- How would you identify the root cause of increased build times?
- Design an alerting strategy for a 24/7 deployment pipeline
- What's the ROI of flaky test tracking?

**Example Project State**:

```
taskflow/
├── .buildkite/
│   ├── pipeline.yml          # Analytics integration
│   └── hooks/
│       ├── post-command      # Slack notifications
│       └── pre-exit          # Metrics collection
├── kubernetes/
│   ├── cloudwatch-agent.yaml
│   └── prometheus.yaml
├── terraform/
│   ├── cloudwatch.tf         # Dashboards and alarms
│   └── sns.tf                # Alert topics
├── scripts/
│   ├── track_build_metrics.py
│   └── analyze_flaky_tests.py
└── dashboards/
    ├── grafana-buildkite.json
    └── cloudwatch-agents.json
```

---

### **Module 15: Cost Optimization & Resource Management**

**Objective**: Minimize CI/CD infrastructure costs while maintaining performance

- **15.1** Agent Cost Analysis
  - EC2 vs. EKS cost comparison
  - On-demand vs. spot instance pricing
  - Instance type selection (compute vs. memory optimized)
  - Utilization metrics and rightsizing
  - Idle agent costs
- **15.2** Spot Instances for Agents
  - EKS managed node groups with spot
  - Spot interruption handling
  - Spot instance pools and diversification
  - Graceful agent shutdown on termination
  - Fallback to on-demand for critical builds
- **15.3** Build Optimization for Cost
  - Reducing build duration = lower costs
  - Parallel vs. sequential trade-offs
  - Cache hit rates and storage costs
  - Artifact storage costs (S3, Buildkite-hosted)
  - Test selection vs. full suite runs
- **15.4** Autoscaling Best Practices
  - Queue-based scaling vs. time-based
  - Scale-down delay configuration
  - Minimum pool size optimization
  - Cost of over-provisioning vs. queue wait time
  - Scheduled scaling for predictable load
- **15.5** Cost Allocation & Chargeback
  - Tagging agents by team/project
  - CloudWatch metrics for cost attribution
  - Build duration by pipeline
  - S3 storage costs per pipeline
  - Creating cost reports for stakeholders

**Hands-on**:

- Configure EKS node group with spot instances:
  ```hcl
  # terraform/eks-agents.tf
  eks_managed_node_groups = {
    spot_agents = {
      instance_types = ["t3.medium", "t3a.medium", "t2.medium"]
      capacity_type  = "SPOT"
      min_size       = 1
      max_size       = 20
      desired_size   = 2

      labels = {
        workload = "buildkite-agent"
      }

      taints = [{
        key    = "buildkite"
        value  = "spot"
        effect = "NoSchedule"
      }]

      tags = {
        CostCenter = "Engineering"
        Pipeline   = "CI-CD"
      }
    }

    on_demand_critical = {
      instance_types = ["t3.medium"]
      capacity_type  = "ON_DEMAND"
      min_size       = 1
      max_size       = 3
      desired_size   = 1

      labels = {
        workload = "buildkite-agent-critical"
      }
    }
  }
  ```
- Implement cost tracking:

  ```python
  # scripts/calculate_build_cost.py
  import boto3
  from datetime import datetime, timedelta

  def calculate_agent_cost(hours, instance_type, spot=True):
      # Spot pricing ~70% discount
      pricing = {
          't3.medium': 0.0416 if not spot else 0.0125,
          't3.large': 0.0832 if not spot else 0.0250
      }
      return hours * pricing.get(instance_type, 0)

  def get_build_duration(build_id):
      # Fetch from Buildkite API
      pass

  def main():
      # Calculate monthly cost
      builds = get_builds_this_month()
      total_cost = sum(
          calculate_agent_cost(
              build['duration'] / 3600,
              build['instance_type'],
              build['spot']
          )
          for build in builds
      )
      print(f"Total CI/CD cost this month: ${total_cost:.2f}")
  ```

- Set up AWS Cost Explorer tags
- Create cost optimization dashboard
- Implement build time budget alerts
- Configure aggressive cache strategies
- Analyze and optimize longest-running tests

**Knowledge Checkpoint**:

- What's the break-even point for spot vs. on-demand?
- How do you balance cost vs. developer wait time?
- Design a cost-optimal agent fleet for 500 builds/day
- What are hidden costs in CI/CD infrastructure?

**Example Project State**:

```
taskflow/
├── .buildkite/
│   └── pipeline.yml          # Cost-optimized queues
├── terraform/
│   ├── eks-spot.tf           # Spot instance configuration
│   └── cost-tags.tf
├── scripts/
│   ├── calculate_build_cost.py
│   ├── analyze_cache_hits.py
│   └── optimize_instance_types.py
└── docs/
    └── cost-optimization.md
```

---

### **Module 16: Troubleshooting & Debugging**

**Objective**: Master debugging techniques for common Buildkite and deployment issues

- **16.1** Common Agent Issues
  - Agent not picking up jobs: queue mismatch, token issues
  - Agent connectivity problems: firewall, security groups
  - Agent out of memory/disk space
  - Agent version mismatch
  - EKS pod eviction and scheduling failures
- **16.2** Pipeline Debugging
  - YAML syntax errors and validation
  - Plugin configuration issues
  - Environment variable interpolation bugs
  - Docker build failures
  - pytest collection errors
- **16.3** AWS-Specific Issues
  - ECR authentication failures
  - IAM permission errors (least privilege debugging)
  - S3 artifact upload failures
  - Secrets Manager access denied
  - EKS cluster connectivity issues
- **16.4** Django Test Failures
  - Database connection errors in tests
  - Migration conflicts and test database setup
  - Static files not found
  - Settings module import errors
  - Fixture loading failures
- **16.5** Debugging Techniques
  - SSH into agent for live debugging
  - kubectl exec into agent pods
  - Build logs analysis and searching
  - Buildkite debug mode: `BUILDKITE_AGENT_DEBUG=true`
  - Reproducing issues locally
  - Timeline analysis for performance issues

**Hands-on**:

- Create debugging runbook:

  ```markdown
  # Buildkite Troubleshooting Guide

  ## Agent Not Running Jobs

  1. Check agent is running: `buildkite-agent status`
  2. Verify token: `cat ~/.buildkite-agent/buildkite-agent.cfg`
  3. Check queue: `buildkite-agent meta-data get "queue"`
  4. Review security groups for outbound HTTPS

  ## ECR Push Failing

  1. Check IAM role: `aws sts get-caller-identity`
  2. Test ECR login: `aws ecr get-login-password`
  3. Verify repository exists: `aws ecr describe-repositories`

  ## Tests Failing in CI but Pass Locally

  1. Check Python version match
  2. Verify DATABASE_URL is set
  3. Check for timezone differences
  4. Review test isolation issues
  ```

- Set up debugging pipeline:
  ```yaml
  # .buildkite/debug-pipeline.yml
  steps:
    - label: ":bug: Debug Environment"
      command: |
        echo "=== System Info ==="
        uname -a
        python --version
        pip --version

        echo "=== Environment Variables ==="
        env | grep -E "(BUILDKITE|DJANGO|DATABASE)" || true

        echo "=== Disk Space ==="
        df -h

        echo "=== Memory ==="
        free -h

        echo "=== Network ==="
        curl -I https://api.buildkite.com

        echo "=== AWS Credentials ==="
        aws sts get-caller-identity || echo "No AWS credentials"

        echo "=== Docker ==="
        docker info || echo "Docker not available"
  ```
- Common issue scripts:

  ```python
  # scripts/diagnose_test_failure.py
  import subprocess
  import sys

  def check_database_connection():
      """Verify database connectivity"""
      try:
          from django.db import connection
          connection.ensure_connection()
          print("✓ Database connection successful")
          return True
      except Exception as e:
          print(f"✗ Database connection failed: {e}")
          return False

  def check_migrations():
      """Check for pending migrations"""
      result = subprocess.run(
          ['python', 'manage.py', 'showmigrations', '--plan'],
          capture_output=True,
          text=True
      )
      if '[ ]' in result.stdout:
          print("✗ Pending migrations detected")
          print(result.stdout)
          return False
      print("✓ All migrations applied")
      return True

  if __name__ == '__main__':
      checks = [
          check_database_connection,
          check_migrations
      ]

      if not all(check() for check in checks):
          sys.exit(1)
  ```

- Implement log aggregation with CloudWatch Logs Insights queries
- Create alert rules for common failures
- Build failure pattern detection script

**Knowledge Checkpoint**:

- How would you debug an agent that claims jobs but never reports results?
- What logs do you check for a "permission denied" error pushing to ECR?
- How do you troubleshoot a test that passes locally but fails in CI?
- Design a debugging workflow for production deployment failures

**Example Project State**:

```
taskflow/
├── .buildkite/
│   ├── pipeline.yml
│   └── debug-pipeline.yml
├── docs/
│   ├── troubleshooting.md
│   └── runbook.md
├── scripts/
│   ├── diagnose_test_failure.py
│   ├── check_agent_health.sh
│   ├── debug_ecr_auth.sh
│   └── reproduce_ci_env.sh
└── cloudwatch/
    └── log-insights-queries.txt
```

---

### **Module 17: Security & Compliance**

**Objective**: Implement enterprise-grade security for CI/CD pipeline

- **17.1** Agent Security Hardening
  - EKS pod security standards (restricted)
  - Least privilege IAM roles (IRSA)
  - Network policies for agent isolation
  - Secrets scanning in code (truffleHog, git-secrets)
  - Container image scanning (Trivy, Clair)
- **17.2** Pipeline Security
  - Code injection prevention in dynamic pipelines
  - Input validation for block steps
  - Signed commits verification
  - Branch protection integration
  - Dependency vulnerability scanning (Safety, pip-audit)
- **17.3** Secret Management Security
  - No secrets in code or environment variables
  - AWS Secrets Manager with rotation
  - KMS encryption for artifacts
  - Audit logging for secret access
  - Secret redaction in build logs
- **17.4** Supply Chain Security
  - Signed Docker images with Cosign
  - SBOM generation (Syft)
  - Dependency pinning and verification
  - Private PyPI mirror for vetted packages
  - Image provenance attestation
- **17.5** Compliance & Audit
  - Build artifact retention policies
  - Audit trail for deployments
  - SOC 2 compliance considerations
  - Access reviews and role management
  - Immutable build artifacts

**Hands-on**:

- Implement pod security policy:
  ```yaml
  # kubernetes/buildkite-agent/psp.yaml
  apiVersion: policy/v1beta1
  kind: PodSecurityPolicy
  metadata:
    name: buildkite-agent-restricted
  spec:
    privileged: false
    allowPrivilegeEscalation: false
    requiredDropCapabilities:
      - ALL
    runAsUser:
      rule: "MustRunAsNonRoot"
    seLinux:
      rule: "RunAsAny"
    fsGroup:
      rule: "RunAsAny"
    volumes:
      - "configMap"
      - "secret"
      - "emptyDir"
      - "persistentVolumeClaim"
  ```
- Set up secrets scanning in pipeline:

  ```yaml
  - label: ":lock: Secrets Scan"
    command: |
      pip install truffleHog3
      truffleHog3 --format json . > secrets-scan.json
      if [ -s secrets-scan.json ]; then
        cat secrets-scan.json
        exit 1
      fi

  - label: ":shield: Dependency Scan"
    command: |
      pip install safety
      safety check --json

  - label: ":docker: Container Scan"
    command: |
      docker pull ${ECR_IMAGE}
      trivy image ${ECR_IMAGE} --severity HIGH,CRITICAL --exit-code 1
  ```

- Implement image signing:

  ```bash
  # scripts/sign-image.sh
  #!/bin/bash
  set -euo pipefail

  IMAGE=$1

  # Sign with Cosign
  cosign sign --key awskms:///alias/buildkite-signing-key ${IMAGE}

  # Generate SBOM
  syft ${IMAGE} -o spdx-json > sbom.json

  # Attach SBOM to image
  cosign attach sbom --sbom sbom.json ${IMAGE}
  ```

- Configure IAM role with minimal permissions:
  ```hcl
  # terraform/iam-agent.tf
  data "aws_iam_policy_document" "agent_policy" {
    statement {
      actions = [
        "ecr:GetAuthorizationToken",
        "ecr:BatchCheckLayerAvailability",
        "ecr:GetDownloadUrlForLayer",
        "ecr:BatchGetImage",
        "ecr:PutImage",
        "ecr:InitiateLayerUpload",
        "ecr:UploadLayerPart",
        "ecr:CompleteLayerUpload"
      ]
      resources = [aws_ecr_repository.taskflow.arn]
    }

    statement {
      actions = [
        "secretsmanager:GetSecretValue"
      ]
      resources = [
        "arn:aws:secretsmanager:*:*:secret:taskflow/*"
      ]
    }
  }
  ```
- Implement audit logging
- Set up SAST scanning (Bandit)
- Create compliance report generation

**Knowledge Checkpoint**:

- How do you prevent secret leakage in build logs?
- Design a secure deployment workflow for PCI compliance
- What are the risks of mounting Docker socket in agent pods?
- How would you handle a compromised signing key?

**Example Project State**:

```
taskflow/
├── .buildkite/
│   ├── pipeline.yml          # Security scans integrated
│   └── security-pipeline.yml
├── kubernetes/
│   └── buildkite-agent/
│       ├── psp.yaml
│       └── network-policy.yaml
├── terraform/
│   ├── iam-minimal.tf        # Least privilege roles
│   └── kms.tf                # Encryption keys
├── scripts/
│   ├── sign-image.sh
│   ├── verify-signature.sh
│   ├── generate-sbom.sh
│   └── audit-access.py
└── security/
    ├── .trivyignore
    ├── bandit.yaml
    └── compliance-checklist.md
```

---

### **Module 18: Capstone Project - Production-Ready CI/CD Platform**

**Objective**: Build a complete, production-ready CI/CD platform for Django on AWS/EKS

**Project Goal**: Design and implement a comprehensive CI/CD platform for the TaskFlow application that demonstrates mastery of all previous modules. This platform should be production-grade, secure, observable, and cost-optimized.

**18.1** Platform Requirements

Your platform must include:

**Core Infrastructure:**

- EKS cluster with autoscaling agent fleet (spot + on-demand)
- Multi-environment setup: dev, staging, production
- RDS PostgreSQL for each environment
- ECR for container registry
- S3 for artifacts and static files
- Route53 + ALB for ingress

**CI Pipeline:**

- Automated testing (unit, integration, e2e)
- Code quality gates (linting, type checking, security)
- Parallel test execution with pytest-xdist
- Coverage enforcement (>80%)
- Dynamic pipeline generation based on changed files
- Docker image building with multi-stage optimization
- Image scanning and signing
- Artifact management

**CD Pipeline:**

- Environment progression: dev → staging → production
- Manual approval gates for production
- Canary deployment capability
- Automated rollback on failure
- Database migration handling
- Blue-green deployment option
- Smoke tests after deployment

**Security:**

- Secrets management with AWS Secrets Manager
- IAM roles with least privilege
- Network policies for agent isolation
- Container image scanning
- Dependency vulnerability scanning
- Audit logging

**Observability:**

- CloudWatch metrics and dashboards
- Buildkite Test Analytics integration
- Build duration and cost tracking
- Slack notifications
- Flaky test detection
- Performance profiling

**Cost Optimization:**

- Spot instances for non-critical workloads
- Aggressive caching strategy
- Cost allocation by team/pipeline
- Build time optimization

**18.2** Deliverables

1. **Infrastructure as Code**

   - Complete Terraform configuration
   - Kubernetes manifests
   - Network architecture diagram

2. **Pipeline Implementation**

   - `.buildkite/pipeline.yml` with all stages
   - Dynamic pipeline generator scripts
   - Deployment pipelines for each environment

3. **Documentation**

   - Architecture decision records (ADRs)
   - Runbooks for common operations
   - Troubleshooting guide
   - Cost analysis report
   - Security audit checklist

4. **Operational Readiness**
   - Disaster recovery procedures
   - Rollback playbooks
   - Monitoring and alerting setup
   - On-call runbook

**18.3** Hands-on Implementation

```
final-project/
├── README.md                      # Project overview
├── ARCHITECTURE.md                # Architecture decisions
├── .buildkite/
│   ├── pipeline.yml              # Main pipeline
│   ├── deploy-dev.yml
│   ├── deploy-staging.yml
│   ├── deploy-production.yml
│   ├── rollback.yml
│   └── hooks/
│       ├── environment
│       ├── post-command
│       └── pre-exit
├── terraform/
│   ├── main.tf
│   ├── vpc.tf
│   ├── eks.tf                    # EKS cluster with node groups
│   ├── rds.tf                    # Multi-AZ PostgreSQL
│   ├── ecr.tf
│   ├── s3.tf
│   ├── iam.tf                    # Roles and policies
│   ├── secrets.tf                # Secrets Manager
│   ├── cloudwatch.tf             # Monitoring
│   ├── route53.tf
│   └── variables.tf
├── kubernetes/
│   ├── namespaces/
│   │   ├── dev.yaml
│   │   ├── staging.yaml
│   │   └── production.yaml
│   ├── buildkite-agent/
│   │   ├── deployment.yaml
│   │   ├── hpa.yaml
│   │   ├── pdb.yaml
│   │   ├── network-policy.yaml
│   │   └── service-account.yaml
│   ├── taskflow/
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   ├── ingress.yaml
│   │   ├── configmap.yaml
│   │   ├── secret.yaml
│   │   ├── hpa.yaml
│   │   └── canary.yaml
│   └── monitoring/
│       ├── prometheus.yaml
│       ├── grafana.yaml
│       └── flagger.yaml
├── taskflow/                      # Django application
│   ├── manage.py
│   ├── taskflow/
│   ├── apps/
│   ├── tests/
│   ├── requirements.txt
│   └── pytest.ini
├── scripts/
│   ├── generate_pipeline.py      # Dynamic pipeline generator
│   ├── deploy.sh
│   ├── rollback.sh
│   ├── smoke_tests.py
│   ├── database_migrate.sh
│   ├── sign_image.sh
│   ├── cost_report.py
│   └── verify_deployment.py
├── docs/
│   ├── runbooks/
│   │   ├── deployment.md
│   │   ├── rollback.md
│   │   ├── database-recovery.md
│   │   └── agent-troubleshooting.md
│   ├── adr/                      # Architecture Decision Records
│   │   ├── 001-use-eks-for-agents.md
│   │   ├── 002-canary-vs-blue-green.md
│   │   └── 003-spot-instances.md
│   └── security/
│       ├── threat-model.md
│       └── compliance-checklist.md
└── monitoring/
    ├── dashboards/
    │   ├── buildkite-metrics.json
    │   ├── agent-health.json
    │   └── cost-tracking.json
    └── alerts/
        ├── build-failures.yaml
        └── deployment-failures.yaml
```

**18.4** Evaluation Criteria

Your platform will be evaluated on:

1. **Functionality** (30%)

   - All pipelines execute successfully
   - Deployments work to all environments
   - Rollback procedures function correctly
   - Tests pass consistently

2. **Architecture** (25%)

   - Well-designed infrastructure
   - Proper separation of concerns
   - Scalability considerations
   - Security best practices

3. **Operational Excellence** (25%)

   - Comprehensive monitoring
   - Clear documentation
   - Runbooks for operations
   - Cost optimization implemented

4. **Code Quality** (20%)
   - Clean, maintainable code
   - Proper error handling
   - Infrastructure as Code best practices
   - Security compliance

**18.5** Advanced Challenges (Optional)

If you want to push further:

1. **Multi-Region Deployment**

   - Deploy to multiple AWS regions
   - Cross-region artifact replication
   - Global failover capability

2. **Advanced GitOps**

   - Integration with ArgoCD or Flux
   - Automated sync from Git
   - Drift detection

3. **Chaos Engineering**

   - Implement fault injection tests
   - Test agent failure scenarios
   - Validate rollback procedures under load

4. **ML Pipeline Integration**
   - Add model training pipeline
   - Model versioning and registry
   - A/B testing for model deployments

**18.6** Knowledge Checkpoint

Final Defense Questions:

- Walk through the entire deployment flow from commit to production
- What happens when a canary deployment fails?
- How do you handle a database migration that needs to be rolled back?
- Explain your cost optimization strategy and estimated monthly cost
- How would you scale this platform to support 100 teams?
- What's your disaster recovery RPO and RTO?
- How do you ensure zero-downtime deployments?
- Defend your choice of spot vs. on-demand instances for agents
- What security risks remain and how would you mitigate them?
- How would you improve this platform with 10x the resources?

**Success Criteria:**

- ✅ Platform deploys successfully to AWS
- ✅ All test stages execute correctly
- ✅ Deployment to all environments works
- ✅ Rollback procedures validated
- ✅ Monitoring and alerting operational
- ✅ Documentation complete and accurate
- ✅ Security scan passes
- ✅ Cost within budget ($200/month for dev/staging)
- ✅ Can demonstrate live to stakeholders
- ✅ Able to defend all architectural decisions

**Estimated Timeline:**

- Week 1: Infrastructure setup (Terraform + EKS)
- Week 2: CI pipeline implementation
- Week 3: CD pipeline and deployments
- Week 4: Observability, security, documentation
- Week 5: Testing, refinement, presentation prep

This capstone project represents the culmination of your Buildkite mastery journey, demonstrating your ability to design, build, and operate production-grade CI/CD systems.
