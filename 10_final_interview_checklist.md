# Final Interview Checklist — HSBC SRE Interview

> **Use this checklist** in the days and hours before your interview  
> **Goal:** Arrive confident, prepared, and professional

---

## Table of Contents

1. [Pre-Interview Checklist (1 Week Before)](#pre-interview-checklist-1-week-before)
2. [Technical Warm-Up Checklist](#technical-warm-up-checklist)
3. [Behavioral Warm-Up Checklist](#behavioral-warm-up-checklist)
4. [Environment & Setup Checklist](#environment--setup-checklist)
5. [Mental Preparation Checklist](#mental-preparation-checklist)
6. [Day-Of Timeline](#day-of-timeline)
7. [HSBC-Specific Final Reminders](#hsbc-specific-final-reminders)
8. [Post-Interview Checklist](#post-interview-checklist)

---

## Pre-Interview Checklist (1 Week Before)

### Research & Company Knowledge

```markdown
□ Read HSBC website — About, Values, Strategy pages
  → https://www.hsbc.com/who-we-are/our-strategy-and-values

□ Know HSBC's four values:
  → We get it done
  → We value difference
  → We take responsibility
  → We succeed together

□ Know HSBC's purpose: "Opening up a world of opportunity"
□ Know HSBC's ambition: "To be the most trusted bank globally"

□ Watch 2-3 HSBC YouTube videos — note specific topics to reference
  → https://www.youtube.com/@HSBC

□ Review job description — map each requirement to your experience

□ Review your CV — prepare 2-min walkthrough for each role
```

### Technical Preparation

```markdown
□ Review SRE fundamentals (SLI/SLO/SLA, error budgets, golden signals)
□ Refresh Kubernetes commands and concepts
□ Review CI/CD pipeline patterns (GitLab CI, Jenkins, GitOps)
□ Practice 3 troubleshooting scenarios (Linux, K8s, network)
□ Review monitoring stack (Prometheus, Grafana, ELK, Jaeger)
□ Refresh scripting examples (Python/Bash/Go)
□ Review system design framework
```

### Behavioral Preparation

```markdown
□ Prepare 5+ STAR stories (3 commercial, 2 non-commercial)
□ Prepare strength answer (one, with impact)
□ Prepare weakness answer (one, with improvement plan)
□ Prepare "Why HSBC?" answer (with specific references)
□ Prepare "Why SRE?" answer
□ Prepare 8-10 questions to ask interviewer
□ Practice answers out loud (2-3 min each)
```

### Logistics

```markdown
□ Confirm interview date, time, timezone
□ Confirm format (video call, platform — Teams/Zoom?)
□ Confirm interviewer names and roles (if provided)
□ Test video call platform
□ Plan quiet location with good internet
□ Lay out professional attire
```

---

## Technical Warm-Up Checklist

### Day Before — Quick Review (30 minutes)

```markdown
□ SLI vs SLO vs SLA — can you explain in 30 seconds?
□ Error budget — what happens when exhausted?
□ Four golden signals — name them
□ Kubernetes: Pod vs Deployment vs StatefulSet
□ Liveness vs readiness probes
□ GitOps workflow — explain ArgoCD flow
□ CI/CD pipeline stages — build, test, scan, deploy
□ Incident response phases — detect, triage, mitigate, resolve, learn
□ CAP theorem — CP vs AP with banking example
□ Load balancer L4 vs L7
```

### Command Refresh

```bash
# Kubernetes
kubectl get pods -n <ns> -o wide
kubectl describe pod <pod> -n <ns>
kubectl logs <pod> -n <ns> --previous
kubectl rollout undo deployment/<name> -n <ns>

# Linux
top / htop
ss -tlnp | grep :PORT
journalctl -u SERVICE -f
df -h && df -i
strace -p PID

# Debugging chain
curl -v URL → ss → logs → describe → events
```

### Troubleshooting Drill (Practice One)

Pick one scenario and walk through it out loud:

```markdown
□ Scenario A: API returning 502 errors
□ Scenario B: Pod in CrashLoopBackOff
□ Scenario C: High latency during peak traffic
□ Scenario D: Node NotReady in Kubernetes
□ Scenario E: Disk full on production server
```

---

## Behavioral Warm-Up Checklist

### Day Before — Practice Out Loud

```markdown
□ "Tell me about yourself" (2 min max)
□ "Why HSBC?" (1 min — include specific reference)
□ "Why SRE?" (1 min)
□ Best incident STAR story (3 min)
□ Best automation STAR story (3 min)
□ Strength + impact (1 min)
□ Weakness + improvement (1 min)
□ "Where do you see yourself in 3-5 years?" (1 min)
```

### STAR Story Readiness

| Story | Ready? | Key Metric |
|-------|--------|------------|
| Production incident | □ | MTTR: ___ min |
| Automation / toil reduction | □ | Time saved: ___ hrs |
| Cross-team collaboration | □ | Outcome: ___ |
| Implementing SLOs/monitoring | □ | Alert reduction: ___% |
| Learning new technology | □ | Time to proficiency: ___ |
| Non-commercial project | □ | Skills gained: ___ |

### Answer Quality Check

For each story, verify:
```markdown
□ Used "I" not "we" for actions
□ Included specific tools/technologies
□ Quantified the result
□ Under 3 minutes when spoken
□ Connected to SRE/banking relevance
```

---

## Environment & Setup Checklist

### HSBC Preparation Guideline: Camera, Lighting, Background

```markdown
□ Camera at eye level (not looking up/down)
□ Face well-lit (light source in front, not behind)
□ Background clean and professional (or blurred)
□ No distractions (notifications off, phone silent)
□ Stable internet connection ( ethernet if possible)
□ Backup plan if internet fails (phone hotspot)
```

### Technology Test (Day Before)

```markdown
□ Test webcam — clear image, eye level
□ Test microphone — clear audio, no echo
□ Test speakers/headphones
□ Test video platform (Teams/Zoom/WebEx)
□ Close unnecessary applications
□ Browser updated, platform link works
□ Phone charged (backup communication)
□ Water nearby
□ Notepad and pen for notes
```

### Physical Environment

```markdown
□ Quiet room — inform household/roommates
□ "Do not disturb" sign on door
□ Desk clear of clutter
□ Professional attire (at least top half — full if standing)
□ Comfortable temperature
```

---

## Mental Preparation Checklist

```markdown
□ Get 7-8 hours of sleep the night before
□ Eat a good meal 1-2 hours before (not too heavy)
□ Arrive / log in 15 MINUTES EARLY (HSBC guideline)
□ Deep breaths before joining — calm and confident
□ Remember: interview is mutual evaluation
□ Smile — HSBC guideline says it really helps
□ Maintain eye contact (look at camera, not screen)
□ It's OK to pause and think before answering
□ It's OK to say "I don't know, but here's how I'd approach it"
□ Have water available
```

### Mindset Reminders

| Thought | Reframe |
|---------|---------|
| "They'll catch what I don't know" | "I've prepared thoroughly and can think through problems" |
| "I need to be perfect" | "They want to see how I think and communicate" |
| "Other candidates are better" | "I bring unique experience and genuine enthusiasm" |
| "What if I blank?" | "I can ask for a moment to think, or ask them to repeat" |

---

## Day-Of Timeline

### 15 Minutes Before (HSBC Guideline)

```
T-15 min  □ Log into video platform
          □ Final camera/mic check
          □ Close all unnecessary apps and tabs
          □ Have water, notepad, pen ready
          □ Open job description (reference, not visible on screen)
          □ Deep breath, smile

T-5 min   □ Review "Why HSBC?" one-liner
          □ Review top 3 questions to ask
          □ Positive self-talk

T-0       □ Join call
          □ Greet with smile and enthusiasm
          □ "Thank you for taking the time to meet with me today"
```

### During Interview

```
□ Listen fully before answering
□ Structure technical answers (approach first, then details)
□ Use STAR for behavioral questions
□ Reference HSBC values/content naturally (don't force)
□ Take brief notes on questions they ask
□ Ask your prepared questions when invited
□ Express enthusiasm for the role
□ Thank them at the end
```

---

## HSBC-Specific Final Reminders

### From HSBC Interview Preparation Guidelines

| Guideline | Action |
|-----------|--------|
| **Know HSBC** | Business, values, products, markets |
| **YouTube content** | Reference 1-2 videos naturally |
| **Technical prep** | Theoretical + practical/case-based scenarios |
| **CV ready** | Contribution, outcomes, tools for each role |
| **STAR method** | Commercial + non-commercial + hypothetical |
| **Strengths & weaknesses** | One each, prepared |
| **Ask questions** | Always say yes — prepare 5+ |
| **Be ready 15 min early** | Log in, test, breathe |
| **Camera, lighting, background** | Professional setup |
| **Eye contact and smile** | Look at camera, be warm |

### From Job Description — Key Topics to Expect

```markdown
□ SRE best practices (SLOs, SLIs, incident response)
□ Docker and Kubernetes (deploy, debug, manage)
□ CI/CD pipelines (Jenkins, GitLab CI)
□ Automation (Python, Bash, Go)
□ Monitoring (Prometheus, Grafana, ELK, Jaeger, Zipkin, Datadog)
□ GitOps (ArgoCD, FluxCD, Tekton)
□ Cloud platforms (AWS, Azure, or GCP)
□ On-call and incident resolution
□ Documentation and post-incident reviews
□ Cross-team collaboration (dev, QA, ops)
```

### HSBC Values — Quick Reference

| Value | Interview Signal |
|-------|-----------------|
| **We get it done** | Deliver results, meet commitments, move at pace |
| **We value difference** | Listen, diverse perspectives, inclusive |
| **We take responsibility** | Own incidents, high standards, speak up |
| **We succeed together** | Collaborate, break silos, support teammates |

---

## Post-Interview Checklist

```markdown
□ Send thank-you email within 24 hours (if appropriate)
□ Note questions you were asked (for future prep)
□ Note questions you struggled with (study those topics)
□ Note interviewer names and roles
□ Jot down anything you forgot to mention (for follow-up)
□ Record your self-assessment: what went well, what to improve
□ Wait for feedback timeline they provided
□ Continue preparing other topics while waiting
```

### Thank-You Email Template

```
Subject: Thank you — SRE Interview

Dear [Interviewer Name],

Thank you for taking the time to speak with me today about the
Site Reliability Engineer role. I enjoyed learning about [specific
topic discussed — e.g., the team's GitOps workflow / current
reliability initiatives].

Our conversation reinforced my enthusiasm for the role, particularly
[specific aspect — e.g., the opportunity to define SLOs for critical
banking services / the team's automation culture].

Please let me know if you need any additional information. I look
forward to hearing from you.

Best regards,
[Your Name]
```

---

## One-Page Quick Checklist (Print or Pin)

```
┌─────────────────────────────────────────────────────────┐
│           HSBC SRE INTERVIEW — FINAL CHECKLIST           │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  RESEARCH                                                │
│  □ HSBC values (4)    □ YouTube videos (2-3)           │
│  □ Job description    □ CV walkthrough ready             │
│                                                          │
│  STORIES                                                 │
│  □ 3 STAR commercial  □ 2 STAR non-commercial            │
│  □ Strength           □ Weakness                         │
│  □ Why HSBC?           □ Why SRE?                         │
│                                                          │
│  TECHNICAL                                               │
│  □ SLO/SLI/SLA       □ K8s debugging commands           │
│  □ CI/CD/GitOps       □ 1 troubleshooting drill         │
│                                                          │
│  QUESTIONS TO ASK (pick 3-5)                             │
│  □ First months expect. □ Team focus/challenges           │
│  □ On-call rotation    □ SLOs defined?                    │
│  □ Feedback timeline                                     │
│                                                          │
│  SETUP                                                   │
│  □ Camera eye level   □ Good lighting                    │
│  □ Quiet room         □ Platform tested                  │
│  □ Professional attire □ Join 15 min early                │
│                                                          │
│  MINDSET                                                 │
│  □ Smile              □ Eye contact (camera)             │
│  □ Pause & think OK   □ Interview is mutual              │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

---

**Good luck with your HSBC SRE interview!**

You've prepared thoroughly across all 10 guides in this package. Trust your preparation, be yourself, and demonstrate the reliability mindset that makes a great SRE.

---

*Package complete. Return to [01_sre_core_concepts.md](./01_sre_core_concepts.md) for any topic review.*
