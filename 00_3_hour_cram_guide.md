# HSBC SRE — Three-Hour Cram Guide

> **Use this document tonight.** It is a spoken-practice plan, not a complete textbook. The wording is editable; keep only details you can defend from your own experience.
>
> **Evidence rule:** Statements tagged **CV-backed** come from [`../resume.md`](../resume.md). Items tagged **transferable** are concepts you can explain but must not present as hands-on production experience. Replace every `[prompt]` with a true detail or say that the detail is not available.

## The 180-minute route

| Time | What to do | Deliverable |
|---|---|---|
| 0–5 | Read the evidence rule and role priorities | No invented tools, incidents, or metrics |
| 5–12 | Read the requirement-to-evidence map | Pick the strongest evidence for each requirement |
| 12–18 | Say “Tell me about yourself” twice | One natural 90-second answer |
| 18–25 | Say “Why SRE?” and “Why HSBC?” | Two 30–45 second answers |
| 25–35 | SLI/SLO/SLA, error budgets, burn rate | Explain without notes |
| 35–45 | Incident response and observability | Recite the investigation structure |
| 45–57 | AKS/Kubernetes and safe deployments | Answer the Kubernetes probes |
| 57–67 | CI/CD, GitOps, and rollback | Explain your real experience and tool gaps |
| 67–77 | Linux and networking | Explain an outside-in investigation |
| 77–85 | Automation and toil | Give one CV-backed example |
| 85–90 | Banking reliability and security | Recite the banking checklist |
| 90–97 | Drill: 502 responses | Speak the full investigation |
| 97–104 | Drill: CrashLoopBackOff | Speak the full investigation |
| 104–111 | Drill: high latency | Speak the full investigation |
| 111–118 | Drill: failed deployment | Speak the full investigation |
| 118–126 | Drill: identity/network failure | Speak the full investigation |
| 126–135 | Drill: saturation | Speak the full investigation |
| 135–140 | Build six story cards from the prompts | Add only true context and results |
| 140–158 | Rehearse stories 1–6, three minutes each | Six concise STAR outlines |
| 158–165 | Repeat the weakest story and gap bridges | Honest, confident wording |
| 165–175 | Run the lightning mock aloud | Score it immediately |
| 175–178 | Choose three interviewer questions | One role, one technical, one culture question |
| 178–180 | Check the interview setup | Ready five minutes before the call |

If time slips, preserve the spoken drills and lightning mock. Do not replace them with passive reading.

## 0–25 minutes — position the CV against the role

### Requirement-to-evidence map

| HSBC requirement | Evidence you can safely use | Boundary to state honestly |
|---|---|---|
| Production reliability and incident response | **CV-backed:** production SaaS environments; Azure Monitor/Log Analytics alerting; incident management and RCA on networking and identity issues | The CV does not specify on-call rotation, severity, MTTR, SLO ownership, or incident metrics |
| Docker, Kubernetes, Helm, Istio | **CV-backed:** daily AKS operation, namespace isolation, RBAC, network policies, Helm/manifests, multi-tenant workloads, zero-downtime deployment pipelines | Istio is not listed; discuss service-mesh concepts as **transferable** |
| Cloud | **CV-backed:** broad Azure infrastructure, security, networking, identity, AKS and PaaS experience | Do not imply equivalent AWS/GCP production depth |
| CI/CD | **CV-backed:** Azure DevOps YAML and GitHub Actions; build, test, and multi-environment deployment; pipeline troubleshooting with developers | Jenkins and GitLab CI are not listed; bridge from pipeline principles |
| GitOps | **CV-backed adjacent knowledge:** Terraform, declarative AKS manifests/Helm, environment promotion, auditable IaC | ArgoCD, FluxCD, and Tekton are not listed; do not say you operated them |
| Monitoring and logging | **CV-backed:** Azure Monitor, Log Analytics, alerting, incident response and RCA | Prometheus, Grafana, ELK, Jaeger/Zipkin and Datadog are not listed |
| Automation | **CV-backed:** Terraform, ARM/Bicep, PowerShell, Bash, Python-capable scripting; Azure automation reduced manual workload by roughly 35% | Do not attribute the 35% to incident response or deployment unless true |
| Collaboration and documentation | **CV-backed:** developer pipeline troubleshooting, config reviews, customer guidance, internal training, senior-engineer onboarding and knowledge transfer | Add audience size or time saved only if you know it |
| Security and regulated operations | **CV-backed:** least-privilege RBAC, managed identities, Key Vault, Conditional Access, Defender for Cloud, SOC 2/HIPAA-aligned controls, auditable Terraform | Do not claim banking-specific regulatory experience unless you have it |

