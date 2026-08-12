# Common SRE Interview Questions & Answers

> **110 questions** organized by category with concise answers and senior-engineer framing
> **Aligned with:** HSBC SRE role — Linux, Cloud, Kubernetes, DevOps, Reliability, Behavioral, System Design
>
> **Evidence note:** Behavioral examples and metrics are prompts, not candidate history. Use only personally verified details; distinguish conceptual knowledge from production experience.

---

## Table of Contents

1. [Linux (20 Questions)](#linux)
2. [Cloud (15 Questions)](#cloud)
3. [Kubernetes (20 Questions)](#kubernetes)
4. [DevOps / CI/CD (15 Questions)](#devops--cicd)
5. [Reliability / SRE (20 Questions)](#reliability--sre)
6. [Behavioral (15 Questions)](#behavioral)
7. [System Design (5 Questions)](#system-design)
8. [Troubleshooting Drills](#troubleshooting-drills)
9. [How Senior Engineers Answer](#how-senior-engineers-answer)

---

## Linux

### Q1: What is the difference between a process and a thread?

**Answer:** A process is an independent execution unit with its own memory space, file descriptors, and PID. A thread is a lightweight execution unit within a process that shares the process's memory and resources. Processes provide strong isolation; threads enable parallelism with lower overhead but shared-state risks.

**Senior framing:** "In production, I look at threads when debugging CPU-bound apps (nginx workers) and processes when investigating isolation issues or resource limits."

---

### Q2: Explain Linux runlevels/systemd targets.

**Answer:** Systemd targets replace traditional runlevels. Key targets: `multi-user.target` (normal multi-user, no GUI), `graphical.target` (with GUI), `rescue.target` (single-user recovery), `poweroff.target`/`reboot.target`. Services are linked to targets via `[Install] WantedBy=`.

---

### Q3: A server has load average of 30 but CPU usage is 10%. What's happening?

**Answer:** Likely **I/O wait** or processes in **uninterruptible sleep (D state)**. Check `top` for `wa` column, run `iostat -x` for disk utilization, and `ps aux | awk '$8 ~ /D/'` for blocked processes.

---

### Q4: How do you find what's using port 8080?

```bash
ss -tlnp | grep :8080
lsof -i :8080
```

---

### Q5: Explain SIGTERM vs SIGKILL.

**Answer:** SIGTERM (15) requests graceful shutdown — process can clean up. SIGKILL (9) forces immediate termination — cannot be caught or ignored. Always try SIGTERM first; use SIGKILL only if process doesn't exit.

---

### Q6: What is an inode? What happens when you run out?

**Answer:** An inode stores file metadata (permissions, owner, timestamps, block pointers) but not the filename. Running out of inodes means you cannot create new files even with free disk space. Check with `df -i`.

---

### Q7: How do you investigate a process using 100% CPU?

```bash
PID=$(pgrep -f appname)
perf record -p $PID -g -- sleep 30 && perf report
strace -c -p $PID
top -H -p $PID   # check if single thread
```

---

### Q8: Explain `/proc` filesystem.

**Answer:** Virtual filesystem exposing kernel and process information. Key files: `/proc/cpuinfo`, `/proc/meminfo`, `/proc/loadavg`, `/proc/<pid>/status`, `/proc/<pid>/fd/`, `/proc/<pid>/maps`.

---

### Q9: What causes zombie processes? How do you fix them?

**Answer:** Zombie (Z state) occurs when a child process exits but parent hasn't called `wait()`. Fix by restarting the **parent process** (not the zombie). Persistent zombies indicate a bug in the parent application.

---

### Q10: How do you debug "No space left on device" when `df -h` shows space available?

**Answer:** Check inode exhaustion (`df -i`), deleted but open files (`lsof +L1 | grep deleted`), disk quotas, or reserved blocks (`tune2fs -l /dev/sda1 | grep Reserved`).

---

### Q11: Explain file permissions 755 and 644.

**Answer:** 755 = owner rwx, group r-x, other r-x (directories, executables). 644 = owner rw-, group r--, other r-- (regular files). First digit is special bits (SUID/SGID/sticky).

---

### Q12: What is swap? When is it a problem?

**Answer:** Swap extends RAM using disk. Problem when actively swapping (si/so in vmstat non-zero) — causes severe latency. Better to OOM-kill or scale up than thrash swap in production.

---

### Q13: How do you trace system calls of a running process?

```bash
strace -p <PID> -f -e trace=network,file
strace -c -p <PID>   # summary
```

---

### Q14: Explain TCP TIME_WAIT state.

**Answer:** After connection closes, socket stays in TIME_WAIT (typically 60s) to ensure late packets don't contaminate new connections. High counts are normal after heavy traffic. Problematic only if ephemeral ports exhaust.

---

### Q15: How do you check disk I/O performance?

```bash
iostat -x 2       # %util, await, r/s, w/s
iotop             # per-process I/O
fio --name=test --rw=readwrite --bs=4k --size=1G --runtime=30
```

---

### Q16: What is the difference between hard links and soft links?

**Answer:** Hard link: another name for same inode (same file). Cannot cross filesystems or link directories. Soft link (symlink): pointer to another path — can cross filesystems, can break if target deleted.

---

### Q17: How do you analyze systemd service failure?

```bash
systemctl status service-name
journalctl -u service-name -b --no-pager
journalctl -u service-name --since "1 hour ago" -p err
systemd-analyze critical-chain service-name
```

---

### Q18: Explain the Linux boot process briefly.

**Answer:** BIOS/UEFI → bootloader (GRUB) → kernel load → init/systemd (PID 1) → target (multi-user) → services start per dependency order.

---

### Q19: How do you limit resources for a process?

**Answer:** cgroups (v2): CPU, memory, I/O limits. In systemd: `CPUQuota=`, `MemoryMax=` in service unit. In Kubernetes: `resources.limits` in pod spec. Legacy: `ulimit`.

---

### Q20: What is `ulimit` and what limits matter for servers?

**Answer:** Per-process resource limits. Key ones: `open files` (nofile — critical for high-connection servers), `max user processes`, `core file size`. Check: `ulimit -a`.

---

## Cloud

### Q21: Explain VPC, subnet, and availability zone.

**Answer:** VPC is an isolated virtual network. Subnets are IP range segments within a VPC, each tied to one AZ. Multi-AZ deployment spreads resources across physically separate data centers for HA.

---

### Q22: Security Group vs NACL — when to use which?

**Answer:** Security Groups: instance-level, stateful, allow-only rules — primary defense. NACLs: subnet-level, stateless, allow/deny — additional network boundary. SGs handle 95% of cases; NACLs for compliance/subnet isolation.

---

### Q23: What is the difference between ALB and NLB?

**Answer:** ALB (L7): HTTP routing, path/host-based routing, SSL termination, WAF integration. NLB (L4): TCP/UDP, extreme performance, static IPs, preserves source IP. Use ALB for HTTP APIs; NLB for non-HTTP or latency-sensitive TCP.

---

### Q24: How does autoscaling work in cloud?

**Answer:** Cloud ASG/LIG monitors metrics (CPU, custom). Scale-out when threshold exceeded; scale-in when below threshold (with cooldown). In K8s: HPA scales pods → Cluster Autoscaler adds nodes → Cloud ASG provisions VMs.

---

### Q25: Explain IAM least privilege principle.

**Answer:** Grant minimum permissions needed for the task. Use roles (not long-lived keys), scope policies to specific resources, regular access reviews, separate dev/staging/prod accounts.

---

### Q26: What is a NAT Gateway used for?

**Answer:** Allows instances in private subnets to initiate outbound internet connections without being directly reachable from the internet. Inbound traffic uses ALB/NLB in public subnets.

---

### Q27: How do you design for multi-region disaster recovery?

**Answer:** Define RTO/RPO per tier. Options: backup/restore (cheapest, highest RTO), pilot light, warm standby, active-active. Use DNS failover or global load balancing. Test DR quarterly.

---

### Q28: S3 vs EBS — when to use each?

**Answer:** S3: object storage, unlimited scale, 11 9s durability — logs, backups, static assets. EBS: block storage attached to EC2 — databases, boot volumes, low-latency random I/O.

---

### Q29: What are cloud shared responsibility models?

**Answer:** Cloud provider secures **of** the cloud (hardware, hypervisor, physical). Customer secures **in** the cloud (OS patching, network config, IAM, application, data encryption).

---

### Q30: How do you estimate cloud costs for a new service?

**Answer:** Calculate: compute (instances × hours), storage (GB/month), data transfer (egress especially), managed services (RDS, LB hours). Use pricing calculator. Add 20-30% buffer. Tag resources for cost allocation.

---

### Q31: Explain DNS failover for HA.

**Answer:** Route 53 health checks monitor endpoints. If primary fails, DNS routes to secondary. TTL affects failover speed (lower TTL = faster but more queries). Combine with load balancer health checks.

---

### Q32: What is a bastion/jump host? Why use it?

**Answer:** Hardened server in public subnet for SSH access to private instances. Centralizes access control, logging, and eliminates direct SSH to production servers. Better: use SSM Session Manager (no SSH keys needed).

---

### Q33: How do you secure data at rest and in transit in cloud?

**Answer:** At rest: encrypt EBS/S3/RDS (KMS keys). In transit: TLS everywhere, VPC endpoints (avoid internet for AWS API calls), mTLS for service-to-service.

---

### Q34: Explain spot/preemptible instances for cost optimization.

**Answer:** Unused cloud capacity at 60-90% discount. Can be terminated with notice. Good for: batch jobs, CI runners, stateless workers. Not for: databases, single-instance critical services. Use mixed with on-demand.

---

### Q35: What is a VPC endpoint?

**Answer:** Private connection to AWS services without traversing internet. Gateway endpoint (S3, DynamoDB) or Interface endpoint (most services). Reduces data transfer costs and improves security.

---

## Kubernetes

### Q36: Explain Kubernetes architecture.

**Answer:** Control plane (API server, etcd, scheduler, controller manager) manages cluster state. Worker nodes run kubelet (pod lifecycle), kube-proxy (network rules), and CNI (pod networking). Desired state reconciliation is core pattern.

---

### Q37: Pod vs Deployment vs StatefulSet?

**Answer:** Pod: single instance, no self-healing. Deployment: stateless, scalable, rolling updates. StatefulSet: stable network identity, ordered deployment, persistent storage (databases, queues).

---

### Q38: What are liveness vs readiness probes?

**Answer:** Liveness: "Is the process alive?" — failure restarts container. Readiness: "Can it serve traffic?" — failure removes from Service endpoints. Readiness should check dependencies; liveness should be lightweight.

---

### Q39: Explain Kubernetes networking model.

**Answer:** Every pod gets unique IP. Pods communicate directly without NAT. Services provide stable DNS + load balancing via kube-proxy. CNI plugin implements pod network on each node.

---

### Q40: What is a PodDisruptionBudget?

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: payment-api-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: payment-api
```
Ensures minimum pods available during voluntary disruptions (node drain, cluster upgrade).

---

### Q41: How do you debug CrashLoopBackOff?

```bash
kubectl describe pod <pod>    # events, exit code, OOMKilled?
kubectl logs <pod> --previous # logs from crashed container
# Exit 137 = OOMKilled, Exit 1 = app error, Exit 143 = SIGTERM
```

---

### Q42: Explain RBAC in Kubernetes.

**Answer:** Role/ClusterRole defines permissions (verbs on resources). RoleBinding/ClusterRoleBinding assigns roles to users/groups/service accounts. Principle of least privilege — dedicated SA per app.

---

### Q43: What is etcd and why is it critical?

**Answer:** Distributed key-value store holding all cluster state. Loss of etcd quorum = cluster control plane failure. Backups critical. Run odd number of nodes (3 or 5) for quorum.

---

### Q44: How does HPA work?

**Answer:** Horizontal Pod Autoscaler watches metrics (CPU, memory, custom). Compares to targets, calculates desired replicas, updates Deployment/StatefulSet. Needs metrics-server or Prometheus adapter for custom metrics.

---

### Q45: Explain ConfigMap vs Secret.

**Answer:** ConfigMap: non-sensitive configuration (plain text in etcd). Secret: sensitive data (base64 encoded in etcd, encrypted at rest with encryption provider). Production: use external secrets manager (Vault, AWS SM).

---

### Q46: What is a NetworkPolicy?

**Answer:** Pod-level firewall. Defines ingress/egress rules by pod labels, namespaces, CIDR blocks. Default: all traffic allowed. Requires CNI that supports NetworkPolicy (Calico, Cilium).

---

### Q47: How do you safely drain a node?

```bash
kubectl cordon node-name          # mark unschedulable
kubectl drain node-name \
  --ignore-daemonsets \
  --delete-emptydir-data \
  --grace-period=300
# Verify PDBs respected, pods rescheduled
kubectl uncordon node-name        # when maintenance done
```

---

### Q48: Explain resource requests vs limits.

**Answer:** Requests: guaranteed resources, used for scheduling. Limits: maximum allowed. CPU: throttled beyond limit. Memory: OOMKilled beyond limit. Always set requests; set limits to prevent noisy neighbors.

---

### Q49: What is the difference between Ingress and LoadBalancer Service?

**Answer:** LoadBalancer Service: one external LB per service (expensive, many LBs). Ingress: one LB routing to multiple services by hostname/path (cost-effective, centralized TLS).

---

### Q50: How do you manage Kubernetes upgrades?

**Answer:** Upgrade control plane first, then worker nodes one at a time. Drain → upgrade → uncordon. Test in non-prod. Check API deprecation warnings. Use managed K8s (EKS/GKE) for control plane upgrades.

---

### Q51: What is a DaemonSet?

**Answer:** Ensures one pod runs on every (or selected) node. Use cases: log collectors (Fluentd), monitoring agents (node-exporter), CNI plugins, security agents.

---

### Q52: Explain Helm's purpose.

**Answer:** Package manager for Kubernetes. Charts template K8s manifests with values. Enables versioning, rollback, dependency management. `helm upgrade --install` for idempotent deployments.

---

### Q53: What is GitOps?

**Answer:** Git as single source of truth for infrastructure and deployments. Changes via PR, automated sync to cluster (ArgoCD/Flux). Benefits: audit trail, rollback via Git revert, drift detection.

---

### Q54: How do you handle secrets in Kubernetes production?

**Answer:** External Secrets Operator syncing from Vault/AWS SM. Never in Git. Rotate regularly. Restrict RBAC on Secret resources. Enable encryption at rest. Use sealed-secrets for GitOps workflows.

---

### Q55: What causes pods stuck in Pending state?

**Answer:** Insufficient resources (CPU/memory), no matching node selector/affinity, PVC not bound, resource quotas exceeded, taints without tolerations. Check: `kubectl describe pod` → Events section.

---

## DevOps / CI/CD

### Q56: CI vs CD — explain the difference.

**Answer:** CI: automatically build and test on every commit (integration). CD (Delivery): code always deployable, manual release. CD (Deployment): automatic production deploy on passing pipeline.

---

### Q57: What makes a good CI/CD pipeline?

**Answer:** Fast feedback (< 10 min for CI), reliable (no flaky tests), secure (scanning, secrets management), auditable (who deployed what when), rollback-capable, environment parity.

---

### Q58: How do you handle database migrations in CI/CD?

**Answer:** Backward-compatible migrations only (expand-contract pattern). Run migrations before app deploy. Test rollback. Never destructive migrations in automated pipeline without manual gate.

---

### Q59: Explain Infrastructure as Code benefits.

**Answer:** Reproducible, version-controlled, reviewable (PR), testable (plan/validate), self-documenting, drift detection. Eliminates manual "snowflake" servers.

---

### Q60: Terraform state — why is it important?

**Answer:** Maps real-world resources to config. Enables plan/apply and prevents duplicate creation. Must be remote with locking (S3 + DynamoDB) for team use. Never commit secrets in state.

---

### Q61: How do you implement deployment rollback?

**Answer:** K8s: `kubectl rollout undo`. GitOps: revert Git commit. Docker: redeploy previous image tag. Database: backward-compatible migrations. Always test rollback procedure.

---

### Q62: What is a canary deployment?

**Answer:** Route small percentage of traffic to new version. Monitor error rate/latency. Gradually increase if healthy; rollback if metrics degrade. Tools: Istio, Flagger, Argo Rollouts.

---

### Q63: How do you manage pipeline secrets?

**Answer:** CI/CD native secret stores (GitLab CI variables, Jenkins credentials). Inject at runtime, never log. Rotate regularly. Separate secrets per environment. Audit access.

---

### Q64: Explain blue-green vs rolling deployment.

**Answer:** Blue-green: two identical environments, instant switch, easy rollback, 2x resource cost. Rolling: gradual replacement, no extra resources, slower rollback, potential mixed-version issues.

---

### Q65: How do you reduce pipeline execution time?

**Answer:** Parallel stages, caching (dependencies, Docker layers), selective testing (only affected modules), smaller test suites in CI + comprehensive in nightly, pre-built base images.

---

### Q66: What is trunk-based development?

**Answer:** Short-lived feature branches merged to main frequently (daily). Requires strong CI, feature flags, and automated testing. Reduces merge conflicts and integration pain.

---

### Q67: How do you scan for vulnerabilities in CI?

**Answer:** Container scanning (Trivy, Snyk), dependency scanning (Dependabot), SAST (Semgrep, CodeQL), DAST for web apps. Fail pipeline on critical CVEs. Track and remediate.

---

### Q68: Explain Ansible vs Terraform.

**Answer:** Terraform: declarative, provisions infrastructure (create/destroy), state management. Ansible: procedural, configures existing systems (install packages, edit configs). Often used together: Terraform creates, Ansible configures.

---

### Q69: How do you ensure deployment audit compliance?

**Answer:** Every deployment logged with: who, what (commit SHA), when, where (environment), approval chain. Immutable artifacts. Change management tickets linked. Git history as audit trail.

---

### Q70: What is idempotency and why does it matter in automation?

**Answer:** Running the same operation multiple times produces the same result. Critical for automation safety — scripts can be re-run without side effects. Ansible, Terraform, and Kubernetes are idempotent by design.

---

## Reliability / SRE

### Q71: SLI vs SLO vs SLA?

**Answer:** SLI: what we measure (latency, error rate). SLO: internal target (99.9% availability). SLA: contractual commitment with consequences. SLO should be stricter than SLA.

---

### Q72: What is an error budget?

**Answer:** Allowed unreliability: 100% - SLO. 99.9% SLO = 0.1% budget ≈ 43 min/month. When exhausted, freeze features and focus on reliability.

---

### Q73: Explain the four golden signals.

**Answer:** Latency (response time), Traffic (demand), Errors (failure rate), Saturation (resource utilization). Monitor these for every service.

---

### Q74: What is a blameless postmortem?

**Answer:** Incident review focusing on systems and processes, not individuals. Timeline, root cause, action items with owners. Goal: learn and prevent recurrence, not assign blame.

---

### Q75: How do you define SLOs for a new service?

**Answer:** 1) Identify user journeys. 2) Choose SLIs (availability, latency, correctness). 3) Set targets based on user expectations and dependencies. 4) Define error budget policy. 5) Build dashboards and alerts.

---

### Q76: What is toil? How do you reduce it?

**Answer:** Manual, repetitive, automatable, tactical work with no enduring value. Measure percentage (target ≤ 50%). Identify top tasks by frequency × time. Automate highest-impact first.

---

### Q77: Explain circuit breaker pattern.

**Answer:** When failures exceed threshold, circuit "opens" — fail fast without calling downstream. After timeout, "half-open" tests one request. Success closes circuit. Prevents cascading failures.

---

### Q78: CAP theorem — explain with example.

**Answer:** During network partition, choose Consistency or Availability (Partition tolerance is mandatory). Banking balances: CP (consistent). Product catalog cache: AP (available, eventually consistent).

---

### Q79: What is MTTR vs MTBF vs MTTA?

**Answer:** MTTR: Mean Time to Repair/Recover. MTBF: Mean Time Between Failures. MTTA: Mean Time to Acknowledge. SRE focuses on reducing MTTR and MTTA.

---

### Q80: How do you design alerting?

**Answer:** Alert on symptoms (user impact), not causes. Every alert must be actionable with runbook. Route by severity. Reduce noise (aggregation, inhibition). Review and tune weekly.

---

### Q81: What is observability vs monitoring?

**Answer:** Monitoring: predefined dashboards/alerts for known issues. Observability: ability to explore and ask arbitrary questions (via metrics, logs, traces). Need both.

---

### Q82: Explain RTO and RPO.

**Answer:** RPO: max acceptable data loss (backup frequency). RTO: max acceptable downtime (recovery speed). Define per service tier; design DR accordingly.

---

### Q83: How do you handle an incident?

**Answer:** Detect → Triage (severity) → Assign IC → Mitigate (rollback/scale) → Resolve → Communicate → Postmortem. Mitigate first, root cause later.

---

### Q84: What is chaos engineering?

**Answer:** Intentionally inject failures to test system resilience. Start in non-prod. Hypothesis-driven: "System survives AZ failure." Tools: Chaos Monkey, Litmus, Gremlin.

---

### Q85: How do you prioritize reliability work?

**Answer:** Error budget status, incident frequency/severity, toil percentage, business impact, regulatory requirements. When budget exhausted, reliability > features.

---

### Q86: Explain retry with exponential backoff.

**Answer:** On failure, wait before retrying: 1s, 2s, 4s, 8s... plus jitter (randomness). Prevents thundering herd. Set max retries and timeout. Use retry budgets.

---

### Q87: What is a runbook?

**Answer:** Step-by-step guide for responding to specific alerts/incidents. Symptoms, immediate actions, escalation path, verification steps. Linked from every alert.

---

### Q88: How do you measure SRE team effectiveness?

**Answer:** SLO achievement rate, MTTR trend, toil percentage, deployment frequency, change failure rate, alert noise ratio, postmortem action item completion.

---

### Q89: What is the difference between proactive and reactive reliability?

**Answer:** Proactive: SLOs, capacity planning, chaos testing, automation. Reactive: incident response, firefighting. Target: shift from reactive to proactive over time.

---

### Q90: Explain the concept of "nines" in availability.

**Answer:** Each nine = 10× less downtime. 99% = 3.65 days/year. 99.9% = 8.76 hours. 99.99% = 52 min. Each additional nine costs exponentially more. Match nines to business need.

---

## Behavioral

### Q91: Tell me about a production incident you handled.

**Use STAR.** Focus on: systematic approach, communication, mitigation speed, postmortem learnings. Example structure in [08_behavioral_star_method.md](./08_behavioral_star_method.md).

---

### Q92: Describe a time you automated a manual process.

**Use STAR.** Quantify toil reduction. Show initiative. Mention idempotency and testing.

---

### Q93: How do you handle disagreements with developers about reliability priorities?

**Answer:** Data-driven: show SLO metrics, error budget status, incident history. Frame as shared goal (user experience). Propose compromises (canary, feature flags). Escalate only when needed.

---

### Q94: Tell me about a time you had to learn a new technology quickly.

**Use STAR.** Show structured learning approach: docs, hands-on lab, ask experts, deliver incrementally.

---

### Q95: What is your greatest strength?

**Answer:** Choose one relevant to SRE (systematic troubleshooting, automation mindset, calm under pressure). Give specific impact example.

---

### Q96: What is your weakness?

**Answer:** Real weakness you're actively improving. Example: "I sometimes dive too deep into root cause during incidents. I've learned to mitigate first, investigate later, and time-box debugging."

---

### Q97: Why do you want to work at HSBC?

**Answer:** Reference HSBC's global scale, technology transformation, commitment to [specific value from website/YouTube]. Connect your skills (K8s, SRE practices) to their infrastructure needs.

---

### Q98: How do you handle on-call stress?

**Answer:** Runbooks reduce uncertainty. Escalation paths are clear. Post-incident learning reduces repeat stress. Work-life balance through fair rotation. Practice incident response via game days.

---

### Q99: Describe your ideal team culture.

**Answer:** Blameless, learning-oriented, data-driven decisions, automation-first, shared on-call responsibility, documentation culture. Align with HSBC values.

---

### Q100: Where do you see yourself in 3-5 years?

**Answer:** Growing as senior SRE: leading reliability initiatives, mentoring, driving platform improvements. Align with HSBC career paths (senior SRE, team lead, platform engineering).

---

### Q101: How do you prioritize when everything is urgent?

**Answer:** Assess user impact and blast radius. Sev1 > Sev2 > planned work. Communicate trade-offs to stakeholders. Use error budget and SLO data to justify prioritization.

---

### Q102: Tell me about a project you're proud of.

**Use STAR.** Quantify only with a result you can defend. If no metric exists, state the verified operational outcome and how you would measure it next time.

---

### Q103: How do you stay current with technology?

**Answer:** Specific sources: SRE books (Google SRE), conferences (KubeCon), blogs, hands-on labs, internal tech talks. Mention recent learning relevant to role.

---

### Q104: Describe working with a difficult stakeholder.

**Use STAR.** Show empathy, data-driven communication, finding common ground, professional persistence.

---

### Q105: Why SRE over DevOps?

**Answer:** SRE applies engineering rigor to reliability — SLOs, error budgets, data-driven decisions. DevOps is broader culture/practice. SRE is a specific implementation with measurable outcomes.

---

## System Design

### Q106: Design a highly available API.

**Framework:** Requirements → API design → Data model → High-level architecture → Deep dive (LB, caching, DB replication) → Failure modes → Monitoring.

Key components: L7 LB, multi-AZ deployment, auto-scaling, read replicas, caching (Redis), circuit breakers, health checks, SLO monitoring.

---

### Q107: Design a monitoring and alerting system.

Components: Metrics collection (Prometheus), log aggregation (ELK), tracing (Jaeger), dashboards (Grafana), alert routing (Alertmanager → PagerDuty), SLO tracking, runbook integration.

---

### Q108: Design a CI/CD pipeline for a microservice.

Git push → lint → test → build image → scan → push registry → deploy staging (GitOps) → smoke test → manual approval → canary deploy → SLO check → promote/rollback.

---

### Q109: Design a logging pipeline for 1000+ microservices.

Agents (Fluentd/Fluent Bit) on each node → Kafka (buffer) → Elasticsearch (storage) → Kibana (search). Structured JSON logs with trace IDs. Retention policies. Index lifecycle management.

---

### Q110: How would you migrate a monolith to Kubernetes?

1) Containerize (Dockerfile). 2) Deploy to K8s as single deployment. 3) Extract services incrementally (strangler fig). 4) Add service mesh for observability. 5) Implement GitOps. 6) Define SLOs per service.

---

## Troubleshooting Drills

### Drill 1: API returning 502 Bad Gateway

```
Check chain:
1. LB health checks → backend healthy?
2. App pods running? kubectl get pods
3. App logs → errors? kubectl logs
4. App listening on correct port?
5. Service selector matches pod labels?
6. NetworkPolicy blocking traffic?
7. Upstream dependency down?
```

### Drill 2: Database connections exhausted

```
1. Check connection count: SELECT count(*) FROM pg_stat_activity;
2. Find long-running queries
3. Check app connection pool settings
4. Look for connection leaks (not closing connections)
5. Scale read replicas for read traffic
6. Implement connection pooling (PgBouncer)
```

### Drill 3: Kubernetes node running out of disk

```
1. kubectl describe node → DiskPressure condition
2. SSH to node: df -h, df -i
3. Clean: docker system prune, journalctl --vacuum-size=500M
4. Check emptyDir volumes, log accumulation
5. Increase disk size or add nodes
```

---

## How Senior Engineers Answer

### Pattern 1: Structure Before Detail

**Junior:** Jumps into commands immediately.  
**Senior:** "Let me outline my approach first: I'll check user impact, then work through the stack from outside in — LB, app, dependencies, infrastructure."

### Pattern 2: Trade-offs, Not Absolutes

**Junior:** "Use Kubernetes for everything."  
**Senior:** "K8s makes sense for microservices with dynamic scaling needs. For a stable monolith with predictable load, managed PaaS might be simpler and cheaper."

### Pattern 3: Quantify Impact

**Junior:** "I improved monitoring."  
**Senior:** "I implemented SLO-based alerting that reduced pages from 15/week to 3/week while catching 2 incidents 10 minutes earlier."

### Pattern 4: Acknowledge Unknowns

**Junior:** Guesses or bluffs.  
**Senior:** "I haven't operated Flux or Argo CD in production. My adjacent experience is declarative Terraform, Helm/manifests, reviewed environment changes, and auditable CI/CD. I understand reconciliation and drift conceptually; I would validate the tool in a lab and pair on a low-risk service first."

### Pattern 5: Connect to Business

**Junior:** Talks only about technology.  
**Senior:** "For a payment service at a bank, I'd prioritize correctness and auditability over raw availability — a wrong balance is worse than a brief outage."

---

*Next: [06_system_design_sre.md](./06_system_design_sre.md)*
