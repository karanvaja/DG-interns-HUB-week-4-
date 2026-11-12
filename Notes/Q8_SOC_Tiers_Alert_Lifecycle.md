# Q8 — SOC Tiers, Alert Lifecycle, and Triage Questions

## Overview

A Security Operations Center (SOC) is organized into tiers based on task
complexity. Alerts flow through a lifecycle from generation to closure. Every
triage note must answer five questions: **who, what, when, where, why**.

---

## SOC Tiers (L1, L2, L3)

| Tier | Role | Responsibilities | Skills |
|---|---|---|---|
| **L1** | Triage Analyst | Monitor alert queue; validate alerts; escalate true positives; close false positives; document findings | Alert triage, basic investigation, SIEM querying |
| **L2** | Incident Responder | Investigate escalated alerts; correlate across data sources; determine scope and impact; contain threats | Deep investigation, forensics, containment, malware analysis |
| **L3** | Threat Hunter / Senior IR | Proactive hunting; detection engineering; complex incident handling; tuning rules; forensic analysis | Threat hunting, detection engineering, reverse engineering, IR leadership |

### L1 — Triage Analyst

- **What they do:** Monitor the alert queue, validate alerts using runbooks,
  escalate true positives, close false positives with documentation.
- **What they don't do:** Deep investigation, containment, or rule writing.
- **Key skill:** Fast, accurate triage using the five questions.
- **Metrics:** Alert closure time, false positive rate, escalation accuracy.

### L2 — Incident Responder

- **What they do:** Investigate escalated alerts, correlate across data sources,
  determine scope and impact, contain threats, coordinate remediation.
- **What they don't do:** Proactive hunting or detection engineering.
- **Key skill:** Deep investigation and containment.
- **Metrics:** Time to contain, scope accuracy, remediation success.

### L3 — Threat Hunter / Senior IR

- **What they do:** Proactive hunting for threats not caught by rules, detection
  engineering (writing/tuning rules), complex incident handling, forensic analysis,
  mentoring L1/L2.
- **What they don't do:** Routine triage (that's L1/L2).
- **Key skill:** Hypothesis-driven hunting and detection engineering.
- **Metrics:** New detections deployed, hunting findings, MTTR reduction.

### Tier Escalation Flow

Alert → L1 (triage) → if true positive → L2 (investigate/contain)
→ if complex/unknown → L3 (hunt/engineer)
→ if false positive → close + feedback for tuning


---

## Alert Lifecycle

| Stage | What Happens | Who |
|---|---|---|
| **1. Alert Generated** | Rule or correlation fires | SIEM |
| **2. Queued** | Alert enters analyst workflow | SIEM |
| **3. Triage** | Validate: is this real? What is affected? | L1 |
| **4. Escalation** | Investigate, contain, remediate | L2 (or L3) |
| **5. Closure** | Document resolution and tuning recommendations | L1/L2 |
| **6. Feedback** | Update rules, allowlists, or thresholds | L3 / Detection Engineering |

### Lifecycle Diagram

| Stage | Name | What Happens | Who |
|---|---|---|---|
| **1** | **Generated** | Rule or correlation fires | SIEM |
| **2** | **Queued** | Alert enters analyst workflow | SIEM |
| **3** | **Triage** | L1 validates using who/what/when/where/why | L1 |
| **4a** | **False Positive → Close** | L1 closes alert and documents why it was benign | L1 |
| **4b** | **True Positive → Escalate** | L1 escalates to L2 for investigation/containment | L1 → L2 |
| **5** | **Closure** | L2 documents resolution after investigation | L2 |
| **6a** | **Feedback (from FP)** | Tune rule/allowlist to prevent recurrence | L3 / Detection Eng |
| **6b** | **Feedback (from TP)** | Update detections based on attack findings | L3 / Detection Eng |

---

## Five Triage Questions (Who, What, When, Where, Why)

Every triage note must answer these five questions. No exceptions.

