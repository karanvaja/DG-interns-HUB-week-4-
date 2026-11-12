# Q5 — True Positive, False Positive, True Negative, False Negative, Alert Fatigue, and Tuning

## Overview

Every detection rule produces one of four outcomes based on two dimensions:
**did the alert fire?** and **was the activity actually malicious?** These four
outcomes form the **confusion matrix** of detection quality. Understanding them is
essential for measuring SIEM effectiveness and tuning rules.

---

## The Four Outcomes (Confusion Matrix)

| | **Activity is malicious** | **Activity is benign** |
|---|---|---|
| **Alert fired** | True Positive (TP) | False Positive (FP) |
| **No alert fired** | False Negative (FN) | True Negative (TN) |

---

## Definitions with Examples

### 1. True Positive (TP)

| Aspect | Detail |
|---|---|
| **Definition** | Alert fires and the activity is genuinely malicious |
| **Example** | SSH brute force alert fires for 50 failed logins from a foreign IP targeting `root` — and it really is an attacker |
| **Handling** | Escalate, investigate, contain, remediate |
| **Goal** | Maximize these — they are the reason the SIEM exists |

### 2. False Positive (FP)

| Aspect | Detail |
|---|---|
| **Definition** | Alert fires but the activity is benign |
| **Example** | Vulnerability scanner performs authenticated checks causing failed login alerts — the scanner is authorized, not an attacker |
| **Handling** | Tune the rule, allowlist the scanner, or adjust threshold |
| **Impact** | Wastes analyst time; high FP rates cause alert fatigue |

### 3. True Negative (TN)

| Aspect | Detail |
|---|---|
| **Definition** | No alert fires and no malicious activity occurred |
| **Example** | Normal user logs in successfully during business hours; no alert fires — correct behavior |
| **Handling** | No action needed — this is the desired state for benign activity |
| **Note** | TNs are invisible in daily operations but essential for measuring specificity |

### 4. False Negative (FN)

| Aspect | Detail |
|---|---|
| **Definition** | No alert fires but malicious activity occurred |
| **Example** | Attacker uses a legitimate credential and logs in silently; no rule detects the anomalous login |
| **Handling** | Improve detection coverage — add rules, baselines, or behavioral analytics |
| **Impact** | Most dangerous outcome — a real attack goes unnoticed |

---

## Summary Table

| Term | Alert Fired? | Malicious? | Example | Action |
|---|---|---|---|---|
| **True Positive (TP)** | Yes | Yes | SSH brute force from foreign IP | Escalate and investigate |
| **False Positive (FP)** | Yes | No | Vulnerability scanner causes failed logins | Tune rule / allowlist |
| **True Negative (TN)** | No | No | Normal successful login | No action |
| **False Negative (FN)** | No | Yes | Attacker uses valid creds silently | Improve detection coverage |

---

## Alert Fatigue

### Definition
Alert fatigue occurs when analysts are overwhelmed by high volumes of low-fidelity
alerts, leading to slower response times, rubber-stamping closures, and missed true
threats.

### Causes
- Rules with high false positive rates (e.g., 92% FP rate generating 847 alerts/day)
- Poorly tuned thresholds that fire on normal business activity
- Duplicate alerts for the same event from multiple rules
- Lack of context (no asset criticality, no user risk score)

### Consequences
- Analysts start ignoring alerts
- Real attacks buried in noise go unnoticed
- Morale drops; burnout increases
- SOC effectiveness degrades over time

### Example Scenario
> A rule fires on every failed login (level 5). The environment has 500 failed
> logins per day from normal user typos. The analyst sees 500 alerts/day and starts
> closing them without investigation. On day 30, a real brute force attack generates
> 50 alerts — all closed as "known noise." The attacker succeeds.

---

## Tuning

### Definition
Tuning is the process of adjusting detection logic to reduce false positives and
false negatives while maintaining coverage for real threats.

### Tuning Methods

| Method | Description | Example |
|---|---|---|
| **Allowlisting** | Suppress alerts for known benign sources | Add vulnerability scanner IPs to allowlist |
| **Raising thresholds** | Increase the count/timeframe required to fire | Change 5 failures → 10 in 5 minutes |
| **Adding context** | Use asset criticality or user risk scores to prioritize | Critical DB server alerts rank higher than dev laptop |
| **Disabling rules** | Remove rules that produce noise without security value | Disable a rule that fires on every Windows update |
| **Risk-based alerting** | Multiple low-level events contribute to a risk score instead of each firing separately | 3 low events + 2 medium = one high-risk alert |
| **Adding exclusions** | Exclude specific users, hosts, or patterns | Exclude `svc_backup` from failed login alerts during backup window |
| **Baseline adjustment** | Set thresholds based on observed normal behavior | Normal is 2 failures/hour → threshold = 10 in 5 minutes |

### Tuning Workflow

1. **Identify noisy rules** — track FP rate per rule over time
2. **Analyze the noise** — what is causing false positives? (scanner, maintenance, typos)
3. **Apply the right fix** — allowlist, threshold, exclusion, or disable
4. **Test the change** — verify FPs reduced without introducing FNs
5. **Monitor** — re-evaluate periodically; environments change

### Goal
Maximize the **signal-to-noise ratio** — true positives per false positive — while
maintaining coverage for real threats.

---

## Measuring Detection Quality

| Metric | Formula | Meaning |
|---|---|---|
| **Precision** | TP / (TP + FP) | Of all alerts fired, how many were real? |
| **Recall (Sensitivity)** | TP / (TP + FN) | Of all real attacks, how many did we catch? |
| **False Positive Rate** | FP / (FP + TN) | How often do we alert on benign activity? |
| **False Negative Rate** | FN / (FN + TP) | How often do we miss real attacks? |

**Trade-off:** Raising thresholds reduces FPs but may increase FNs. Lowering
thresholds catches more attacks but increases FPs. Tuning finds the balance.

---

## Key Takeaways

- **TP/FP/TN/FN** form the confusion matrix of detection quality.
- **TP** = correct alert; **FP** = noise; **TN** = correct silence; **FN** = missed attack.
- **Alert fatigue** is a people problem caused by a data problem (noisy rules).
- **Tuning** is continuous, driven by triage feedback.
- Never allowlist without verifying the activity is truly and permanently benign.
- Review allowlists and thresholds periodically — environments change.
- Measure **precision** and **recall** to track detection quality over time.