### “Tell me about yourself” — 90 seconds

> “I’m an Azure-focused DevOps and platform engineer with more than seven years across cloud infrastructure, CI/CD, and production SaaS environments. In my current role I maintain Azure DevOps pipelines for Windows and AKS workloads, manage Terraform across environments, and operate AKS with namespace isolation, RBAC, network policies, and Helm or manifest deployments. I also review infrastructure and pipeline changes with development teams and help troubleshoot release failures.
>
> Previously, I guided customers through Terraform migrations and production Azure environments, and worked on monitoring, incident management, and root-cause analysis for networking and identity issues. A consistent theme in my work is making infrastructure repeatable, auditable, secure, and easier for teams to operate.
>
> I’m moving toward SRE because it gives that work a sharper reliability framework: user-focused service levels, incident learning, automation, and reduction of operational toil. This HSBC role fits my Azure, Kubernetes, infrastructure-as-code, and production troubleshooting background while giving me room to deepen the formal SRE practices and toolchain.”

Do not add team size, cluster size, request volume, uptime, MTTR, or deployment frequency unless you can verify it.

### “Why SRE?” — 30–45 seconds

> “The work I enjoy most already sits between software delivery and operations: making deployments safer, automating infrastructure, troubleshooting production issues, and improving how teams operate services. SRE adds measurable reliability through SLIs, SLOs, error budgets, and disciplined incident learning. I bring hands-on Azure, AKS, Terraform, CI/CD, identity, and monitoring experience, while being honest that I’m still deepening formal SLO practice and some of the specific open-source tools in the role.”

### “Why HSBC?” — 30–45 seconds

> “HSBC appeals to me because reliability at global banking scale has direct consequences for customer trust. The role matches my background in Azure, AKS, auditable Terraform, deployment pipelines, monitoring, and least-privilege identity. I also connect with HSBC’s four stated values: **We get it done, We value difference, We take responsibility, and We succeed together**. They map naturally to safe delivery, learning-focused incident reviews, ownership of risk, and cross-team reliability work.”

