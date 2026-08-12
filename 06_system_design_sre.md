# System Design for SRE — Interview Guide

> **Framework, example designs, trade-offs, failure modes**  
> **Aligned with:** HSBC reliability expectations for banking infrastructure

---

## Table of Contents

1. [System Design Framework](#system-design-framework)
2. [Design 1: Highly Available API](#design-1-highly-available-api)
3. [Design 2: Logging Pipeline](#design-2-logging-pipeline)
4. [Design 3: Monitoring System](#design-3-monitoring-system)
5. [Design 4: Scalable Microservice Platform](#design-4-scalable-microservice-platform)
6. [Common Trade-offs](#common-trade-offs)
7. [Failure Mode Analysis](#failure-mode-analysis)
8. [HSBC Reliability Expectations](#hsbc-reliability-expectations)

---

## System Design Framework

### Step-by-Step Approach (45-minute interview)

```
Minute 0-5:   CLARIFY requirements
Minute 5-10:  ESTIMATE scale (back-of-envelope)
Minute 10-15: HIGH-LEVEL design (boxes and arrows)
Minute 15-30: DEEP DIVE (2-3 components)
Minute 30-40: FAILURE MODES and mitigation
Minute 40-45: MONITORING, SLOs, and wrap-up
```

### Requirements Checklist

Always ask (or state assumptions):

| Category | Questions |
|----------|-----------|
| **Functional** | What does the system do? Who are the users? |
| **Scale** | QPS? Data volume? Number of users? Growth rate? |
| **Availability** | How many nines? Acceptable downtime? |
| **Latency** | p50, p99 targets? |
| **Consistency** | Strong or eventual? |
| **Durability** | Data loss tolerance (RPO)? |
| **Security** | Auth? Encryption? Compliance? |
| **Budget** | Cost constraints? Build vs buy? |

### Back-of-Envelope Calculations

```
Example: 10M users, 10 requests/day each
  = 100M requests/day
  = ~1,200 requests/second average
  = ~3,600 QPS peak (3× average)

Storage: 1KB per log entry × 100M/day = 100GB/day = 3TB/month

Bandwidth: 3,600 QPS × 1KB response = 3.6 MB/s = ~30 Mbps
```

---

## Design 1: Highly Available API

### Requirements (Banking Context)

- REST API for account balance inquiries
- 10,000 QPS peak, 99.95% availability
- p99 latency < 200ms
- Read-heavy (95% reads, 5% writes)
- Regulatory: audit trail, data residency (EU)

### High-Level Architecture

```
                        ┌─────────────┐
                        │  Route 53   │
                        │  (DNS)      │
                        └──────┬──────┘
                               │
                        ┌──────▼──────┐
                        │     ALB     │
                        │  (L7, TLS)  │
                        └──────┬──────┘
                               │
              ┌────────────────┼────────────────┐
              ▼                ▼                ▼
        ┌──────────┐    ┌──────────┐    ┌──────────┐
        │  AZ-a    │    │  AZ-b    │    │  AZ-c    │
        │ ┌──────┐ │    │ ┌──────┐ │    │ ┌──────┐ │
        │ │ API  │ │    │ │ API  │ │    │ │ API  │ │
        │ │ Pods │ │    │ │ Pods │ │    │ │ Pods │ │
        │ └──┬───┘ │    │ └──┬───┘ │    │ └──┬───┘ │
        │    │     │    │    │     │    │    │     │
        │ ┌──▼───┐ │    │ ┌──▼───┐ │    │ ┌──▼───┐ │
        │ │Redis │ │    │ │Redis │ │    │ │Redis │ │
        │ │Cache │ │    │ │Cache │ │    │ │Cache │ │
        │ └──────┘ │    │ └──────┘ │    │ └──────┘ │
        └────┬─────┘    └────┬─────┘    └────┬─────┘
             │               │               │
             └───────────────┼───────────────┘
                             ▼
                    ┌────────────────┐
                    │   RDS Primary  │
                    │   (Multi-AZ)   │
                    └───────┬────────┘
                            │ replication
                    ┌───────▼────────┐
                    │  Read Replica  │
                    └────────────────┘
```

### Deep Dive: Caching Strategy

```
Request flow:
  1. Client → ALB → API Pod
  2. API checks Redis cache (key: account:{id})
  3. Cache HIT → return (p99 < 5ms)
  4. Cache MISS → query read replica → populate cache (TTL: 30s) → return

Cache invalidation:
  - On write (balance update): delete cache key
  - TTL fallback: 30 seconds max staleness
  - NOT suitable for real-time balances during transactions
```

### Deep Dive: Auto-Scaling

```yaml
# HPA based on CPU + custom metric
minReplicas: 6    # 2 per AZ minimum
maxReplicas: 30
targetCPU: 70%
targetRPS: 500 per pod

# Cluster Autoscaler adds nodes when pods pending
# Scale-up: fast (30s)
# Scale-down: slow (5 min stabilization)
```

### SLO Definition

| SLI | SLO | Measurement |
|-----|-----|-------------|
| Availability | 99.95% | Successful responses / total |
| Latency | p99 < 200ms | Response time histogram |
| Error rate | < 0.05% | 5xx / total requests |

### Failure Modes

| Failure | Impact | Mitigation |
|---------|--------|------------|
| Single pod crash | None (others serve) | K8s auto-restart, PDB |
| AZ outage | 33% capacity loss | Multi-AZ, auto-scale |
| Redis failure | Increased DB load | Circuit breaker, fallback to DB |
| DB primary failure | Brief write outage | Multi-AZ RDS failover (~60s) |
| Cache stampede | DB overload | Request coalescing, jitter |

---

## Design 2: Logging Pipeline

### Requirements

- 500 microservices, 50TB logs/month
- Search by service, trace ID, log level, time range
- 90-day retention (compliance)
- p95 search query < 5 seconds

### Architecture

```
┌─────────── Kubernetes Cluster ──────────────────────────────┐
│                                                              │
│  Pod → stdout/stderr → Fluent Bit (DaemonSet on each node)  │
│                              │                               │
└──────────────────────────────┼───────────────────────────────┘
                               │ JSON structured logs
                               ▼
                    ┌─────────────────────┐
                    │       Kafka           │
                    │  (3 brokers, buffer)  │
                    │  Topic: logs-raw     │
                    └──────────┬───────────┘
                               │
                    ┌──────────▼───────────┐
                    │   Logstash / Flink   │
                    │   (parse, enrich,    │
                    │    PII redaction)    │
                    └──────────┬───────────┘
                               │
                    ┌──────────▼───────────┐
                    │   Elasticsearch      │
                    │   (hot: 7 days)      │
                    │   (warm: 30 days)    │
                    │   (cold: 90 days)    │
                    └──────────┬───────────┘
                               │
                    ┌──────────▼───────────┐
                    │      Kibana          │
                    │   (search, dashboards)│
                    └──────────────────────┘
```

### Structured Log Format

```json
{
  "@timestamp": "2026-08-11T14:32:01.123Z",
  "level": "ERROR",
  "service": "payment-api",
  "environment": "production",
  "trace_id": "abc123def456",
  "span_id": "span789",
  "message": "Payment authorization failed",
  "error_code": "TIMEOUT",
  "latency_ms": 5001,
  "account_id_hash": "sha256:abc...",
  "region": "eu-west-1"
}
```

### Index Lifecycle Management

| Phase | Age | Storage | Actions |
|-------|-----|---------|---------|
| Hot | 0-7 days | SSD | Full indexing, fast search |
| Warm | 7-30 days | HDD | Reduced replicas |
| Cold | 30-90 days | S3 snapshot | Searchable, slower |
| Delete | > 90 days | — | Compliance retention met |

### Scaling Considerations

```
Ingest rate: 50TB/month ≈ 20MB/s average, ~60MB/s peak

Kafka: 3 brokers, 12 partitions, replication factor 3
  → handles 100MB/s ingest with headroom

Elasticsearch: 6 data nodes (hot), index per service per day
  → shard size target: 20-50GB

Cost optimization: S3 for cold storage (90% cheaper than ES)
```

---

## Design 3: Monitoring System

### Requirements

- Monitor 500 services across 3 K8s clusters
- SLI/SLO tracking with error budget dashboards
- Alert routing by team and severity
- Integration with PagerDuty and Slack

### Architecture

```
┌─────────── Kubernetes Clusters ─────────────────────────────┐
│                                                              │
│  App Pods → /metrics endpoint                               │
│       │                                                      │
│  Prometheus (per cluster) ← ServiceMonitor (auto-discovery) │
│       │                                                      │
│  Node Exporter, kube-state-metrics                          │
│                                                              │
└──────────────────────────┬───────────────────────────────────┘
                           │ remote write (federated)
                           ▼
                ┌─────────────────────┐
                │  Thanos / Cortex     │
                │  (long-term storage, │
                │   global view)       │
                └──────────┬───────────┘
                           │
              ┌────────────┼────────────┐
              ▼            ▼            ▼
        ┌──────────┐ ┌──────────┐ ┌──────────┐
        │ Grafana  │ │Alertmgr  │ │ SLO      │
        │Dashboards│ │          │ │ Dashboard│
        └──────────┘ └────┬─────┘ └──────────┘
                          │
              ┌───────────┼───────────┐
              ▼           ▼           ▼
        ┌──────────┐ ┌────────┐ ┌──────────┐
        │PagerDuty │ │ Slack  │ │ Email    │
        │(Sev1/2)  │ │(Sev3+) │ │(reports) │
        └──────────┘ └────────┘ └──────────┘
```

### SLO Dashboard Design

```
┌─── Payment API SLO Dashboard ──────────────────────────────┐
│                                                             │
│  Availability (28-day rolling)                              │
│  ████████████████████░░  99.92%  (SLO: 99.9%)  ✅          │
│  Error budget remaining: 72%                                │
│                                                             │
│  Latency p99 (28-day rolling)                               │
│  ██████████████████████  185ms   (SLO: 200ms)  ✅          │
│                                                             │
│  Error Budget Burn Rate (1h / 6h / 24h)                   │
│  1h: 0.5x  │  6h: 1.2x  │  24h: 0.8x                       │
│                                                             │
│  Recent Alerts: 2 warnings (auto-resolved)                │
│  Last Incident: 3 days ago (Sev3, 12 min)                  │
└─────────────────────────────────────────────────────────────┘
```

### Alert Design Principles

```yaml
# Good alert: symptom-based, actionable
- alert: PaymentAPIHighErrorRate
  expr: |
    sum(rate(http_requests_total{service="payment-api",code=~"5.."}[5m]))
    / sum(rate(http_requests_total{service="payment-api"}[5m])) > 0.01
  for: 5m
  labels:
    severity: critical
  annotations:
    summary: "Payment API error rate > 1%"
    runbook: "https://wiki/runbooks/payment-api-errors"
    dashboard: "https://grafana/d/payment-api"

# Bad alert: cause-based, noisy
- alert: CPUHigh    # ← every service, not actionable alone
  expr: cpu_usage > 80%
```

---

## Design 4: Scalable Microservice Platform

### Requirements (HSBC Platform Context)

- Platform team supports 50+ development teams
- Self-service deployment to K8s
- GitOps workflow with guardrails
- Standard observability stack included

### Platform Architecture

```
┌──────────── Developer Experience ────────────────────────────┐
│                                                               │
│  Developer → Git Push → CI Pipeline (GitLab CI)              │
│       │         │                                             │
│       │         ├── Build + Test + Scan                      │
│       │         └── Update K8s manifests in Git              │
│       │                    │                                  │
│       │                    ▼                                  │
│       │           ┌──────────────┐                           │
│       │           │   ArgoCD     │                           │
│       │           │  (GitOps)    │                           │
│       │           └──────┬───────┘                           │
│       │                  │                                    │
│       ▼                  ▼                                    │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │              Kubernetes Cluster (EKS)                     │ │
│  │                                                          │ │
│  │  ┌─────────┐  ┌──────────┐  ┌───────────┐             │ │
│  │  │ Istio   │  │Prometheus│  │ Fluent Bit│             │ │
│  │  │ (mesh)  │  │ (metrics)│  │ (logs)    │             │ │
│  │  └─────────┘  └──────────┘  └───────────┘             │ │
│  │                                                          │ │
│  │  ┌──────────────────────────────────────────────┐      │ │
│  │  │  Team A namespace  │  Team B namespace  │...│      │ │
│  │  │  ┌──────┐ ┌──────┐│  ┌──────┐ ┌──────┐  │  │      │ │
│  │  │  │Svc 1 │ │Svc 2 ││  │Svc 3 │ │Svc 4 │  │  │      │ │
│  │  │  └──────┘ └──────┘│  └──────┘ └──────┘  │  │      │ │
│  │  └──────────────────────────────────────────────┘      │ │
│  │                                                          │ │
│  │  Guardrails: NetworkPolicy, ResourceQuota, PSS, OPA     │ │
│  └─────────────────────────────────────────────────────────┘ │
└───────────────────────────────────────────────────────────────┘
```

### Golden Path for Teams

```
1. Create service from template (Backstage/Cookiecutter)
   → Dockerfile, K8s manifests, CI pipeline, monitoring included

2. Push code → CI builds, tests, scans

3. Merge to main → ArgoCD deploys to staging

4. Promote to production (PR to prod overlay)

5. Observability automatic:
   - Metrics scraped via ServiceMonitor
   - Logs collected via Fluent Bit
   - Traces via Istio sidecar
   - SLO dashboard pre-built
   - Alerts routed to team channel
```

### Guardrails (Policy as Code)

```yaml
# OPA/Gatekeeper policy: require resource limits
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sRequiredResources
metadata:
  name: require-resources
spec:
  match:
    kinds:
      - apiGroups: ["apps"]
        kinds: ["Deployment"]
  parameters:
    limits:
      - cpu
      - memory
```

---

## Common Trade-offs

| Decision | Option A | Option B | When to Choose A | When to Choose B |
|----------|----------|----------|-------------------|-------------------|
| Consistency | Strong (CP) | Eventual (AP) | Financial transactions | Product catalog |
| Scaling | Vertical | Horizontal | Database (limited) | Stateless APIs |
| Deployment | Blue-green | Canary | Simple apps, instant rollback | Risk-sensitive, gradual |
| Storage | SQL | NoSQL | Relational data, ACID | Flexible schema, scale |
| Messaging | Sync (HTTP) | Async (queue) | Real-time response needed | Decoupling, buffering |
| Caching | Redis | In-memory (local) | Shared cache, consistency | Ultra-low latency |
| Monitoring | Pull (Prometheus) | Push (StatsD) | K8s, service discovery | Short-lived jobs |
| Secrets | Vault | K8s Secrets | Enterprise, rotation | Simple, dev/staging |

---

## Failure Mode Analysis

### Template

For each component, ask:

1. **What fails?** (hardware, software, network, human)
2. **What's the blast radius?** (one pod, one AZ, global)
3. **How do we detect it?** (alert, user report)
4. **How do we mitigate?** (auto-failover, manual runbook)
5. **How do we prevent recurrence?** (automation, design change)

### Example: Complete Failure Analysis

| Component | Failure | Detection | Mitigation | Prevention |
|-----------|---------|-----------|------------|------------|
| ALB | AZ outage | Health check fails | Route to other AZs | Multi-AZ ALB |
| API Pod | OOMKilled | Liveness probe fails | K8s restart | Set memory limits, fix leak |
| Redis | Node crash | Connection errors | Sentinel failover | Redis Cluster, 3+ nodes |
| RDS Primary | Hardware failure | Connection timeout | Multi-AZ auto-failover | Multi-AZ deployment |
| Kafka | Broker down | Under-replicated partitions | ISR re-election | RF=3, min.insync.replicas=2 |
| DNS | Resolution failure | External health check | Secondary DNS provider | Route 53 with health checks |

---

## HSBC Reliability Expectations

### Banking-Specific Considerations

| Requirement | Design Impact |
|-------------|---------------|
| **Regulatory compliance** | Audit logs, 7-year retention, data residency |
| **Financial correctness** | Strong consistency for transactions, idempotency |
| **Business hours peaks** | Auto-scaling for payroll, market open |
| **Change management** | Approval gates, maintenance windows |
| **Security** | mTLS, encryption at rest/transit, least privilege |
| **Multi-region** | Data sovereignty (EU data in EU) |

### Interview Closing Statement Template

> "For a banking environment like HSBC, I'd prioritize **correctness and auditability** alongside availability. Every design decision would include: SLO definition, failure mode analysis, monitoring plan, and compliance considerations. I'd start with the simplest architecture that meets requirements and evolve based on measured need — not premature optimization."

---

*Next: [07_hsbc_company_preparation.md](./07_hsbc_company_preparation.md)*
