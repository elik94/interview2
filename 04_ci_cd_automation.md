# CI/CD & Automation — SRE Interview Guide

> **Target role:** HSBC SRE — Jenkins, GitLab CI, Terraform, Ansible, Python/Bash/Go automation  
> **Focus:** Pipeline design, IaC, toil reduction, production-grade automation

---

## Table of Contents

1. [CI/CD Fundamentals](#cicd-fundamentals)
2. [Build → Test → Deploy Flows](#build--test--deploy-flows)
3. [Jenkins & GitLab CI](#jenkins--gitlab-ci)
4. [Infrastructure as Code](#infrastructure-as-code)
5. [Scripting Examples](#scripting-examples)
6. [Automation Patterns](#automation-patterns)
7. [How SREs Reduce Toil](#how-sres-reduce-toil)
8. [HSBC Pipeline Expectations](#hsbc-pipeline-expectations)

---

## CI/CD Fundamentals

### Definitions

| Term | Definition |
|------|------------|
| **CI (Continuous Integration)** | Automatically build and test code on every commit |
| **CD (Continuous Delivery)** | Code is always deployable; manual release trigger |
| **CD (Continuous Deployment)** | Every passing build auto-deploys to production |
| **Pipeline** | Automated sequence of stages |
| **Artifact** | Immutable build output (Docker image, JAR, binary) |
| **Pipeline as Code** | Pipeline defined in version-controlled file |

### CI/CD Pipeline Architecture

```
┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐
│ Source  │──►│  Build  │──►│  Test   │──►│ Deploy  │──►│ Verify  │
│ (Git)   │   │         │   │         │   │         │   │         │
└─────────┘   └─────────┘   └─────────┘   └─────────┘   └─────────┘
                  │              │              │              │
                  ▼              ▼              ▼              ▼
              Docker image    Unit tests     Staging       Smoke tests
              Binary          Integration    Production    Canary analysis
                              Security scan  GitOps sync   Rollback trigger
```

### Pipeline Quality Gates

| Gate | Tool Examples | Fail Criteria |
|------|---------------|---------------|
| **Lint/Format** | golangci-lint, eslint, black | Style violations |
| **Unit Tests** | pytest, jest, go test | Coverage < threshold, test failures |
| **Integration Tests** | Testcontainers, Postman | API contract failures |
| **Security Scan** | Trivy, Snyk, SonarQube | Critical CVEs |
| **SAST** | Semgrep, CodeQL | High-severity findings |
| **Performance** | k6, JMeter | Latency regression > 10% |
| **Manual Approval** | Jenkins input, GitLab environment | Required for production |

---

## Build → Test → Deploy Flows

### Standard Flow (HSBC-Aligned)

```
Developer commits → feature branch
        │
        ▼
   CI Pipeline (on PR)
   ├── Lint
   ├── Unit tests
   ├── Build Docker image
   ├── Integration tests
   ├── Security scan (Trivy)
   └── Push to registry (tag: commit-SHA)
        │
        ▼
   Code review + approval
        │
        ▼
   Merge to main
        │
        ▼
   CD Pipeline
   ├── Build production image (tag: v1.2.3)
   ├── Deploy to staging (GitOps sync)
   ├── Smoke tests
   ├── Manual approval gate
   ├── Deploy to production (canary)
   ├── Monitor SLO metrics (15 min)
   └── Promote or rollback
```

### GitLab CI Example

```yaml
# .gitlab-ci.yml
stages:
  - lint
  - test
  - build
  - security
  - deploy-staging
  - deploy-production

variables:
  IMAGE: registry.company.com/payments/payment-api

lint:
  stage: lint
  image: golang:1.22
  script:
    - golangci-lint run ./...

unit-test:
  stage: test
  image: golang:1.22
  script:
    - go test -race -coverprofile=coverage.out ./...
    - go tool cover -func=coverage.out
  coverage: '/total:\s+\(statements\)\s+(\d+\.\d+)%/'
  artifacts:
    # Go's coverage.out is not Cobertura XML. Keep the native profile as an
    # artifact; add an explicit converter before using GitLab coverage_report.
    paths:
      - coverage.out

build:
  stage: build
  image: docker:24
  services:
    - docker:24-dind
  script:
    - docker build -t $IMAGE:$CI_COMMIT_SHA .
    - docker push $IMAGE:$CI_COMMIT_SHA
  only:
    - main
    - merge_requests

security-scan:
  stage: security
  image: aquasec/trivy:latest
  script:
    - trivy image --severity HIGH,CRITICAL --exit-code 1 $IMAGE:$CI_COMMIT_SHA

deploy-staging:
  stage: deploy-staging
  script:
    - |
      cd k8s/overlays/staging
      kustomize edit set image payment-api=$IMAGE:$CI_COMMIT_SHA
      git commit -am "Deploy $CI_COMMIT_SHA to staging"
      git push
    # ArgoCD auto-syncs from Git
  environment:
    name: staging
  only:
    - main

deploy-production:
  stage: deploy-production
  script:
    - |
      cd k8s/overlays/production
      kustomize edit set image payment-api=$IMAGE:$CI_COMMIT_SHA
      git commit -am "Deploy $CI_COMMIT_SHA to production"
      git push
  environment:
    name: production
  when: manual
  only:
    - main
```

### Jenkins Pipeline (Declarative)

```groovy
// Jenkinsfile
pipeline {
    agent any
    
    environment {
        IMAGE = 'registry.company.com/payments/payment-api'
        DOCKER_CRED = credentials('docker-registry')
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Test') {
            steps {
                sh 'go test -race ./...'
            }
        }
        
        stage('Build & Push') {
            steps {
                sh """
                    docker build -t ${IMAGE}:${GIT_COMMIT} .
                    docker push ${IMAGE}:${GIT_COMMIT}
                """
            }
        }
        
        stage('Security Scan') {
            steps {
                sh "trivy image --severity HIGH,CRITICAL --exit-code 1 ${IMAGE}:${GIT_COMMIT}"
            }
        }
        
        stage('Deploy Staging') {
            steps {
                sh """
                    kubectl set image deployment/payment-api \
                        payment-api=${IMAGE}:${GIT_COMMIT} \
                        -n payments-staging
                    kubectl rollout status deployment/payment-api -n payments-staging
                """
            }
        }
        
        stage('Deploy Production') {
            input {
                message "Deploy to production?"
                ok "Deploy"
            }
            steps {
                sh """
                    kubectl set image deployment/payment-api \
                        payment-api=${IMAGE}:${GIT_COMMIT} \
                        -n payments
                    kubectl rollout status deployment/payment-api -n payments
                """
            }
        }
    }
    
    post {
        failure {
            slackSend channel: '#payments-alerts', 
                      message: "Pipeline failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
        }
    }
}
```

---

## Infrastructure as Code

### Terraform

**Core concepts:** Providers, Resources, State, Modules, Plan/Apply

```hcl
# main.tf — EKS cluster for payment services
terraform {
  required_version = ">= 1.5"
  backend "s3" {
    bucket = "company-terraform-state"
    key    = "payments/eks/terraform.tfstate"
    region = "eu-west-1"
    dynamodb_table = "terraform-locks"
    encrypt = true
  }
}

provider "aws" {
  region = var.aws_region
}

module "eks" {
  source  = "terraform-aws-modules/eks/aws"
  version = "~> 20.0"

  cluster_name    = "payments-prod"
  cluster_version = "1.29"

  vpc_id     = module.vpc.vpc_id
  subnet_ids = module.vpc.private_subnets

  eks_managed_node_groups = {
    general = {
      min_size     = 3
      max_size     = 10
      desired_size = 5
      instance_types = ["m6i.xlarge"]
    }
  }

  tags = {
    Environment = "production"
    Team        = "payments-sre"
    ManagedBy   = "terraform"
  }
}
```

### Terraform Workflow

```bash
terraform init          # Initialize providers, backend
terraform fmt -check    # Format validation
terraform validate      # Syntax check
terraform plan          # Preview changes
terraform apply         # Apply changes
terraform state list    # List managed resources
terraform import        # Import existing resource
```

### Terraform Best Practices

| Practice | Why |
|----------|-----|
| Remote state with locking | Prevent concurrent modifications |
| Modules for reuse | DRY, consistent patterns |
| `terraform plan` in CI | Review before apply |
| Separate state per environment | Blast radius isolation |
| Never commit secrets | Use variables + secret manager |
| Pin provider versions | Reproducible builds |

### Ansible

**Use case:** Configuration management, patching, ad-hoc automation

```yaml
# playbook-deploy-monitoring-agent.yml
---
- name: Deploy Prometheus node exporter
  hosts: production
  become: yes
  vars:
    node_exporter_version: "1.7.0"
  
  tasks:
    - name: Download node exporter
      get_url:
        url: "https://github.com/prometheus/node_exporter/releases/download/v{{ node_exporter_version }}/node_exporter-{{ node_exporter_version }}.linux-amd64.tar.gz"
        dest: /tmp/node_exporter.tar.gz
    
    - name: Extract binary
      unarchive:
        src: /tmp/node_exporter.tar.gz
        dest: /usr/local/bin/
        remote_src: yes
        extra_opts: [--strip-components=1]
    
    - name: Create systemd service
      template:
        src: templates/node_exporter.service.j2
        dest: /etc/systemd/system/node_exporter.service
      notify: Restart node exporter
    
    - name: Enable and start service
      systemd:
        name: node_exporter
        enabled: yes
        state: started
  
  handlers:
    - name: Restart node exporter
      systemd:
        name: node_exporter
        state: restarted
```

```bash
# Run playbook
ansible-playbook -i inventory/production deploy-monitoring-agent.yml --check  # dry run
ansible-playbook -i inventory/production deploy-monitoring-agent.yml
```

---

## Scripting Examples

### Python — Health Check Automation

```python
#!/usr/bin/env python3
"""Automated health check for payment API endpoints."""

import sys
import time
import requests
from dataclasses import dataclass
from typing import Optional

@dataclass
class HealthResult:
    url: str
    status_code: Optional[int]
    latency_ms: float
    healthy: bool
    error: Optional[str] = None

ENDPOINTS = [
    "https://api.payments.internal/health",
    "https://api.payments.internal/ready",
]

def check_endpoint(url: str, timeout: float = 5.0) -> HealthResult:
    start = time.monotonic()
    try:
        resp = requests.get(url, timeout=timeout)
        latency = (time.monotonic() - start) * 1000
        return HealthResult(
            url=url,
            status_code=resp.status_code,
            latency_ms=latency,
            healthy=resp.status_code == 200,
        )
    except requests.RequestException as e:
        latency = (time.monotonic() - start) * 1000
        return HealthResult(
            url=url, status_code=None, latency_ms=latency,
            healthy=False, error=str(e),
        )

def main() -> int:
    results = [check_endpoint(url) for url in ENDPOINTS]
    all_healthy = all(r.healthy for r in results)
    
    for r in results:
        status = "OK" if r.healthy else "FAIL"
        print(f"[{status}] {r.url} — {r.status_code} ({r.latency_ms:.0f}ms)")
        if r.error:
            print(f"       Error: {r.error}")
    
    return 0 if all_healthy else 1

if __name__ == "__main__":
    sys.exit(main())
```

### Bash — Incident Response Helper

```bash
#!/bin/bash
# incident-snapshot.sh — Collect diagnostic info during incident
set -euo pipefail

NAMESPACE="${1:-payments}"
OUTPUT_DIR="/tmp/incident-$(date +%Y%m%d-%H%M%S)"
mkdir -p "$OUTPUT_DIR"

echo "Collecting incident snapshot for namespace: $NAMESPACE"
echo "Output: $OUTPUT_DIR"

# Cluster state
kubectl get all -n "$NAMESPACE" -o wide > "$OUTPUT_DIR/resources.txt"
kubectl get events -n "$NAMESPACE" --sort-by='.lastTimestamp' > "$OUTPUT_DIR/events.txt"
kubectl top pods -n "$NAMESPACE" > "$OUTPUT_DIR/pod-metrics.txt" 2>/dev/null || true

# Pod details
for pod in $(kubectl get pods -n "$NAMESPACE" -o jsonpath='{.items[*].metadata.name}'); do
    kubectl describe pod "$pod" -n "$NAMESPACE" > "$OUTPUT_DIR/pod-${pod}-describe.txt"
    kubectl logs "$pod" -n "$NAMESPACE" --tail=500 > "$OUTPUT_DIR/pod-${pod}-logs.txt" 2>/dev/null || true
    kubectl logs "$pod" -n "$NAMESPACE" --previous --tail=500 > "$OUTPUT_DIR/pod-${pod}-prev-logs.txt" 2>/dev/null || true
done

# Recent deployments
kubectl rollout history deployment -n "$NAMESPACE" > "$OUTPUT_DIR/rollout-history.txt"

echo "Snapshot complete: $OUTPUT_DIR"
tar -czf "${OUTPUT_DIR}.tar.gz" -C "$(dirname "$OUTPUT_DIR")" "$(basename "$OUTPUT_DIR")"
echo "Archive: ${OUTPUT_DIR}.tar.gz"
```

### Go — Simple HTTP Health Server

```go
package main

import (
    "encoding/json"
    "net/http"
    "runtime"
    "time"
)

type HealthResponse struct {
    Status    string    `json:"status"`
    Timestamp time.Time `json:"timestamp"`
    GoRoutines int      `json:"goroutines"`
    Uptime    string    `json:"uptime"`
}

var startTime = time.Now()

func healthHandler(w http.ResponseWriter, r *http.Request) {
    resp := HealthResponse{
        Status:     "healthy",
        Timestamp:  time.Now().UTC(),
        GoRoutines: runtime.NumGoroutine(),
        Uptime:     time.Since(startTime).String(),
    }
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(resp)
}

func main() {
    http.HandleFunc("/health", healthHandler)
    http.ListenAndServe(":8080", nil)
}
```

---

## Automation Patterns

### Pattern Catalog

| Pattern | Description | Example |
|---------|-------------|---------|
| **Self-healing** | Auto-restart failed components | K8s liveness probes, systemd Restart= |
| **Auto-scaling** | Scale based on demand | HPA, Cluster Autoscaler |
| **GitOps** | Git as source of truth | ArgoCD auto-sync |
| **ChatOps** | Trigger actions from Slack | `/incident declare sev2` bot |
| **Scheduled automation** | Cron-based tasks | Certificate renewal, backup |
| **Event-driven** | React to events | Lambda on S3 upload, K8s operator |
| **Runbook automation** | Codify manual steps | Ansible playbooks, Rundeck |

### Idempotency Principle

**Every automation script must be safe to run multiple times.**

```bash
# BAD — not idempotent
echo "option=value" >> /etc/app/config.conf

# GOOD — idempotent
grep -q "option=value" /etc/app/config.conf || echo "option=value" >> /etc/app/config.conf

# BETTER — use proper tools
# Ansible, Terraform, or dedicated config management
```

---

## How SREs Reduce Toil

### Toil Definition (Google SRE Book)

Toil is work that is:
- **Manual** — requires human action
- **Repetitive** — done over and over
- **Automatable** — could be done by a script
- **Tactical** — reactive, not strategic
- **No enduring value** — doesn't permanently improve the service
- **Scales linearly** — grows with service growth

### Toil Reduction Framework

```
1. MEASURE  — Track time spent on toil (ticket tags, time tracking)
2. IDENTIFY — Top 3 most frequent manual tasks
3. PRIORITIZE — By frequency × time × risk
4. AUTOMATE — Script → Pipeline → Self-service tool
5. VERIFY   — Measure toil reduction, monitor automation reliability
```

### Before/After Examples

| Toil Task | Manual (Before) | Automated (After) |
|-----------|-----------------|-------------------|
| Certificate renewal | Email alert → SSH → manual certbot | cert-manager auto-renewal |
| Scale for peak traffic | On-call manually scales HPA | Scheduled HPA + predictive scaling |
| Deploy to staging | 15-step runbook, 30 min | Git push → ArgoCD auto-sync, 3 min |
| Collect incident logs | 10 kubectl commands | `incident-snapshot.sh` one command |
| User access provisioning | Ticket → manual RBAC edit | Self-service portal + Terraform |

### Toil Budget

```
Target: ≤ 50% of SRE time on toil
Track: Weekly toil hours / total hours
Review: Monthly — identify next automation candidate
```

---

## HSBC Pipeline Expectations

### Enterprise Pipeline Requirements

| Requirement | Implementation |
|-------------|----------------|
| **Audit trail** | Every deployment logged with who, what, when |
| **Approval gates** | Manual approval for production |
| **Security scanning** | SAST, container scanning, dependency checks |
| **Immutable artifacts** | Same image SHA from staging → production |
| **Rollback capability** | One-command rollback tested regularly |
| **Environment parity** | Staging mirrors production topology |
| **Secrets management** | Vault, AWS Secrets Manager — never in Git |
| **Compliance tagging** | All resources tagged for cost/compliance tracking |
| **Change windows** | Restricted deploy times for critical services |

### Interview Talking Points

1. **"How do you ensure deployment safety?"**
   - Automated tests, security scans, canary deployments, SLO monitoring post-deploy, automated rollback on error budget burn

2. **"How do you handle secrets in CI/CD?"**
   - Never in Git; use CI/CD secret stores (GitLab CI variables, Jenkins credentials); inject at runtime; rotate regularly

3. **"Describe your ideal CI/CD pipeline."**
   - PR triggers lint + test + build; merge triggers staging deploy via GitOps; smoke tests pass → manual prod approval → canary deploy → SLO check → promote or rollback

4. **"How do you reduce toil?"**
   - Measure toil percentage; prioritize highest-frequency tasks; automate with idempotent scripts; build self-service tools; verify reduction with metrics

---

*Next: [05_common_questions_and_answers.md](./05_common_questions_and_answers.md)*
