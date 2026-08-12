# SRE Core Concepts — Complete Interview Guide

> **Target role:** Site Reliability Engineer at HSBC (3–5 years experience)  
> **Aligned with:** Cloud production support, Kubernetes, monitoring, CI/CD, on-call, automation

---

## Table of Contents

1. [What Is SRE?](#what-is-sre)
2. [SLI, SLO, SLA, Error Budgets](#sli-slo-sla-error-budgets)
3. [Reliability vs Availability](#reliability-vs-availability)
4. [Incident Response Lifecycle](#incident-response-lifecycle)
5. [On-Call Best Practices](#on-call-best-practices)
6. [Monitoring, Alerting, Observability](#monitoring-alerting-observability)
7. [Logging, Tracing, Metrics](#logging-tracing-metrics)
8. [Distributed Systems Fundamentals](#distributed-systems-fundamentals)
9. [Scalability Patterns](#scalability-patterns)
10. [Load Balancing (L4/L7)](#load-balancing-l4l7)
11. [Caching Strategies](#caching-strategies)
12. [High Availability Architectures](#high-availability-architectures)
13. [Disaster Recovery (RTO/RPO)](#disaster-recovery-rtorpo)
14. [How to Think Like an SRE](#how-to-think-like-an-sre)
15. [HSBC-Style Case Studies](#hsbc-style-case-studies)

---

## What Is SRE?

Site Reliability Engineering (SRE) is a discipline that applies **software engineering principles to operations problems**. Google pioneered SRE; the core idea is that **reliability is a feature**, not an afterthought.

| Role | Focus | Typical Output |
|------|-------|----------------|
| **DevOps Engineer** | Pipeline, infrastructure delivery | CI/CD, IaC, deployment automation |
| **SRE** | Reliability, scalability, toil reduction | SLOs, error budgets, incident response, capacity planning |
| **Platform Engineer** | Internal developer platform | Self-service tooling, golden paths |
| **Production Support** | Break-fix, ticket handling | Incident resolution, runbooks |

**HSBC SRE role alignment (from job description):**
- Maintain production systems for **high availability, reliability, scalability**
- Implement **monitoring, alerting, SLOs/SLIs, incident response**
- Manage **Docker/Kubernetes** containerized workloads
- Build **CI/CD pipelines** and **automate toil** (Python, Bash, Go)
- Participate in **on-call rotation** and **post-incident reviews**

```
┌─────────────────────────────────────────────────────────────┐
│                    SRE FEEDBACK LOOP                        │
│                                                             │
│   Define SLOs ──► Measure SLIs ──► Error Budget Policy     │
│        ▲                                    │               │
│        │                                    ▼               │
│   Postmortem ◄── Incident ◄── Alert ◄── Monitor            │
│        │                                                    │
│        └──► Automate / Fix Root Cause / Reduce Toil       │
└─────────────────────────────────────────────────────────────┘
```

---

## SLI, SLO, SLA, Error Budgets

### Definitions

| Term | Definition | Example (Payment API) |
|------|------------|----------------------|
| **SLI** (Service Level Indicator) | A **quantitative measure** of service behavior | `successful_requests / total_requests` over 5 min |
| **SLO** (Service Level Objective) | An **internal target** for an SLI | 99.95% availability over 30 days |
| **SLA** (Service Level Agreement) | A **contractual commitment** with consequences | 99.9% uptime or credit to customer |
| **Error Budget** | Allowed unreliability: `100% - SLO` | 99.95% SLO → 0.05% budget ≈ 21.6 min/month |

### SLI Selection — The Four Golden Signals (Google)

| Signal | What It Measures | Example SLI |
|--------|------------------|-------------|
| **Latency** | Time to serve a request | p99 < 300ms |
| **Traffic** | Demand on the system | requests/sec |
| **Errors** | Rate of failed requests | HTTP 5xx rate < 0.1% |
| **Saturation** | How "full" the system is | CPU > 80%, queue depth |

### Worked Example: HSBC-Style Retail Banking API

```
Service: Account Balance Inquiry API
SLI:     proportion of requests returning 200 within 500ms
SLO:     99.9% of requests meet SLI over rolling 28 days
SLA:     99.5% (external commitment to business unit)
Error Budget: 0.1% = ~43 minutes downtime/month
```

**Error budget policy:**

```
IF error_budget_remaining > 50%:
    → Allow feature releases, risky changes
ELIF error_budget_remaining > 10%:
    → Slow releases, increase testing
ELSE:
    → Feature freeze, focus on reliability work only
```

### Common Interview Traps

| Question | Weak Answer | Strong Answer |
|----------|-------------|---------------|
| "What's the difference between SLO and SLA?" | "They're the same" | "SLO is internal engineering target; SLA is contractual with business/customer consequences. SLO should be stricter than SLA." |
| "Should every service have 99.99% SLO?" | "Yes, always" | "No — SLO should match user expectations and cost. Over-engineering reliability wastes budget." |

---

## Reliability vs Availability

| Concept | Definition | Formula / Notes |
|---------|------------|-----------------|
| **Availability** | System is **up and responding** | `uptime / (uptime + downtime)` |
| **Reliability** | System performs **correctly over time** | Includes correctness, not just uptime |
| **Durability** | Data is **not lost** | Relevant for databases, backups |
| **Maintainability** | How easily system can be **repaired** | MTTR matters |

**Key insight:** A service can be **available but unreliable** — e.g., returning HTTP 200 with wrong balances (data corruption). In banking, **correctness > uptime**.

```
Availability "nines" reference:

  99%     = 3.65 days/year downtime
  99.9%   = 8.76 hours/year
  99.95%  = 4.38 hours/year
  99.99%  = 52.6 minutes/year
  99.999% = 5.26 minutes/year
```

---

## Incident Response Lifecycle

### Standard Phases

```
  DETECT → TRIAGE → MITIGATE → RESOLVE → LEARN
     │         │          │          │         │
  Alert/    Severity    Rollback/   Root cause  Postmortem
  User      Assign IC   Scale up    Fix         Action items
  report    Comms       Failover    Verify
```

### Severity Levels (Typical Enterprise)

| Sev | Impact | Response Time | Example (Banking) |
|-----|--------|---------------|-------------------|
| **Sev1** | Complete outage, revenue/regulatory impact | Immediate, all-hands | Core payment processing down |
| **Sev2** | Major degradation, workaround exists | < 30 min | Mobile app login failures for 20% users |
| **Sev3** | Partial impact, non-critical | < 4 hours | Internal dashboard slow |
| **Sev4** | Minor, no user impact | Next business day | Non-prod environment issue |

### Incident Commander (IC) Responsibilities

1. **Own the incident** — single decision-maker
2. **Assign roles:** Ops Lead, Comms Lead, Scribe
3. **Control scope** — prevent random debugging
4. **Time-box** — escalate if no progress in 15 min
5. **Declare resolution** — only when verified

### Communication Template

```
INCIDENT UPDATE — [Sev1] Payment API Degradation
Time:     2026-08-11 14:32 UTC
Status:   MITIGATING
Impact:   ~15% of card authorization requests timing out
Actions:  Rolled back deployment v2.4.1 → v2.4.0
Next:     Monitoring error rate for 15 min before all-clear
ETA:      14:50 UTC
IC:       Jane Smith
```

### Post-Incident Review (Blameless Postmortem)

**Required sections:**
1. Summary (what happened, duration, impact)
2. Timeline (detailed, UTC timestamps)
3. Root cause (5 Whys)
4. What went well
5. What went poorly
6. Action items (owner + due date)

**Blameless principle:** Focus on **systems and processes**, not individuals.

---

## On-Call Best Practices

### On-Call Rotation Design

```
Week 1: Engineer A (primary), Engineer B (secondary)
Week 2: Engineer B (primary), Engineer C (secondary)
Week 3: Engineer C (primary), Engineer A (secondary)

Rules:
- Max 1 primary shift per 3 weeks
- Secondary must acknowledge page within 5 min
- Escalation path defined (L2 → Manager → Vendor)
```

### Alert Quality Checklist

| Good Alert | Bad Alert |
|------------|-----------|
| Actionable — tells you what to do | "CPU high" with no context |
| Urgent — needs human now | Informational noise |
| Symptom-based | Cause-based (may be wrong) |
| Has runbook link | No documentation |

### Runbook Structure

```markdown
# Runbook: Payment API High Error Rate

## Symptoms
- Alert: `payment_api_error_rate > 1%`
- Grafana dashboard: [link]

## Immediate Actions (first 5 min)
1. Check recent deployments: `kubectl rollout history -n payments`
2. Check dependency health: fraud-service, ledger-db
3. If error spike correlates with deploy → rollback

## Escalation
- After 15 min: Page payments team lead
- After 30 min: Sev1 escalation

## Recovery Verification
- Error rate < 0.1% for 15 consecutive minutes
```

### On-Call Health Metrics

- **Pages per shift** (target: < 2 actionable pages/night)
- **MTTA** (Mean Time to Acknowledge)
- **MTTR** (Mean Time to Resolve)
- **Toil percentage** (target: < 50% of SRE time on toil)

---

## Monitoring, Alerting, Observability

### Monitoring vs Observability

| Monitoring | Observability |
|------------|---------------|
| Known-unknowns: predefined checks | Unknown-unknowns: explore any question |
| "Is CPU high?" | "Why is latency high for users in APAC?" |
| Dashboards + alerts | Metrics + logs + traces correlated |

### Three Pillars

```
         ┌──────────┐
         │ METRICS  │  ← Aggregated, time-series (Prometheus)
         └────┬─────┘
              │
    ┌─────────┼─────────┐
    │         │         │
┌───▼───┐ ┌───▼───┐ ┌───▼───┐
│ LOGS  │ │TRACES │ │PROFILING│
│(ELK)  │ │(Jaeger)│ │(perf)  │
└───────┘ └───────┘ └────────┘
```

### Alert Routing Architecture

```
Prometheus Alertmanager
        │
        ├── route: severity=critical → PagerDuty → On-call phone
        ├── route: severity=warning  → Slack #alerts
        └── route: team=payments     → Slack #payments-oncall
```

### RED Method (Services)

| Letter | Metric |
|--------|--------|
| **R**ate | Requests per second |
| **E**rrors | Failed requests per second |
| **D**uration | Latency distribution |

### USE Method (Resources)

| Letter | Metric |
|--------|--------|
| **U**tilization | % time resource busy |
| **S**aturation | Queue depth, wait time |
| **E**rrors | Error count |

---

## Logging, Tracing, Metrics

### Tool Comparison

| Tool | Type | Best For | HSBC JD Mention |
|------|------|----------|-----------------|
| **Prometheus** | Metrics | Time-series, alerting | ✅ Yes |
| **Grafana** | Visualization | Dashboards, correlation | ✅ Yes |
| **ELK (Elasticsearch, Logstash, Kibana)** | Logging | Log aggregation, search | ✅ Yes |
| **Jaeger** | Tracing | Distributed request flow | ✅ Yes |
| **Zipkin** | Tracing | Alternative to Jaeger | ✅ Yes |
| **Datadog** | All-in-one | Enterprise APM | ✅ Yes |

### Prometheus Example

```yaml
# prometheus alert rule
groups:
  - name: payment_api
    rules:
      - alert: HighErrorRate
        expr: |
          sum(rate(http_requests_total{job="payment-api",code=~"5.."}[5m]))
          /
          sum(rate(http_requests_total{job="payment-api"}[5m]))
          > 0.01
        for: 5m
        labels:
          severity: critical
          team: payments
        annotations:
          summary: "Payment API error rate above 1%"
          runbook: "https://wiki/runbooks/payment-api-errors"
```

### Structured Logging Example

```json
{
  "timestamp": "2026-08-11T14:32:01.123Z",
  "level": "ERROR",
  "service": "payment-api",
  "trace_id": "abc123def456",
  "span_id": "span789",
  "message": "Authorization failed",
  "error_code": "INSUFFICIENT_FUNDS",
  "account_id": "****1234",
  "latency_ms": 245
}
```

### Distributed Tracing Flow

```
User Request
    │
    ▼
[API Gateway] ──span──► [Auth Service] ──span──► [User DB]
    │
    └──span──► [Payment Service] ──span──► [Ledger DB]
                      │
                      └──span──► [Fraud Check Service]
```

**Interview tip:** Always mention **correlation IDs / trace IDs** linking logs, metrics, and traces.

---

## Distributed Systems Fundamentals

### CAP Theorem

In a network partition, you can only guarantee **two of three:**

| | Consistency | Availability | Partition Tolerance |
|---|-------------|--------------|---------------------|
| **C** | All nodes see same data | | |
| **A** | Every request gets response | | |
| **P** | System works despite network splits | | |

**Reality:** Partition tolerance is mandatory in distributed systems → choose **CP** or **AP**.

| System Type | Choice | Banking Example |
|-------------|--------|-----------------|
| Account balance | **CP** (Consistency) | Strong consistency required |
| Product catalog cache | **AP** (Availability) | Stale data acceptable briefly |

### Failure Modes

| Failure | Symptom | Mitigation |
|---------|---------|------------|
| **Network partition** | Split brain | Quorum, fencing tokens |
| **Cascading failure** | One service takes down others | Circuit breakers, bulkheads |
| **Thundering herd** | Cache expiry → DB overload | Jitter, request coalescing |
| **Retry storm** | Amplified load | Exponential backoff, retry budgets |
| **Poison message** | Queue consumer stuck | Dead letter queue (DLQ) |

### Circuit Breaker Pattern

```
         CLOSED (normal)
            │
            │ failures > threshold
            ▼
         OPEN (fail fast)
            │
            │ timeout expires
            ▼
      HALF-OPEN (test one request)
            │
     success │ failure
       ▼         ▼
    CLOSED     OPEN
```

---

## Scalability Patterns

### Vertical vs Horizontal Scaling

| | Vertical (Scale Up) | Horizontal (Scale Out) |
|---|---------------------|------------------------|
| **Method** | Bigger machine | More machines |
| **Limit** | Hardware ceiling | Near-unlimited |
| **Complexity** | Low | Higher (state, routing) |
| **Downtime** | Often requires restart | Rolling, zero-downtime |
| **Banking fit** | Databases (with care) | Stateless APIs, workers |

### Common Patterns

```
┌─────────────────────────────────────────────────────┐
│  LOAD BALANCER                                      │
│       │                                             │
│   ┌───┴───┬───────┬───────┐                        │
│   ▼       ▼       ▼       ▼                        │
│  App1   App2   App3   App4   ← Stateless replicas  │
│       │                                             │
│       ▼                                             │
│  ┌─────────┐    ┌──────────┐                       │
│  │  Cache  │    │ Database │                       │
│  │ (Redis) │    │ (Primary)│                       │
│  └─────────┘    └──────────┘                       │
└─────────────────────────────────────────────────────┘
```

| Pattern | Use Case |
|---------|----------|
| **Read replicas** | Read-heavy workloads (statements, history) |
| **Sharding** | Very large datasets (transaction logs) |
| **Queue-based** | Async processing (notifications, reconciliation) |
| **CQRS** | Separate read/write models |
| **Auto-scaling** | Variable traffic (payday spikes) |

---

## Load Balancing (L4/L7)

### Layer Comparison

| | L4 (Transport) | L7 (Application) |
|---|----------------|-------------------|
| **OSI Layer** | TCP/UDP | HTTP/gRPC |
| **Decisions based on** | IP, port | URL, headers, cookies |
| **Examples** | AWS NLB, HAProxy (TCP mode) | AWS ALB, NGINX, Istio |
| **SSL termination** | Pass-through or terminate | Typically terminates |
| **Routing** | Simple | Path-based, host-based, weighted |

### Algorithms

| Algorithm | Behavior | Best For |
|-----------|----------|----------|
| **Round Robin** | Equal rotation | Homogeneous backends |
| **Least Connections** | Fewest active connections | Long-lived connections |
| **Weighted** | Proportional traffic | Mixed capacity nodes |
| **IP Hash** | Same client → same backend | Session affinity (careful!) |
| **Consistent Hash** | Minimal remapping on change | Cache-heavy systems |

### Health Checks

```
LB Health Check:
  GET /health → 200 OK within 2s
  Interval: 10s
  Unhealthy threshold: 3 failures
  Healthy threshold: 2 successes

Application /health should check:
  ✓ Process running
  ✓ Database connectivity
  ✓ Critical dependencies
  ✗ NOT: Full business logic test
```

---

## Caching Strategies

| Strategy | Description | Risk |
|----------|-------------|------|
| **Cache-aside** | App reads cache, on miss reads DB, writes cache | Stale data |
| **Write-through** | Write to cache + DB simultaneously | Write latency |
| **Write-behind** | Write to cache, async flush to DB | Data loss on crash |
| **Read-through** | Cache handles DB fetch on miss | Cache complexity |

### Cache Invalidation

> "There are only two hard things in Computer Science: cache invalidation and naming things." — Phil Karlton

**TTL-based:** Simple, eventual consistency  
**Event-based:** Pub/sub invalidation on data change  
**Versioned keys:** `account:12345:v7`

### Banking Consideration

Never cache **authoritative financial balances** without strict TTL and invalidation. Cache **reference data** (branch codes, FX rates with timestamp) aggressively.

---

## High Availability Architectures

### Active-Passive (Hot Standby)

```
Primary (Active) ──replication──► Secondary (Standby)
     │                                    │
     └── heartbeat ──────────────────────┘
     
Failover: Manual or automatic (VIP/DNS switch)
RTO: Minutes to hours (depends on detection)
```

### Active-Active (Multi-Region)

```
Region A (Active) ◄──sync/async──► Region B (Active)
       │                                  │
   Users (GeoDNS)                    Users (GeoDNS)
       
Failover: Automatic via DNS/load balancer
RTO: Seconds to minutes
Complexity: Data consistency challenges
```

### Kubernetes HA (Relevant to HSBC Role)

```
┌─────────────────────────────────────────┐
│  Control Plane (3+ nodes, etcd quorum)  │
├─────────────────────────────────────────┤
│  Worker Nodes (spread across AZs)       │
│    ├── Pod replicas (anti-affinity)     │
│    ├── PDB (Pod Disruption Budget)    │
│    └── HPA (Horizontal Pod Autoscaler)  │
└─────────────────────────────────────────┘
```

---

## Disaster Recovery (RTO/RPO)

| Metric | Definition | Example |
|--------|------------|---------|
| **RPO** (Recovery Point Objective) | Max **data loss** acceptable | RPO = 1 hour → lose max 1 hour of transactions |
| **RTO** (Recovery Time Objective) | Max **downtime** acceptable | RTO = 4 hours → restore within 4 hours |

### DR Tier Classification

| Tier | RTO | RPO | Strategy |
|------|-----|-----|----------|
| **Tier 0** | < 1 min | ~0 | Active-active, sync replication |
| **Tier 1** | < 1 hour | < 15 min | Hot standby, frequent backups |
| **Tier 2** | < 4 hours | < 1 hour | Warm standby |
| **Tier 3** | < 24 hours | < 24 hours | Cold backup, restore from tape |

### DR Testing

**Never trust untested DR.** Run quarterly game days:

1. Simulate region failure
2. Execute failover runbook
3. Measure actual RTO/RPO
4. Document gaps

---

## How to Think Like an SRE

### Mental Model Checklist

When facing any reliability question, ask:

1. **What is the user impact?** (Not "what broke internally")
2. **What is the blast radius?** (One pod? One region? Global?)
3. **What is the fastest mitigation?** (Rollback > fix forward in incidents)
4. **How do we prevent recurrence?** (Automation, not heroics)
5. **What is the error budget impact?**

### Toil vs Engineering

| Toil (Eliminate) | Engineering (Invest In) |
|------------------|-------------------------|
| Manual restarts | Self-healing controllers |
| Copy-paste deployments | GitOps pipelines |
| Repeated ticket triage | Automated runbooks |
| Manual capacity checks | Auto-scaling policies |

**Google SRE guideline:** Cap operational toil at **50%** of time; invest remainder in engineering.

---

## HSBC-Style Case Studies

### Case Study 1: Payment API Latency Spike

**Situation:** Mobile banking app reports slow transfers during peak hours (18:00–20:00 local).

**Investigation path:**
1. Check SLI dashboard — p99 latency jumped from 200ms to 2s
2. Correlate with traffic — 3x normal load
3. Check saturation — DB connection pool at 100%
4. Check recent changes — new reporting query added to same DB

**Resolution:**
- Immediate: Scale connection pool, add read replica for reporting
- Short-term: Rate limit non-critical queries
- Long-term: Separate OLTP and OLAP workloads, implement SLO-based alerting

**Interview answer structure:** Symptom → Data → Root cause → Mitigation → Prevention

### Case Study 2: Kubernetes Deployment Causes Outage

**Situation:** Rolling deployment of auth-service v3.0 causes 40% login failures.

**Investigation:**
```bash
kubectl rollout history deployment/auth-service -n banking
kubectl describe pod auth-service-xyz -n banking
kubectl logs auth-service-xyz -n banking --previous
```

**Root cause:** New version requires env var `JWT_ISSUER` not set in ConfigMap.

**Prevention:**
- Pre-deploy validation (OPA/Kyverno policy)
- Canary deployment (5% → 25% → 100%)
- Automated smoke tests in CI/CD pipeline

### Case Study 3: Defining SLOs for a New Microservice

**Service:** Fraud detection API (internal, called synchronously during payments)

**Stakeholder input:**
- Payments team: "Must not add more than 100ms to checkout"
- Compliance: "Must log all decisions for 7 years"

**Proposed SLOs:**
- Availability: 99.95%
- Latency: p99 < 80ms
- Error rate: < 0.01% (false negatives handled separately)

**Error budget policy:** If budget exhausted, freeze ML model updates until reliability restored.

---

## Troubleshooting Drill

**Scenario:** Alert fires at 03:00 — `kube_pod_status_phase{phase="Failed"} > 0` in production namespace `payments`.

**Your 10-minute playbook:**

```bash
# 1. Which pods?
kubectl get pods -n payments --field-selector=status.phase=Failed

# 2. Why failed?
kubectl describe pod <pod-name> -n payments

# 3. Logs from previous container
kubectl logs <pod-name> -n payments --previous

# 4. Recent events
kubectl get events -n payments --sort-by='.lastTimestamp' | tail -20

# 5. Recent deployments?
kubectl rollout history deployment -n payments
```

**Decision tree:**
- OOMKilled → Increase limits or fix memory leak
- CrashLoopBackOff → Check logs, config, dependencies
- ImagePullBackOff → Registry/auth issue
- Evicted → Node pressure, check cluster capacity

---

## Quick Reference Tables

### Interview One-Liners

| Topic | One-Liner |
|-------|-----------|
| SLI | "What we measure" |
| SLO | "What we target" |
| SLA | "What we promise" |
| Error budget | "How much failure we can afford" |
| Blameless postmortem | "Fix the system, not blame the person" |
| Toil | "Manual, repetitive, automatable work" |
| Observability | "Ability to ask arbitrary questions of the system" |

### Tools from HSBC Job Description

| Category | Tools |
|----------|-------|
| Containers | Docker, Kubernetes, Istio, Helm |
| Cloud | AWS, Azure, GCP |
| Monitoring | Prometheus, Grafana, Datadog |
| Logging | ELK |
| Tracing | Zipkin, Jaeger |
| CI/CD | Jenkins, GitLab CI |
| GitOps | ArgoCD, FluxCD, Tekton |
| Scripting | Python, Bash, Go |

---

*Next: [02_linux_networking.md](./02_linux_networking.md)*
