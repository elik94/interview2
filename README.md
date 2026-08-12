# HSBC SRE Interview Preparation

This repository is a preparation library for a mixed technical and behavioral HSBC SRE interview. It does not guarantee an interview outcome.

## If the interview is in three hours

Open **[the self-contained three-hour cram guide](./preparation/00_3_hour_cram_guide.md)** and follow its 180-minute schedule. Speak the answers and drills aloud; do not try to read the full library tonight.

The guide prioritizes:

1. CV- and job-description positioning
2. SRE, Kubernetes, CI/CD, observability, Linux, networking, and automation fundamentals
3. Six structured troubleshooting drills
4. Six CV-grounded STAR prompts
5. Honest bridges for tools not shown on the CV
6. Banking reliability, security, audit, and recovery concerns
7. A scored lightning mock and interview setup

## Evidence and accuracy rule

[`resume.md`](./resume.md) is read-only source material for tailoring. It supports hands-on Azure, AKS, Terraform, Azure DevOps/GitHub Actions, Azure Monitor/Log Analytics, identity/security, incident work, collaboration, and automation claims.

Do not convert generic examples in the preparation library into candidate history. In particular, Prometheus/Grafana, Jenkins/GitLab, Argo CD/Flux/Tekton, Istio, Go, specific incident metrics, formal SLO ownership, and on-call details must be described as conceptual or transferable unless the candidate can independently verify hands-on experience.

## Preparation library

| Guide | Use it for |
|---|---|
| [00 — Three-hour cram guide](./preparation/00_3_hour_cram_guide.md) | Tonight’s complete, high-yield route |
| [01 — SRE core concepts](./preparation/01_sre_core_concepts.md) | SLOs, error budgets, incidents, observability, reliability patterns |
| [02 — Linux and networking](./preparation/02_linux_networking.md) | Commands, network layers, host troubleshooting |
| [03 — Cloud and Kubernetes](./preparation/03_cloud_kubernetes.md) | Containers, Kubernetes operations, cloud patterns |
| [04 — CI/CD and automation](./preparation/04_ci_cd_automation.md) | Pipelines, IaC, deployment safety, scripting |
| [05 — 110 common questions](./preparation/05_common_questions_and_answers.md) | Wider question bank after the priority review |
| [06 — SRE system design](./preparation/06_system_design_sre.md) | Requirements, architecture, failure modes, capacity |
| [07 — HSBC company preparation](./preparation/07_hsbc_company_preparation.md) | Company and values research; recheck time-sensitive facts |
| [08 — Behavioral STAR method](./preparation/08_behavioral_star_method.md) | STAR structure and generic templates—not candidate history |
| [09 — Questions to ask](./preparation/09_questions_to_ask_interviewer.md) | Questions by stage and topic |
| [10 — Final checklist](./preparation/10_final_interview_checklist.md) | Day-before, day-of, and post-interview checks |

Source material:

- [HSBC SRE job description](./job_description_SRE.md) — transcription corrections are marked
- [Candidate resume](./resume.md) — source evidence only; do not edit as part of this preparation pack
- [Original interview-preparation notes](./interview_preparation.md)

## HSBC values

The four current values to use are:

- **We get it done**
- **We value difference**
- **We take responsibility**
- **We succeed together**

Source: [HSBC — Our values](https://www.hsbc.com/who-we-are/our-strategy-and-values/our-values)

Avoid memorizing volatile company figures. Recheck any customer, country, employee, strategy, or organization claim on HSBC’s official site on interview day.

## After this interview

Lower-priority expansion backlog:

- Replace generic STAR templates with personally verified story cards.
- Build small labs for Prometheus/Grafana, Argo CD or Flux, Istio, and Go.
- Add a source/date note to company facts that may change.
- Run a full-length mock and keep only questions that exposed a real gap.
- Record unknown metrics during future work so later examples can be precise without guessing.
