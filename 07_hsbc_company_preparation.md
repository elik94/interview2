# HSBC Company Preparation — SRE Interview Guide

> **Purpose:** Align your interview answers with HSBC's business, values, culture, and technical expectations  
> **Sources:** HSBC website, YouTube channel, job description, interview preparation guidelines
>
> **Evidence note:** Generic examples in this guide are prompts, not candidate history. Do not repeat tools, incidents, domains, or metrics unless they are independently true and defensible from your experience.

---

## Table of Contents

1. [HSBC Business Overview](#hsbc-business-overview)
2. [Key Values and Principles](#key-values-and-principles)
3. [Culture and Working Style](#culture-and-working-style)
4. [How SRE Fits HSBC Infrastructure](#how-sre-fits-hsbc-infrastructure)
5. [Aligning Your Answers with HSBC](#aligning-your-answers-with-hsbc)
6. [HSBC YouTube Content Strategy](#hsbc-youtube-content-strategy)
7. [HSBC-Specific Behavioral Questions](#hsbc-specific-behavioral-questions)
8. [HSBC-Specific Technical Expectations](#hsbc-specific-technical-expectations)
9. [Research Checklist](#research-checklist)

---

## HSBC Business Overview

### Who Is HSBC?

HSBC is a global banking and financial services organisation. Its current stated purpose is **“Opening up a world of opportunity”**, and its ambition is to be **“the most trusted bank globally, putting customers at the heart of everything we do.”**

Current sources:

- [Who we are: purpose and strategy](https://www.hsbc.com/who-we-are)
- [Our values](https://www.hsbc.com/who-we-are/our-strategy-and-values/our-values)
- [Our markets](https://www.hsbc.com/who-we-are/our-markets)

HSBC's organisation, market footprint, and headline figures can change. Recheck the official pages on interview day rather than memorising customer, country, employee, or business-line counts from this guide.

### Why Technology Matters in This Role

- **Digital services:** Banking channels must be available, secure, and correct
- **Regulatory technology:** Compliance systems, audit trails, reporting
- **Global infrastructure:** Services cross regions, time zones, and failure domains
- **Security:** Financial data protection, fraud detection, cyber resilience
- **Modern platforms:** The job description explicitly calls for cloud, Kubernetes, CI/CD, observability, and automation

**Your SRE role sits at the intersection of all these priorities.**

---

## Key Values and Principles

### HSBC Values (Reference in Interviews)

HSBC's four core values guide strategic decisions and day-to-day interactions. Source: [hsbc.com/who-we-are/our-strategy-and-values/our-values](https://www.hsbc.com/who-we-are/our-strategy-and-values/our-values)

| Value | What It Means | How SRE Demonstrates It |
|-------|---------------|-------------------------|
| **We get it done** | Move at pace, make things happen, keep our word | Deliver automation that reduces MTTR; meet SLO commitments; follow through on postmortem action items |
| **We value difference** | Seek diverse perspectives, listen, remove barriers | Blameless postmortems; inclusive incident reviews; learn from different team viewpoints |
| **We take responsibility** | High standards, accountability, speak up when something isn't right | Own on-call incidents; proactive monitoring; escalate risks before they become outages |
| **We succeed together** | Collaborate across boundaries, break down silos | Cross-team work with dev, QA, security; shared reliability goals; platform thinking |

**Purpose:** Opening up a world of opportunity  
**Ambition:** To be the most trusted bank globally, putting customers at the heart of everything we do  

### HSBC Strategic Priorities (Align Your Narrative)

When answering "Why HSBC?" or "Why this role?", connect to:

1. **Global footprint** — "I'm interested in the reliability challenges of services that cross markets, regions, and time zones."
2. **Technology investment** — "HSBC's stated investment in innovative technology aligns with my Azure, AKS, infrastructure-as-code, and CI/CD experience."
3. **Impact** — "Reliability in banking directly affects people's financial lives — that responsibility motivates me."
4. **Learning** — "The complexity of regulated, global financial infrastructure offers continuous growth."

---

## Culture and Working Style

### What HSBC Looks for in Candidates

Based on the job description and preparation guidelines:

| Quality | Evidence in Interview |
|---------|----------------------|
| **Enthusiastic & collaborative** | Examples of cross-team work, positive tone |
| **Passionate about software + operations** | SRE projects, automation, coding examples |
| **Strong troubleshooting** | Incident stories with systematic approach |
| **Willingness to learn** | Recent technology adoption, certifications |
| **Documentation culture** | Runbooks, postmortems, knowledge sharing |
| **On-call readiness** | Calm under pressure, escalation discipline |

### Working Environment Expectations

- **Global teams:** You may collaborate across time zones (APAC, EMEA, Americas)
- **Regulated environment:** Change management, approval processes, audit requirements
- **Hybrid working:** Many HSBC technology roles offer flexible/hybrid arrangements
- **Enterprise scale:** Processes exist for good reason — show respect while suggesting improvements

---

## How SRE Fits HSBC Infrastructure

### SRE Role in HSBC Context

```
┌─────────────── HSBC Technology Stack ───────────────────────┐
│                                                              │
│  Business Applications (banking, trading, mobile)           │
│       │                                                      │
│  Platform Layer (Kubernetes, service mesh, GitOps)          │
│       │                                                      │
│  ┌────▼─────────────────────────────────────────────────┐  │
│  │              SRE TEAM (YOUR ROLE)                      │  │
│  │  • Production stability & monitoring                    │  │
│  │  • SLO/SLI definition & error budgets                  │  │
│  │  • Incident response & on-call                         │  │
│  │  • CI/CD pipeline support                              │  │
│  │  • Automation & toil reduction                         │  │
│  │  • Capacity planning & scaling                         │  │
│  └────────────────────────────────────────────────────────┘  │
│       │                                                      │
│  Cloud Infrastructure (AWS / Azure / GCP)                   │
│       │                                                      │
│  Physical / Network / Security                              │
└──────────────────────────────────────────────────────────────┘
```

### Day-to-Day Responsibilities (From Job Description)

| Responsibility | Evidence-safe talking point |
|----------------|-----------------------------|
| Maintain production systems | Discuss production SaaS, Azure infrastructure, monitoring, and AKS responsibilities; add scale or service levels only if verified. |
| Implement SRE best practices | Explain SLI/SLO/error-budget concepts, while being clear that the CV does not claim formal production SLO ownership. |
| Docker/Kubernetes management | Use hands-on AKS, namespace isolation, RBAC, network policies, Helm/manifests, and multi-environment delivery. |
| CI/CD pipeline development | Use Azure DevOps YAML and GitHub Actions experience; bridge honestly to Jenkins/GitLab principles. |
| Automation (Python/Bash/Go) | Use Terraform, Azure automation, PowerShell/Bash, and Python-capable scripting. The CV's roughly 35% metric applies only to manual data-process workload. |
| Cross-team collaboration | Use developer pipeline troubleshooting, infrastructure reviews, customer guidance, training, and onboarding. |
| On-call & incident response | Use incident management and RCA on networking/identity issues; do not claim an on-call rotation or incident metrics unless true. |
| Documentation | Use knowledge-transfer and training evidence; add runbooks/postmortems only if personally verified. |

---

## Aligning Your Answers with HSBC

### Answer Alignment Framework

For every answer, mentally check:

```
┌─────────────────────────────────────────┐
│  Does my answer demonstrate...          │
│                                         │
│  ✓ Reliability mindset (SLOs, data)    │
│  ✓ Collaboration (cross-team examples)  │
│  ✓ Automation (reduce manual work)      │
│  ✓ Learning culture (postmortems)       │
│  ✓ Global/enterprise awareness          │
│  ✓ Security & compliance awareness      │
│  ✓ Enthusiasm for HSBC specifically     │
└─────────────────────────────────────────┘
```

### Example: Transforming a Generic Answer

**Generic:** "I managed Kubernetes clusters and set up monitoring."

**HSBC-aligned and evidence-safe:** "In my AKS work, I have operated multi-tenant workloads using namespace isolation, RBAC, network policies, and Helm or manifest deployments. My monitoring experience is with Azure Monitor and Log Analytics rather than Prometheus. In a banking context I would connect those platform controls to customer impact, least privilege, auditable change, and measurable service reliability." Add a result only when you can explain how it was measured.

### Keywords to Naturally Include

- Reliability, availability, scalability
- SLOs, SLIs, error budgets
- Blameless postmortems
- Automation, toil reduction
- GitOps, infrastructure as code
- Security, compliance, audit trail
- Cross-functional collaboration
- Global scale, 24/7 operations

---

## HSBC YouTube Content Strategy

### Why Reference YouTube Content

Per HSBC preparation guidelines: referencing specific HSBC content demonstrates **genuine preparation and enthusiasm**.

### How to Use YouTube Content

1. **Before interview:** Visit [HSBC YouTube channel](https://www.youtube.com/@HSBC)
2. **Find 2-3 videos** relevant to your skills or interests:
   - Technology/digital transformation videos
   - Sustainability or community initiatives (if relevant to your values)
   - Global connectivity themes
3. **Weave naturally into answers:**

**Example in "Why HSBC?":**
> "I watched HSBC's video on [specific topic] and was impressed by [specific point]. It connects to my experience with [your skill] because [link]."

**Example in "What do you know about HSBC?":**
> "I noticed HSBC's focus on [current point from the website/video]. In my platform work, I've seen how global infrastructure requires [your verified insight]."

### Do's and Don'ts

| Do | Don't |
|----|-------|
| Reference specific video topics | Say "I watched all your videos" ( vague) |
| Connect video content to your experience | Force irrelevant references |
| Show genuine interest | Recite marketing copy verbatim |
| Mention 1-2 videos naturally | List multiple videos mechanically |

---

## HSBC-Specific Behavioral Questions

### Likely Questions and Approach

| Question | Strategy |
|----------|----------|
| "Why HSBC?" | Global scale + technology transformation + personal values alignment |
| "Why SRE?" | Engineering approach to operations, measurable reliability, impact |
| "Tell me about an incident" | STAR with banking-relevant impact (customer-facing, financial) |
| "How do you handle on-call?" | Runbooks, escalation, calm approach, learning from incidents |
| "Describe cross-team collaboration" | Dev + QA + ops + security example |
| "How do you prioritize reliability vs features?" | Error budget framework, data-driven decisions |
| "Experience with regulated environments?" | Change management, audit trails, approval processes |
| "How do you document your work?" | Runbooks, postmortems, architecture docs examples |
| "Strengths and weaknesses" | One strength with impact; one weakness with improvement plan |
| "Where do you see yourself in 5 years?" | Senior SRE, platform leadership, continuous learning at HSBC |

### HSBC Competency Areas

Prepare STAR examples for:

1. **Technical excellence** — Troubleshooting, automation, system design
2. **Collaboration** — Working across teams, time zones, disciplines
3. **Accountability** — On-call, incident ownership, follow-through
4. **Innovation** — Process improvement, automation, new tool adoption
5. **Risk awareness** — Security, compliance, careful change management

---

## HSBC-Specific Technical Expectations

### From Job Description — Must-Know Topics

| Category | Technologies | Depth Expected |
|----------|-------------|----------------|
| **Containers** | Docker, Kubernetes, Istio, Helm | Deploy, debug, configure |
| **Cloud** | AWS, Azure, or GCP | VPC, compute, storage, IAM |
| **Monitoring** | Prometheus, Grafana, ELK, Jaeger, Zipkin, Datadog | Setup alerts, dashboards, troubleshoot |
| **CI/CD** | Jenkins, GitLab CI | Pipeline design, security gates |
| **GitOps** | ArgoCD, FluxCD, Tekton | Workflow, sync, rollback |
| **Scripting** | Python, Bash, Go | Automation, tooling |
| **SRE practices** | SLOs, SLIs, incident response, on-call | Define, implement, improve |

### Technical Scenario Examples (HSBC Context)

**Scenario 1:** "A payment processing service in Kubernetes is experiencing intermittent failures during peak hours. Walk me through your investigation."

**Scenario 2:** "How would you implement GitOps for deploying microservices to production at a bank?"

**Scenario 3:** "Design monitoring and alerting for a critical banking API with 99.95% SLO."

**Scenario 4:** "How would you automate the onboarding of a new microservice to the platform (CI/CD, monitoring, logging)?"

### Compliance-Aware Technical Answers

Always mention when relevant:

- **Audit trails:** Git history, deployment logs, change records
- **Secrets management:** Vault, not in Git, rotation policies
- **Access control:** RBAC, least privilege, regular reviews
- **Data residency:** EU data stays in EU regions
- **Change management:** Approval gates, maintenance windows, rollback plans

---

## Research Checklist

### Before Your Interview

```markdown
□ Visit hsbc.com — read About, Careers, and Values pages
□ Watch 2-3 HSBC YouTube videos — note specific topics
□ Review job description — map your experience to each bullet
□ Prepare 5+ STAR stories (commercial + non-commercial)
□ Prepare strength and weakness answers
□ Prepare 5+ questions to ask interviewer
□ Review technical topics: K8s, CI/CD, monitoring, Linux
□ Test camera, lighting, microphone
□ Be ready 15 minutes early
```

### HSBC Website Pages to Review

| Page | What to Look For |
|------|------------------|
| **About HSBC** | History, scale, global presence |
| **Careers / Technology** | Tech strategy, team culture |
| **Sustainability** | ESG commitments (if values-aligned) |
| **Investor relations** | Strategic priorities, annual report highlights |
| **Newsroom** | Recent technology announcements |

### Your 30-Second HSBC Elevator Pitch Template

> "HSBC is a global banking and financial services organisation whose purpose is opening up a world of opportunity. I'm drawn to the SRE role because it combines my experience in [verified Kubernetes/automation/reliability evidence] with production systems in a regulated environment. My work on [specific verified achievement] aligns with the team's focus on [job-description requirement]. I'm particularly interested in [current initiative from an official HSBC source]."

---

*Next: [08_behavioral_star_method.md](./08_behavioral_star_method.md)*