Official source: [HSBC values](https://www.hsbc.com/who-we-are/our-strategy-and-values/our-values). Avoid quoting volatile customer, country, employee, or strategy figures unless you recheck them on interview day.

## 25–90 minutes — rapid technical answers

### The universal answer shape

For a knowledge question:

1. Define it in one sentence.
2. Explain why it matters to users or the business.
3. Give a small example.
4. State a trade-off or failure mode.
5. Connect to your real experience—or label the answer as conceptual.

For any live incident:

> **Clarify impact → check recent changes → inspect signals → isolate layers → mitigate safely → verify recovery → prevent recurrence.**

Keep communication, timestamps, ownership, and escalation running in parallel with technical diagnosis.

### SLI, SLO, SLA, error budget, and burn rate

**30 seconds**

> “An SLI is a measured user-facing reliability indicator, such as successful request ratio or latency. An SLO is the target for that SLI over a window, for example 99.9% successful requests over 30 days. An SLA is an external commitment with business consequences. The allowed unreliability is the error budget; burn rate tells us how quickly we are consuming it.”

**90 seconds**

> “I would start from a critical user journey and define good events divided by valid events—for example, successful non-user-error API requests. The SLO should reflect user expectations, historical capability, and business risk, and it should be tighter internally than any SLA. A 99.9% 30-day availability SLO leaves about 43 minutes of budget, but I would alert on multi-window burn rate rather than wait for the monthly number. Fast burn catches severe incidents; slow burn catches sustained degradation. If the budget is nearly exhausted, the response is a business decision: reduce risky releases and prioritize reliability work.”

**Deeper follow-ups / checkpoints**

- Golden signals—latency, traffic, errors, saturation—are diagnostic coverage categories; they are not automatically all SLIs.
- Prefer user-visible, ratio-based SLIs at the service boundary.
- Exclude only explicitly agreed invalid events; convenient exclusions can hide pain.
- Availability is not enough: correctness and data integrity can matter more in banking.
- **Experience boundary:** “I understand and can apply the framework; my CV does not claim formal production SLO ownership.”

### Incident response and observability

**30 seconds**

> “I first establish user impact and severity, assign clear ownership, and look for recent changes. I inspect metrics, logs, and traces or available platform telemetry to isolate the failing layer. I mitigate with the lowest-risk reversible action, verify user recovery, communicate status, and then drive a blameless review with owned follow-ups.”

**90 seconds**

> “I separate coordination from diagnosis. I establish affected users, functions, regions, start time, and data-integrity risk; open the incident channel; and record decisions. Technically I compare symptoms with recent deploys or config changes, then work outside-in through DNS/load balancer, ingress, service, pods, dependencies, and infrastructure. I correlate metrics, logs, traces, and Kubernetes events. During impact I prefer rollback, traffic shift, scaling, or feature disablement over speculative fixes. Recovery means both user-level checks and stable leading indicators. The post-incident work captures contributing conditions, detection gaps, and actions with owners and dates.”

**Deeper follow-ups / checkpoints**

- Monitoring answers known questions; observability supports investigation of unanticipated states.
- Telemetry types are metrics, logs, and traces. The four golden signals are service-health categories; do not call these two lists interchangeable.
- Never prioritize root-cause elegance over safe mitigation.
- In a bank, explicitly check duplicate processing, partial writes, reconciliation, audit evidence, and customer communication.

### AKS/Kubernetes

**30 seconds**

> “My strongest hands-on match is AKS: I operate clusters day to day using namespace isolation, RBAC, network policies, Helm/manifests, and multi-environment deployment pipelines. For reliability I focus on requests and limits, readiness and liveness behavior, disruption budgets, controlled rollouts, observability, and least privilege.”

**90 seconds: readiness vs liveness vs startup**

> “Readiness controls whether a pod receives traffic; failure should remove it from service without restarting it. Liveness detects a process that cannot recover and triggers a restart, so it must not depend on fragile downstream services or it can amplify an outage. A startup probe protects slow-starting applications from premature liveness failures. I tune thresholds from measured startup and recovery behavior, and pair probes with graceful shutdown and enough rollout capacity for zero-downtime deployment.”

**Deeper follow-ups / checkpoints**

- `Deployment`: stateless replicas and rolling updates. `StatefulSet`: stable identity/storage and ordered behavior.
- Requests influence scheduling; limits constrain usage. CPU limits can throttle; memory limit breaches can cause OOM kills.
- Debug in order: pod status/events → previous logs → spec/config/secrets → resources/probes → node/network/dependencies.
- AKS isolation: namespaces are organizational boundaries, not complete security boundaries; combine RBAC, network policies, workload identity, admission policy, quotas, and appropriate node-pool/cluster separation.

### CI/CD, GitOps, and safe rollback

**30 seconds**

> “My hands-on pipeline experience is Azure DevOps YAML and GitHub Actions. The reliable flow is lint and test, build once, scan and sign, promote the same immutable artifact, apply environment controls, deploy progressively, verify health, and roll back safely. GitOps adds a reviewed Git source of truth and continuous reconciliation.”

**90 seconds**

> “I design pipelines for repeatability, traceability, and failure containment. Code and infrastructure changes are reviewed; tests and security checks block promotion; secrets come from a managed store; artifacts are immutable and traceable to a commit; and production has least-privilege identities and appropriate approvals. Deployment uses rolling, canary, or blue-green based on risk, with automated smoke and SLO checks. Rollback is pre-planned, but database changes need backward compatibility because application rollback may not reverse data changes safely. I have not claimed production ArgoCD, Flux, Tekton, Jenkins, or GitLab operation; the transferable experience is declarative Terraform/Helm, environment promotion, Azure DevOps pipelines, and auditable changes.”

**Deeper follow-ups / checkpoints**

- GitOps controller pulls desired state and detects drift; a CI pipeline commonly tests and updates the config repository.
- Tekton is a Kubernetes-native pipeline framework, not itself a GitOps reconciler in the same sense as Argo CD or Flux.
- Banking controls: separation of duties, protected branches, signed/traceable artifacts, approvals proportional to risk, evidence retention, tested rollback.
- “Zero downtime” is a design objective to validate, not an unconditional promise.

### Linux and networking

**30 seconds**

> “I start with scope and the request path, then test each boundary: DNS, TCP/TLS, load balancer or ingress, service endpoint, application, and dependencies. On the host I check CPU, memory, load, I/O, disk and inodes, sockets, process state, and recent logs. I compare with healthy instances and recent changes.”

**Useful commands to explain, not recite blindly**

```bash
dig <host>
curl -vk --connect-timeout 5 https://<host>/health
ss -lntp
top
vmstat 1
iostat -xz 1
df -h
df -i
journalctl -u <service> --since "15 min ago"
```

Know the interpretation:

- High load with low CPU can mean tasks waiting in uninterruptible I/O.
- `df -h` free but “no space left” can mean inode exhaustion (`df -i`) or deleted-open files.
- A DNS success does not prove TCP, TLS, routing, policy, or application health.
- A timeout suggests drop, route, policy, saturation, or slow dependency; “connection refused” usually means reachability exists but nothing is accepting that port.

### Automation and toil

**30 seconds**

> “I look for work that is manual, repetitive, automatable, and operationally necessary. My CV includes Terraform and Azure automation, plus PowerShell, Bash, and Python-capable scripting. One documented outcome is an Azure-based automation that reduced repetitive data-process manual workload by roughly 35%. I would explain the baseline and measurement only from details I can verify.”

**Good automation answer checkpoints**

- State the trigger, inputs, idempotency, retries/timeouts, validation, least privilege, logging, rollback, and ownership.
- Automate a stable process; do not encode an unclear or unsafe process faster.
- Add a dry-run mode and explicit failure behavior for risky changes.

### Banking reliability and security checklist

Mention the relevant items, not all of them in every answer:

- **Customer impact and trust:** availability plus correctness, confidentiality, and data integrity.
- **Auditability:** who changed what, when, why, what was approved, and which artifact ran.
- **Least privilege:** managed/workload identities, scoped RBAC, short-lived credentials, no secrets in code or logs.
- **Change safety:** peer review, separation of duties, progressive delivery, maintenance constraints, tested rollback.
- **Resilience:** dependency timeouts, bounded retries with jitter, circuit breakers, capacity headroom, zone/region failure planning.
- **Recovery:** explicit RTO (maximum acceptable restoration time) and RPO (maximum acceptable data loss), tested backups and restore exercises.
- **Data safety:** idempotency, deduplication, transactional boundaries, reconciliation, and no blind replay of financial operations.
- **Incident discipline:** evidence preservation, controlled access, clear escalation, accurate status updates, and learning without blame.

## 90–135 minutes — speak-through troubleshooting drills

For each drill, begin with impact and recent changes before commands. Finish with verification and prevention.

### 1. API returns 502

**Clarify:** Which paths, users, regions, and start time? Are all instances affected? Any deploy/config/certificate/DNS change?

**Isolate:** Client → DNS/TLS → load balancer/ingress → service/endpoints → pods → dependency.

**Checks:**

```bash
curl -vk https://<host>/<path>
kubectl get ingress,svc,endpoints,pods -n <ns> -o wide
kubectl describe ingress <name> -n <ns>
kubectl logs <pod> -n <ns> --since=15m
```

Check backend health, ingress controller logs, service selector/target port, endpoint readiness, application listening port, network policy, and upstream timeout/refusal.

**Mitigate:** Roll back a correlated change, shift traffic to healthy backends, or restore endpoint readiness. Do not randomly restart all pods.

**Verify/prevent:** User-level request succeeds; error rate and latency stabilize; add config validation, dependency alerts, and a tested runbook.

### 2. CrashLoopBackOff

**Clarify:** One pod or rollout-wide? New image/config/secret? Customer impact and healthy replica count?

**Checks:**

```bash
kubectl get pods -n <ns>
kubectl describe pod <pod> -n <ns>
kubectl logs <pod> -n <ns> --previous
kubectl get events -n <ns> --sort-by=.lastTimestamp
```

Interpret exit code, `OOMKilled`, probe failures, missing Secret/ConfigMap, bad command, permissions, dependency startup, architecture mismatch, and node conditions.

**Mitigate:** Pause/undo the rollout if correlated; keep healthy old replicas serving. Fix the evidenced cause rather than increasing restart thresholds blindly.

**Verify/prevent:** Pod remains ready through an observation window; rollout and user checks pass; add startup validation, realistic resources/probes, and canary gates.

### 3. High latency

**Clarify:** p50, p95, or p99? Which operation/region/tenant? Is error rate also rising? Traffic change?

**Inspect:** Golden signals, trace spans if available, application queue/thread/connection pools, dependency latency, node/pod CPU throttling and memory pressure, database locks/queries, network loss/retransmits.

**Isolate:** Compare healthy vs unhealthy instances and time windows; split server processing from network time; follow the slowest span or dependency.

**Mitigate:** Scale a proven bottleneck, shed optional load, disable an expensive feature, route around a dependency, or roll back a correlated change. Avoid multiplying retries.

**Verify/prevent:** User percentile returns to target without correctness issues; tune capacity, query/index, timeout/retry budget, caching, or alerting based on root cause.

### 4. Failed deployment

**Clarify:** Did build, promotion, deployment, readiness, smoke test, or post-deploy SLO gate fail? Was production traffic affected?

**Checks:** Pipeline logs and immutable artifact identity; deployment events and rollout status; manifest diff; image pull; identity/secrets/config; quota/resources; probes; database migration compatibility.

```bash
kubectl rollout status deployment/<name> -n <ns>
kubectl rollout history deployment/<name> -n <ns>
kubectl describe deployment/<name> -n <ns>
```

**Mitigate:** Stop promotion. Roll back application/config if safe; roll forward only when rollback is unsafe and the fix is understood. Treat data migrations separately.

**Verify/prevent:** Smoke and user-journey checks pass; signals stabilize; add preflight policy, progressive rollout, backward-compatible migration, and clearer failure evidence.

### 5. Identity or network failure

**Clarify:** Authentication (who are you), authorization (may you do it), name resolution, route, TCP/TLS, or policy? One workload/identity/region?

**Identity checks:** Token audience/issuer/expiry; workload or managed identity binding; RBAC scope; recent role assignment; Key Vault/access policy; clock skew. Never paste tokens into logs or tickets.

**Network checks:** DNS answer; route and peering; NSG/firewall; private endpoint and private DNS; Kubernetes NetworkPolicy; proxy; port; TLS chain/SNI.

**Mitigate:** Restore the last known-good role/policy/DNS/config through an approved path. Do not grant broad permanent access to “test.”

**Verify/prevent:** Test from the affected identity and network path; confirm audit logs; add synthetic checks, policy tests, expiry alerts, and least-privilege review.

### 6. Production saturation

**Clarify:** Which resource—CPU, memory, connections, threads, queue, disk I/O, file descriptors, or dependency quota? Demand spike or lost capacity?

**Inspect:** Traffic, queue depth, utilization and throttling, request limits, HPA status, node pressure, pool exhaustion, dependency rate limits, and per-instance skew.

**Mitigate:** Scale only if the constrained dependency can absorb it; shed low-priority load, apply backpressure/rate limits, fail fast, or shift traffic. Protect correctness and critical transactions.

**Verify/prevent:** Queues drain, latency/errors recover, no hidden data loss; capacity-test, set headroom and quotas, bound concurrency, and alert before exhaustion.

## 135–165 minutes — six CV-grounded STAR cards

These are **outlines, not completed stories**. The CV establishes the topic, not the exact situation, action sequence, or result. Fill brackets with true details before speaking. If no measurable result exists, describe the verified operational outcome without inventing a number.

### Story 1 — networking or identity incident

- **Question:** “Tell me about an incident or difficult production issue.”
- **CV anchor:** LTIMindtree incident management, Azure Monitor/Log Analytics, RCA on networking and identity issues.
- **S:** `[service/environment, symptom, user/business impact you actually observed]`
- **T:** `[your assigned responsibility; avoid claiming incident commander/on-call unless true]`
- **A:** Establish impact; inspect Monitor/Log Analytics; compare recent changes; isolate network vs identity; apply the specific safe mitigation; communicate/escalate.
- **R:** `[verified restoration or prevention outcome; actual follow-up]`
- **Probe:** What evidence ruled alternatives out? What would you improve?
- **HSBC link:** We take responsibility; audit decisions and preserve data integrity.

### Story 2 — Terraform migration

- **Question:** “Describe a complex change or improvement you delivered.”
- **CV anchor:** Guided customers through Terraform IaC migrations and production Azure environments.
- **S/T:** `[customer baseline and your exact migration responsibility]`
- **A:** Inventory and dependency mapping; module/state/environment design; plan review; secrets/RBAC; staged rollout; validation and rollback.
- **R:** Repeatable, auditable provisioning is CV-backed; add `[scope/time/quality outcome]` only if known.
- **Probe:** How did you handle Terraform state and drift?
- **HSBC link:** We get it done, with change control and evidence.

### Story 3 — safer AKS delivery

- **Question:** “How have you improved deployment reliability?”
- **CV anchor:** Azure DevOps YAML for AKS, build/test/zero-downtime deployment across dev/staging/prod; developer troubleshooting.
- **S/T:** `[real pipeline or release problem and your ownership]`
- **A:** `[actual gates]`; immutable artifact/environment promotion; readiness/rollout checks; least-privilege identity; rollback; developer collaboration.
- **R:** `[verified reliability or friction outcome; no invented percentage]`
- **Probe:** How did you validate “zero downtime”? How were database changes handled?
- **HSBC link:** We succeed together; safe, traceable releases.

### Story 4 — AKS multi-tenant isolation

- **Question:** “Tell me about Kubernetes security or platform ownership.”
- **CV anchor:** Namespace isolation, RBAC, network policies, Helm/manifests for multi-tenant AKS.
- **S/T:** `[tenant/environment risks and your exact responsibility]`
- **A:** Namespace and role design; deny-by-default/required network paths; service or managed identities; quotas/policies if actually used; deployment validation.
- **R:** Proper workload separation is CV-backed; add `[audit, incident, onboarding, or operational outcome]` only if known.
- **Probe:** Why is a namespace not a complete trust boundary?
- **HSBC link:** Least privilege and We take responsibility.

### Story 5 — automation reduced manual work

- **Question:** “Tell me about something you automated.”
- **CV anchor:** PepsiCo Azure automation reduced manual workload by roughly 35%.
- **S/T:** `[specific repetitive data process, baseline, users]`
- **A:** `[actual Azure services/code]`; validate inputs/outputs; failure handling; rollout and adoption.
- **R:** Roughly 35% less manual workload is the only CV-backed metric. Explain how it was estimated if asked.
- **Probe:** What did you deliberately leave manual and why?
- **HSBC link:** We get it done; automate safely and measure value.

### Story 6 — collaboration, review, or onboarding

- **Question:** “How do you help other engineers or handle cross-team work?”
- **CV anchor:** Reviewed developer infrastructure/pipeline changes; troubleshot failures; onboarded senior DevOps engineers; delivered knowledge transfer and training.
- **S/T:** Choose **one** real case: `[risky review]`, `[pipeline failure]`, or `[onboarding need]`.
- **A:** Listen and clarify; show evidence; pair on the solution; document the reasoning/runbook; confirm understanding.
- **R:** `[verified risk avoided, issue resolved, or readiness outcome]`
- **Probe:** What disagreement occurred? What did you learn from the other perspective?
- **HSBC link:** We value difference and We succeed together.

### Honest tool-gap bridges

Use: **truth → transferable evidence → concept → ramp plan**.

- **Prometheus/Grafana/ELK/tracing:** “My hands-on monitoring evidence is Azure Monitor and Log Analytics. I have not presented the listed open-source stack as production experience. The transferable model is instrumenting service indicators, querying correlated telemetry, alerting on actionable user impact, and building runbooks. I would map Azure metrics/KQL concepts to PromQL, dashboards, and the team’s logging/tracing conventions, then validate in a non-production service.”
- **Jenkins/GitLab CI:** “My production pipeline work is Azure DevOps YAML and GitHub Actions. The syntax and runners differ, but artifact immutability, gated stages, secret handling, promotion, auditability, and rollback transfer. I would start from an existing pipeline, make a small reviewed change, and learn the team’s shared libraries and security controls.”
- **Argo CD/Flux/Tekton:** “I use Terraform, Helm/manifests, and declarative multi-environment delivery, but I do not claim production operation of these tools. I understand reconciliation, desired state, drift and Git review. I would lab the controller, observe sync/health/rollback behavior, and pair on a low-risk service.”
- **Istio:** “I have hands-on Kubernetes networking and network-policy experience, not stated production Istio experience. I understand that a service mesh can centralize mTLS, traffic policy, retries, and telemetry, with complexity and failure-domain trade-offs. I would first learn why the team uses it and avoid unsafe retry multiplication.”
- **Go:** “My CV supports PowerShell, Bash, and Python-capable scripting, not production Go development. I can explain automation engineering—idempotency, errors, tests, observability, and safe privileges—and would build a small tested operational tool in Go to learn the team’s conventions.”

Never say “I used a similar tool” when you mean only “I understand a similar concept.”

## 165–180 minutes — lightning mock and setup

### Ten-minute lightning mock

Answer aloud without notes. Target 30–90 seconds each.

1. Tell me about yourself.
2. Why SRE, and why HSBC?
3. Explain SLI, SLO, SLA, error budget, and burn rate.
4. A release starts returning 502s. What do you do?
5. How do readiness and liveness probes differ?
6. Design a safe production deployment pipeline for a regulated bank.
7. Tell me about a networking or identity issue you investigated.
8. You have not used Argo CD in production. How would you contribute?
9. Availability is healthy but customers report duplicate transactions. What matters now?
10. Which HSBC value best matches one of your real examples?

### Self-score: 0, 1, or 2 each

| Dimension | 0 | 1 | 2 |
|---|---|---|---|
| Directness | Did not answer | Answer buried | Answer first, then support |
| Structure | Rambling | Partly ordered | Clear framework |
| Evidence | Invented/vague | Generic | CV-grounded or honestly hypothetical |
| Technical depth | Incorrect | Basic | Correct with trade-off/failure mode |
| User/business focus | Resource-only | Mentions impact | Prioritizes customer, correctness, risk |
| Incident safety | Random action | Some checks | Impact → evidence → reversible mitigation → verification |
| Banking awareness | None | Generic security | Audit, least privilege, integrity, change/recovery concerns |
| Communication | Unclear | Understandable | Concise, calm, ownership and escalation |

**Target:** 12/16 or better. Rehearse only the lowest-scoring two answers once more. A score is a practice aid, not a prediction.

### Three interviewer questions

Pick three that were not already answered:

1. “What reliability problem would you want the person in this role to improve in the first three months?”
2. “How does the team define SLOs and use error budgets when making release decisions?”
3. “What does the on-call model look like, and how are engineers supported after difficult incidents?”
4. “Which parts of the Kubernetes, observability, and GitOps stack does this team own directly?”
5. “How do HSBC’s four values show up in engineering decisions and incident reviews?”

### Two-minute setup check

- Join five minutes early; camera, microphone, charger, network, quiet notifications.
- Keep water, the job description, six story cards, and three questions visible.
- Keep secrets, customer names, confidential architecture, and internal screenshots out of answers.
- When uncertain: clarify, state assumptions, reason aloud, and say what evidence you would seek.
- When experience is missing: use the honest bridge; do not bluff.

## Optional depth only after the three-hour sprint

- [SRE core concepts](./01_sre_core_concepts.md)
- [Linux and networking](./02_linux_networking.md)
- [Cloud and Kubernetes](./03_cloud_kubernetes.md)
- [CI/CD and automation](./04_ci_cd_automation.md)
- [110 common questions](./05_common_questions_and_answers.md)
- [SRE system design](./06_system_design_sre.md)
- [HSBC company preparation](./07_hsbc_company_preparation.md)
- [Behavioral STAR method](./08_behavioral_star_method.md) — examples are templates, not candidate history
- [Questions to ask](./09_questions_to_ask_interviewer.md)
- [Final checklist](./10_final_interview_checklist.md)
