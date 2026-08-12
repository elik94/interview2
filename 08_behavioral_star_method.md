# Behavioral Interview Preparation — STAR Method for SRE

> **11 generic STAR templates**, strengths/weaknesses, CV presentation, HSBC alignment
> **Based on:** HSBC interview preparation guidelines and competency-based interview format
>
> **Important:** The examples and metrics in this guide are fictional practice templates, not the candidate's history. Use only facts you can personally verify; use [`00_3_hour_cram_guide.md`](./00_3_hour_cram_guide.md) for CV-grounded story prompts.

---

## Table of Contents

1. [STAR Method Explained](#star-method-explained)
2. [Commercial STAR Examples](#commercial-star-examples)
3. [Non-Commercial STAR Examples](#non-commercial-star-examples)
4. [If You Have No Example — Templates](#if-you-have-no-example--templates)
5. [Strengths & Weaknesses](#strengths--weaknesses)
6. [How to Present Your CV](#how-to-present-your-cv)
7. [Reflection Prompts](#reflection-prompts)
8. [HSBC-Aligned Behavioral Expectations](#hsbc-aligned-behavioral-expectations)

---

## STAR Method Explained

### Structure

| Letter | Meaning | Tips |
|--------|---------|------|
| **S** — Situation | Set the context (where, when, what team) | Keep brief — 2-3 sentences max |
| **T** — Task | Your specific responsibility | Clarify YOUR role, not the team's |
| **A** — Action | What YOU did (step by step) | Use "I" not "we"; be specific about tools/methods |
| **R** — Result | Outcome with metrics if possible | Quantify: time saved, incidents reduced, SLO improved |

### Timing Guide

```
Total answer: 2-3 minutes
  Situation:  20-30 seconds
  Task:       15-20 seconds
  Action:     60-90 seconds  ← bulk of answer
  Result:     20-30 seconds
```

### STAR Quality Checklist

```markdown
□ Used "I" for actions (not "we")
□ Included specific tools/technologies
□ Quantified the result only when the number is known and defensible
□ Relevant to the question asked
□ Shows SRE mindset (reliability, automation, learning)
□ Under 3 minutes when spoken
```

### Technical Questions with STAR

For technical scenarios, adapt STAR:

```
Situation → Problem context
Task      → What needed to be solved
Action    → Investigation steps + solution (show systematic thinking)
Result    → Outcome + what you learned + prevention
```

---

## Commercial STAR Examples

### Example 1: Production Incident Response

**Question:** "Tell me about a time you handled a production incident."

| | |
|---|---|
| **S** | At [Company], our payment processing API started returning 15% errors during peak evening hours, affecting mobile banking transfers for approximately 50,000 users. |
| **T** | As the on-call SRE, I was paged at 19:45 and needed to mitigate the impact quickly and identify the root cause. |
| **A** | I declared a Sev2 incident and became the ops lead. First, I checked our Grafana dashboard and confirmed the error spike correlated with a deployment 30 minutes earlier. I initiated a rollback using `kubectl rollout undo` while simultaneously checking dependency health — the ledger database showed normal metrics. After rollback, error rate dropped to baseline within 5 minutes. I then analyzed logs from the failed version and found a missing environment variable causing authentication failures with the fraud detection service. I documented the timeline in our incident channel and coordinated with the dev team. |
| **R** | Service restored in 12 minutes (within our 15-minute MTTR target). I wrote a blameless postmortem with three action items: add pre-deploy config validation in CI, implement canary deployments, and add a runbook for this failure mode. Canary deployment was implemented the following sprint, and we haven't had a similar incident since. |

---

### Example 2: Automation / Toil Reduction

**Question:** "Describe a time you automated a manual process."

| | |
|---|---|
| **S** | Our team spent approximately 3 hours per week manually collecting diagnostic information during incidents — running 10+ kubectl commands and copying output to tickets. |
| **T** | I was asked to reduce incident diagnostic time and standardize our data collection process. |
| **A** | I wrote a Bash script (`incident-snapshot.sh`) that automatically collects pod status, events, logs, rollout history, and metrics for a given namespace. I integrated it into our runbooks and Slack incident bot so on-call engineers could trigger it with one command. I tested it during three real incidents, iterated based on feedback, and added it to our onboarding documentation. |
| **R** | Diagnostic collection time reduced from ~20 minutes to under 2 minutes. The script is now used by 4 teams across the organization. I estimated this saves ~10 hours/month of toil across the on-call rotation. |

---

### Example 3: Cross-Team Collaboration

**Question:** "Tell me about a time you worked with development teams to improve reliability."

| | |
|---|---|
| **S** | The payments development team was deploying 5-8 times per week, but we had 3 deployment-related incidents in one month, burning through our error budget. |
| **T** | I needed to partner with the dev team to improve deployment safety without slowing their release cadence significantly. |
| **A** | I met with the tech lead to review incident data — all three incidents were caused by config changes, not code bugs. I proposed implementing GitOps with ArgoCD and adding OPA policies to validate required environment variables before deployment. I pair-programmed with a developer to create the first OPA policy and integrated it into their GitLab CI pipeline. I also set up a canary deployment using Flagger with automatic rollback based on error rate metrics. |
| **R** | Deployment-related incidents dropped from 3/month to 0 over the next quarter. The team maintained their release cadence (5-8/week) with confidence. Error budget consumption from deployments decreased by 90%. The OPA policy pattern was adopted by two other teams. |

---

### Example 4: Implementing SLOs

**Question:** "Tell me about a time you improved system reliability."

| | |
|---|---|
| **S** | Our team managed 12 microservices but had no formal reliability targets. Alerts were noisy (40+ pages/week) and we couldn't prioritize reliability work against feature requests. |
| **T** | I was tasked with establishing an SLO framework for our most critical services. |
| **A** | I started with the payment API — our highest-traffic service. I worked with product owners to understand user expectations, then defined SLIs (availability, latency p99, error rate) and SLOs (99.9% availability, p99 < 300ms). I built Grafana dashboards showing error budget burn rate and configured Alertmanager to route alerts based on SLO impact. I presented the error budget policy to stakeholders: when budget is below 10%, feature releases require additional approval. |
| **R** | Alert noise reduced from 40 to 8 actionable pages/week. We had data-driven conversations about reliability vs features. The payment API maintained 99.94% availability over the first quarter (above SLO). The framework was extended to 5 additional services. |

---

### Example 5: Learning New Technology Quickly

**Question:** "Describe a time you had to learn something new quickly."

| | |
|---|---|
| **S** | Our organization decided to migrate from Jenkins-based deployments to GitOps with ArgoCD. I had used Jenkins extensively but had no production ArgoCD experience. |
| **T** | I needed to become proficient enough to lead the migration for our team's 8 services within 6 weeks. |
| **A** | I completed the ArgoCD documentation and set up a local lab cluster with minikube. I migrated one non-critical service end-to-end as a proof of concept, documenting every step. I attended a GitOps community meetup and consulted with a colleague who had ArgoCD experience. I created an internal guide and ran a workshop for my team before migrating production services incrementally. |
| **R** | All 8 services migrated to GitOps within 5 weeks. Deployment time reduced from 30 minutes (manual Jenkins) to 3 minutes (automated sync). Zero deployment incidents during migration. The internal guide was adopted by 3 other teams. |

---

### Example 6: Handling Disagreement

**Question:** "Tell me about a time you disagreed with a colleague."

| | |
|---|---|
| **S** | A development team wanted to deploy a major database schema change during Friday afternoon to meet a Monday deadline. |
| **T** | As SRE, I believed this violated our change management policy and posed significant risk for weekend on-call. |
| **A** | Instead of blocking outright, I scheduled a 30-minute meeting with the tech lead. I presented data: 60% of our Sev2+ incidents occurred on Friday deployments. I proposed an alternative: deploy Saturday morning with full team availability for monitoring, using a backward-compatible migration that allowed rollback. I offered to personally be on-call Saturday morning to support. |
| **R** | The team agreed to the Saturday deployment. It completed successfully with no issues. The tech lead later thanked me and adopted the "no Friday prod deploys" policy for their team. Our relationship strengthened because I offered solutions, not just objections. |

---

### Example 7: On-Call Improvement

**Question:** "How have you improved the on-call experience?"

| | |
|---|---|
| **S** | Our on-call rotation had 12-15 pages per week, with 70% being non-actionable alerts (informational CPU spikes, staging environment noise). Team morale was declining and people dreaded on-call shifts. |
| **T** | I volunteered to lead an alert quality improvement initiative. |
| **A** | I analyzed 4 weeks of alert data, categorizing each page as actionable or noise. I identified the top 5 noisiest alerts and worked with service owners to either fix thresholds, add runbooks, or remove the alert. I implemented Alertmanager inhibition rules to suppress staging alerts and route by severity. I created an "alert review" ritual in our weekly team meeting. |
| **R** | Pages reduced from 12-15/week to 3-4/week (all actionable). Team satisfaction survey showed on-call confidence increased from 5/10 to 8/10. Two other teams adopted our alert review ritual. |

---

### Example 8: Security/Compliance Awareness

**Question:** "Tell me about a time you dealt with a security or compliance requirement."

| | |
|---|---|
| **S** | An internal audit flagged that our Kubernetes secrets were stored as plain ConfigMaps in Git, violating our security policy. |
| **T** | I was assigned to remediate secrets management for our 8 services within 4 weeks. |
| **A** | I evaluated options (Sealed Secrets, External Secrets Operator with Vault) and recommended External Secrets Operator for centralized rotation and audit. I migrated one service as a pilot, created a migration runbook, and paired with developers to update their deployment workflows. I added CI checks to prevent secrets from being committed to Git (git-secrets hook). |
| **R** | All 8 services migrated within 3 weeks. Audit finding closed. Zero secrets in Git across the team. The pattern became the standard for new service onboarding. |

---

## Non-Commercial STAR Examples

Use these when you lack direct commercial experience — personal projects, open source, academic, or volunteer work.

### Example 9: Home Lab Kubernetes Project

**Question:** "Tell me about your Kubernetes experience."

| | |
|---|---|
| **S** | To build hands-on Kubernetes skills, I set up a home lab with 3 Raspberry Pi nodes running K3s. |
| **T** | I wanted to simulate a production-like environment with monitoring, GitOps, and automated deployments. |
| **A** | I deployed a sample microservices application (web frontend + API + PostgreSQL) using Helm charts. I installed Prometheus and Grafana for monitoring, configured ArgoCD for GitOps deployments from a GitHub repo, and set up Fluent Bit for log collection. I intentionally broke things — deleting pods, draining nodes, cutting network — to practice troubleshooting. |
| **R** | Gained practical experience with the full stack mentioned in the job description. Documented my setup in a GitHub repo that helped two colleagues learn Kubernetes. This directly prepared me for production K8s work. |

---

### Example 10: Open Source Contribution

**Question:** "Tell me about a technical challenge you solved."

| | |
|---|---|
| **S** | While using Prometheus at work, I encountered a bug in a community Helm chart that caused ServiceMonitor labels to conflict during upgrades. |
| **T** | I needed a fix for our team and wanted to give back to the community. |
| **A** | I reproduced the issue in a local cluster, identified the root cause in the Helm template logic, and submitted a pull request with the fix and a test case. I engaged with the maintainer's feedback and updated the PR. |
| **R** | PR merged. Fix included in the next chart release. Our team and other users benefited. Demonstrated ability to debug, contribute, and collaborate asynchronously. |

---

### Example 11: Academic / Certification Project

**Question:** "How do you stay current with technology?"

| | |
|---|---|
| **S** | I recognized that observability is critical for SRE work but I had limited production tracing experience. |
| **T** | I decided to build a deep understanding of distributed tracing through a structured learning project. |
| **A** | I completed the Grafana Tempo/Jaeger tutorials, then built a demo application (3 microservices in Go) instrumented with OpenTelemetry. I deployed it on my home lab K3s cluster with Jaeger for trace collection and Grafana for visualization. I wrote a blog post documenting the setup and key learnings about trace context propagation. |
| **R** | Can now confidently discuss tracing in interviews and production contexts. The demo app serves as a reference implementation. Completed CKA certification alongside this project. |

---

## If You Have No Example — Templates

When you genuinely lack direct experience, use the **"Here's how I would approach it"** framework.

### Template Structure

```
1. Acknowledge honestly: "I haven't encountered this exact situation in production."
2. Show structured thinking: "Based on my understanding and related experience..."
3. Outline your approach: Step-by-step plan
4. Reference related experience: "In a similar situation, I..."
5. Show learning intent: "I would also consult runbooks / escalate to / review documentation."
```

### Example: No Production Incident Experience

**Question:** "Tell me about a production incident you handled."

> "I haven't been the primary on-call responder for a Sev1 incident yet, but I've supported incident response as part of my team. In one case, a staging environment failure mirrored a potential production issue — I followed our runbook to collect diagnostics using kubectl, identified a misconfigured secret, and documented the fix. For a production incident, I would follow our established process: assess user impact and severity, mitigate first (rollback or scale), communicate via the incident channel, investigate root cause after stabilization, and contribute to the blameless postmortem. I've studied incident response frameworks and practiced troubleshooting in my home lab by intentionally breaking services and restoring them."

### Example: No SLO Experience

**Question:** "Tell me about defining SLOs."

> "I haven't formally defined SLOs in a production environment, but I understand the framework from study and practice. I would start by identifying the user journey, choose user-centered SLIs such as successful-request ratio and latency, set targets from stakeholder input and historical data, and implement error-budget monitoring. I would use the four golden signals as diagnostic coverage, not assume that every golden signal is itself an SLI. In a lab, I would validate the full setup before applying it to production with the team."

---

## Strengths & Weaknesses

### Strengths — Choose One, Explain Impact

| Strength | Impact Statement | Best For |
|----------|------------------|----------|
| **Systematic troubleshooting** | "I follow a structured approach — user impact first, then work through the stack methodically. This reduced my average diagnosis time by 40%." | SRE, on-call roles |
| **Automation mindset** | "I constantly look for repetitive work to automate. I've eliminated 15+ hours/month of toil through scripting." | DevOps, SRE |
| **Calm under pressure** | "During incidents, I focus on mitigation and communication rather than panic. Teams rely on me to coordinate effectively." | On-call, incident lead |
| **Documentation culture** | "I treat runbooks and postmortems as deliverables, not afterthoughts. This helped onboard 3 new team members faster." | Any technical role |
| **Cross-team collaboration** | "I bridge the gap between development and operations by speaking both languages and focusing on shared goals." | SRE, platform |

**Answer template:**
> "My greatest strength is [strength]. For example, [brief STAR]. This is particularly valuable in an SRE role because [connection to reliability/on-call/automation]."

### Weaknesses — Real, With Improvement Plan

| Weakness | Improvement Plan | Why This Works |
|----------|------------------|----------------|
| **Over-engineering solutions** | "I now time-box design to 30 min and start with the simplest solution. I ask 'What's the minimum viable fix?'" | Shows self-awareness + growth |
| **Diving too deep during incidents** | "I learned to mitigate first, investigate later. I set 15-min time-boxes for root cause during active incidents." | Shows incident maturity |
| **Saying yes to too many tasks** | "I now prioritize using impact/effort matrix and communicate trade-offs to stakeholders early." | Shows prioritization growth |
| **Public speaking / presentations** | "I've volunteered to present in team meetings and completed a communication workshop. I'm improving steadily." | Honest, shows effort |
| **Limited experience with [specific tool]** | "I haven't used Istio in production, but I've studied the concepts and completed a lab. I'm a fast learner — I ramped up on ArgoCD in 5 weeks." | Honest gap + mitigation |

**Answer template:**
> "An area I'm actively improving is [weakness]. In the past, [brief example of how it manifested]. I've been working on this by [specific actions]. For instance, [evidence of improvement]."

---

## How to Present Your CV

### CV Walkthrough Structure (2-3 minutes)

```
1. Current/most recent role (60 seconds)
   → Company, team, scope, key achievement

2. Previous relevant role (30 seconds)
   → Focus on transferable skills

3. Technical summary (30 seconds)
   → Tools matching job description

4. Why this role (15 seconds)
   → Connect to HSBC SRE position
```

### For Each Role, Prepare

| Question | Your Answer |
|----------|-------------|
| What was the company/team context? | Size, industry, your team's mission |
| What were your specific responsibilities? | Daily work, on-call, projects |
| What tools did you use? | Match to HSBC JD keywords |
| What was your biggest achievement? | STAR with metrics |
| What did you learn? | Skills gained, growth |

### Mapping CV to HSBC Job Description

| JD Requirement | Your CV Evidence |
|----------------|------------------|
| Docker, Kubernetes | "Managed K8s cluster with 20+ services..." |
| AWS/Azure/GCP | "Deployed infrastructure on AWS using..." |
| Prometheus, Grafana | "Built monitoring stack with..." |
| Jenkins, GitLab CI | "Maintained CI/CD pipelines in..." |
| ArgoCD, FluxCD | "Implemented GitOps with..." |
| Python, Bash, Go | "Automated X using Python script..." |
| On-call | "Participated in weekly on-call rotation..." |
| SLOs, incident response | "Defined SLOs for..." / "Led incident response..." |

---

## Reflection Prompts

Answer these privately before the interview. Your answers become STAR stories.

### Incident & Reliability

1. What was the worst production incident you were involved in? What happened, what did you do, what changed afterward?
2. Describe a time you prevented an incident before it happened.
3. When did you last say "no" to a risky deployment? What happened?
4. What's the most valuable postmortem you contributed to?

### Automation & Toil

5. What manual task did you automate last? How much time did it save?
6. What's the most repetitive thing you still do manually? Why haven't you automated it?
7. Describe a tool or script you built that others adopted.

### Collaboration

8. Tell me about a time you helped a developer understand a production issue.
9. Describe a cross-team project you contributed to.
10. When did you disagree with a technical decision? How did you handle it?

### Learning & Growth

11. What technology did you learn in the last 6 months? How?
12. What's the hardest technical problem you've solved?
13. What feedback have you received that changed how you work?

### HSBC-Specific

14. Why does working at a global bank appeal to you?
15. How do you handle working with teams in different time zones?
16. What does "trust" mean in the context of banking infrastructure?

---

## HSBC-Aligned Behavioral Expectations

### Map Your Stories to HSBC Values

| HSBC Value | Story to Prepare |
|------------|------------------|
| **We get it done** | Incident mitigation, automation delivery, meeting deadlines |
| **We value difference** | Learning from diverse team perspectives, inclusive postmortems |
| **We take responsibility** | Owning on-call, proactive monitoring, speaking up about risks |
| **We succeed together** | Cross-team collaboration, platform improvements benefiting multiple teams |

### Three Answer Types (Per HSBC Guidelines)

Per your interview preparation document, prepare:

1. **Commercial example** — From paid employment (Examples 1-8 above)
2. **Non-commercial example** — Home lab, open source, certifications (Examples 9-11)
3. **Hypothetical approach** — When you lack direct experience (Templates above)

### Final Behavioral Tips

- **Smile and maintain eye contact** (HSBC preparation guideline)
- **Be enthusiastic** — HSBC wants "enthusiastic and collaborative" candidates
- **Use "I"** for your actions — interviewers want to know YOUR contribution
- **Quantify results** whenever possible
- **Connect to banking context** — reliability affects customer trust
- **Show learning mindset** — postmortems, certifications, home lab

---

*Next: [09_questions_to_ask_interviewer.md](./09_questions_to_ask_interviewer.md)*