| Question | What to Document | Example |
|---|---|---|
| **Who** | Which user account(s) or source IP(s) are involved? Is the account privileged? | `svc_backup` from 10.0.5.22 (service account, privileged) |
| **What** | What activity triggered the alert? | 12 failed RDP logons (Event 4625) |
| **When** | Timestamp of the activity; business hours or off-hours? | 02:14–02:18 UTC, outside backup window |
| **Where** | Which host(s), network segment, or application? Is the asset critical? | `FIN-SQL-01` (critical database server) |
| **Why** | Why did the rule fire? What is the detection logic? Expected or anomalous? | Rule fires on >5 failures in 5 min; no scheduled task explains this |

### Why These Five Questions Matter

| Question | Why It Matters |
|---|---|
| **Who** | Determines if a privileged account is at risk; drives escalation priority |
| **What** | Identifies the attack type; guides investigation steps |
| **When** | Off-hours activity is more suspicious; helps correlate with other events |
| **Where** | Asset criticality determines urgency; critical servers escalate faster |
| **Why** | Confirms the rule fired correctly; distinguishes true positives from false positives |

### Example Triage Note (Complete)

> **Who:** `svc_backup` from 10.0.5.22 (service account, privileged).
> **What:** 12 failed RDP logons (Event 4625).
> **When:** 02:14–02:18 UTC, outside backup window.
> **Where:** `FIN-SQL-01` (critical database server).
> **Why:** Rule fires on >5 failures in 5 min; no scheduled task explains this.
> **Action:** Escalated to L2 — possible brute force.

### Triage Note Template

WHO: [user account / source IP / privileged?]
WHAT: [activity that triggered the alert]
WHEN: [timestamp / business hours or off-hours?]
WHERE: [host / network segment / asset criticality]
WHY: [rule logic / expected or anomalous?]
ACTION: [close / escalate to L2 / escalate to L3]


---

## Tier Responsibilities Matrix

| Task | L1 | L2 | L3 |
|---|---|---|---|
| Monitor alert queue | ✅ | ⚠️ | ❌ |
| Validate alerts (triage) | ✅ | ⚠️ | ❌ |
| Document triage notes | ✅ | ✅ | ✅ |
| Escalate true positives | ✅ | ❌ | ❌ |
| Deep investigation | ❌ | ✅ | ✅ |
| Contain threats | ❌ | ✅ | ✅ |
| Proactive threat hunting | ❌ | ⚠️ | ✅ |
| Detection engineering | ❌ | ⚠️ | ✅ |
| Rule tuning | ❌ | ⚠️ | ✅ |
| Forensic analysis | ❌ | ⚠️ | ✅ |
| Mentor L1/L2 | ❌ | ⚠️ | ✅ |

**Legend:** ✅ = primary responsibility | ⚠️ = secondary/supporting | ❌ = not responsible

---

## Alert Lifecycle Metrics

| Metric | Definition | Target |
|---|---|---|
| **MTTD** (Mean Time to Detect) | Time from event to alert | < 5 minutes |
| **MTTA** (Mean Time to Acknowledge) | Time from alert to L1 acknowledgment | < 15 minutes |
| **MTTR** (Mean Time to Respond) | Time from alert to containment | < 1 hour |
| **False Positive Rate** | FP / (FP + TP) | < 20% |
| **Escalation Accuracy** | Correct escalations / total escalations | > 90% |

---

## Key Takeaways

| Concept | Summary |
|---|---|
| **L1** | Triage and escalate; first line of defense |
| **L2** | Investigate and contain; deep incident response |
| **L3** | Hunt and engineer; proactive and strategic |
| **Alert lifecycle** | Generated → Queued → Triage → Escalate → Close → Feedback |
| **Five questions** | Who, What, When, Where, Why — every triage note must answer all five |

### Golden Rules

1. **L1 triages, L2 investigates, L3 hunts** — respect the tier boundaries.
2. **Every triage note answers who/what/when/where/why** — no exceptions.
3. **Feedback loop** (step 6) is what improves the SIEM over time.
4. **Tiers escalate by complexity and impact**, not by alert severity alone.
5. **Document everything** — triage notes are the institutional memory of the SOC.
6. **Measure MTTD, MTTA, MTTR** to track SOC performance.