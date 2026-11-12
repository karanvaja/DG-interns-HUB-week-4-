# Q7 — Correlation Rules, Thresholds, Baselines, and Allowlists: When to Use Each

## Overview

Detection engineering is about choosing the **right technique** for the **right
situation**. Four core techniques turn raw events into meaningful alerts:

| Technique | Core Idea | One-Line Summary |
|---|---|---|
| **Correlation Rule** | Link multiple events into one signal | "A then B = attack" |
| **Threshold** | Alert when volume exceeds a limit | "Too many = suspicious" |
| **Baseline** | Alert when behavior deviates from normal | "Different from usual = suspicious" |
| **Allowlist** | Suppress known-benign activity | "This is expected = ignore" |

Each solves a different problem. The wrong choice leads to **alert fatigue**
(too many false positives) or **blind spots** (missed attacks).

---

## 1. Correlation Rules

### What It Is
- Links **multiple events** that are individually low-risk but collectively
  indicate an attack.
- The attack is defined by a **sequence**, not a single event.

### When to Use
- The attack pattern requires **A followed by B**.
- Individual events are normal on their own.
- You want to detect multi-stage attacks.
- You need high-fidelity alerts with fewer false positives.

### Example
**Failed logins followed by a success from the same IP** — each event alone is
normal, but the sequence indicates successful brute force.

**Splunk SPL:**
 index=linux sourcetype=linux_secure
| transaction src_ip maxspan=5m
| search "Failed password" AND "Accepted password"
| table _time, src_ip, user


**Wazuh rule:**
```xml
<rule id="100210" level="12" frequency="5" timeframe="300">
  <if_matched_sid>5716</if_matched_sid>
  <same_source_ip />
  <description>Possible successful brute force: failures followed by success</description>
  <mitre><id>T1110</id></mitre>
</rule>
```


### Common Correlation Patterns

| Pattern | Logic | Why It Works |
|---|---|---|
| **Port scan → exploit** | Scan from IP X, then exploit from IP X | Scan alone is noise; scan + exploit = attack |
| **Malware → C2** | Malware alert on host, then outbound to known-bad IP | Two low-confidence signals = high confidence |
| **Phishing → login** | User clicks phishing link, then login from new location | Links delivery to compromise |
| **USB → exfil** | USB inserted, then large file copy to external | Insider threat chain |


## Pros and Cons Table

| Pros | Cons |
|---|---|
| High fidelity — fewer false positives | Requires multiple events to fire |
| Detects multi-stage attacks | Complex to write and maintain |
| Better context for triage | May miss fast attacks that don't fit the sequence |


## 2. Thresholds
  # What It Is
* Alerts when a count exceeds a defined number within a time window.

* The signal is volume, not sequence.

### When to Use
*A single event type is normal, but high volume is suspicious.

* The attack pattern is "many attempts" rather than "specific sequence."

* You need simple, fast detection.

* You want to catch brute force, scanning, or flooding.

## Example
More than 10 failed logins from one IP in 5 minutes — a few failures are
normal, but 10+ indicates brute force.

# Splunk SPL:

index=wineventlog EventCode=4625
| bucket _time span=5m
| stats count as failures by _time, src_ip
| where failures > 10

### Wazuh rule:
<rule id="100200" level="10" frequency="10" timeframe="300">
  <if_matched_sid>5716</if_matched_sid>
  <same_source_ip />
  <description>High volume of failed logins: 10+ in 5 minutes</description>
  <mitre><id>T1110</id></mitre>
</rule>


## Threshold Examples Table

| Detection | Threshold | Window |
|---|---|---|
| **Brute force** | >10 failed logins per IP | 5 min |
| **DNS tunneling** | >100 queries to one domain | 1 min |
| **Port scan** | >50 connection attempts per IP | 1 min |
| **Data exfiltration** | >500 MB outbound per host | 1 hour |


## Pros and Cons Table

| Pros | Cons |
|---|---|
| Simple to write and understand | Easy to tune wrong (FPs or FNs) |
| Fast to evaluate | Doesn't detect slow-and-low attacks |
| Works well for volume-based attacks | Doesn't understand context (scanner vs attacker) |

### Tuning Tip
 * Set the threshold above the 95th percentile of normal activity. Measure for
 2–4 weeks before finalizing.


 ###  3. Baselines
 # What It Is
 * Establishes "normal" behavior for a user, host, or network.

 * Alerts on deviations from that normal.

 * The signal is context, not a fixed number.

 * When to Use
 * "Abnormal" is environment-specific and  cannot be a fixed number.

 * Normal behavior varies by user, host, or time of day.

* You want to detect slow-and-low attacks that stay under thresholds.

* You need behavioral analytics (UEBA) or insider-threat detection.

 # Example
A server that normally makes 5 outbound connections/hour suddenly makes 500 —
500 might be normal for a busy server, but it is abnormal for this server.

### Splunk SPL (baseline with eventstats):

index=firewall action=allowed
| bucket _time span=1h
| stats count as connections by _time, src_ip
| eventstats avg(connections) as baseline, stdev(connections) as std by src_ip
| where connections > baseline + (3 * std)


## Baseline Dimensions Table

| Dimension | What to Baseline | Example |
|---|---|---|
| **User** | Login times, locations, devices | User normally logs in 9 AM–6 PM from Mumbai |
| **Host** | Network connections, process execution | Server normally makes 5 outbound connections/hour |
| **Network** | Traffic volume, protocols | Office network normally sends 10 MB/hour to cloud |
| **Application** | API calls, error rates | App normally handles 100 requests/minute |

## Pros and Cons Table

| Pros | Cons |
|---|---|
| Detects slow-and-low attacks | Requires historical data (2–4 weeks minimum) |
| Adapts to environment changes | Complex to implement and maintain |
| Catches insider threats and compromised accounts | Can be fooled by gradual drift |
| Reduces false positives from fixed thresholds | Needs periodic recalibration |


### Tuning Tip
 * Recalibrate baselines when the environment changes (new servers, new users,
seasonal patterns). Use peer groups for new users/hosts without history. 

 ### 4. Allowlists
# What It Is
* Suppresses alerts for known, verified-benign activity.

* The signal is "this is expected, ignore it."

### When to Use
 * The activity is confirmed benign and will remain benign.

 * The activity is unavoidable (scanner, backup service).

* The noise is flooding the queue and hiding real alerts.

* You have verified the source/user/host is trustworthy.

### Example
* Vulnerability scanner IPs — scanners legitimately fail logins during
authenticated checks. Allowlisting the scanner IP prevents false brute force alerts.

### Wazuh rule (level 0 = never alerts):

<rule id="100202" level="0">
  <if_sid>100200</if_sid>
  <srcip>10.0.5.100</srcip>
  <description>Suppress brute force alert from known scanner</description>
</rule>

### Splunk lookup-based allowlist:

index=wineventlog EventCode=4625
| lookup known_scanners src_ip OUTPUT is_scanner
| where isnull(is_scanner) OR is_scanner != "yes"
| stats count as failures by src_ip
| where failures >= 5

###  Common Allowlist Categories

| Category | Example | Review Frequency |
|---|---|---|
| **Scanners** | Nessus, Qualys, OpenVAS IPs | Quarterly |
| **Service accounts** | `svc_backup`, `svc_monitor` | Quarterly |
| **Monitoring tools** | Nagios, Zabbix, Prometheus | Quarterly |
| **Maintenance windows** | Patch Tuesday, backup windows | Monthly |


## Pros and Cons Table

| Pros | Cons |
|---|---|
| Immediately reduces noise | Creates blind spots if overused |
| Simple to implement | Can be abused by attackers (compromised allowlisted host) |
| Frees analyst time for real alerts | Stale allowlists are dangerous |

### Critical Rule
* Never allowlist without verifying the activity is truly and permanently benign.
An allowlisted host that gets compromised becomes an invisible attacker.

## Allowlist Hygiene

| Practice | Why |
|---|---|
| **Document every entry** | Know why it exists and who approved it |
| **Set an expiry date** | Force periodic review |
| **Review quarterly** | Environments change; stale entries create blind spots |
| **Monitor allowlisted sources** | Even allowlisted hosts should be watched |
| **Use narrow scope** | Allowlist a specific IP + rule, not a whole subnet |


## Comparison Table

| Technique | Signal | When to Use | Example | Risk |
|---|---|---|---|---|
| **Correlation Rule** | Sequence (A then B) | Attack = multi-stage pattern | Failed logins → success | Complex; may miss fast attacks |
| **Threshold** | Volume (count) | Attack = high volume | >10 failed logins in 5 min | Wrong threshold = FPs or FNs |
| **Baseline** | Deviation from normal | Attack = abnormal behavior | Server makes 500 connections vs normal 5 | Requires historical data; can drift |
| **Allowlist** | Known-good suppression | Activity is confirmed benign | Scanner IPs, service accounts | Creates blind spots if overused |
 

 ## Decision Guide Table

| If the attack is defined by… | Use… |
|---|---|
| A **sequence** of events | Correlation rule |
| **Volume** of a single event type | Threshold |
| **Deviation** from normal behavior | Baseline |
| **Known-benign** activity flooding the queue | Allowlist |
 

 ### Decision Flowchart
  Is the activity confirmed benign?
├── YES → Allowlist (with review)
└── NO  → Is the attack a sequence of events?
          ├── YES → Correlation rule
          └── NO  → Is volume the signal?
                    ├── YES → Threshold
                    └── NO  → Is "normal" environment-specific?
                              ├── YES → Baseline
                              └── NO  → Re-evaluate; maybe a simple rule


### Combining Techniques: Layered Detection
* Real-world detection often uses multiple techniques together.

# Example: Brute Force Detection (Layered)

| Layer | Technique | Purpose |
|---|---|---|
| **Layer 1** | Threshold | 5+ failures in 5 min → alert |
| **Layer 2** | Correlation | Failures → success → escalate to HIGH |
| **Layer 3** | Baseline | Failure rate 3x above normal for this IP → alert |
| **Layer 4** | Allowlist | Scanner IPs excluded from all layers |

* Result: Fewer false positives, higher fidelity, coverage for both fast and
slow attacks.

# Example: Insider Threat Detection

| Layer | Technique | Purpose |
|---|---|---|
| **Layer 1** | Baseline | User accesses 10x more files than normal |
| **Layer 2** | Correlation | File access + USB insertion + email to external |
| **Layer 3** | Threshold | >100 file downloads in 1 hour |
| **Layer 4** | Allowlist | Exclude HR during onboarding week |

### Common Mistakes
| Mistake | Consequence | Fix |
|---|---|---|
| **Using threshold when sequence is needed** | Misses multi-stage attacks | Add correlation rule |
| **Using baseline without enough data** | Unreliable alerts | Wait 2–4 weeks for baseline |
| **Allowlisting too broadly** | Blind spots | Narrow scope; review quarterly |
| **Setting threshold too low** | Alert fatigue | Measure baseline first |
| **Setting threshold too high** | False negatives | Test with simulated attacks |
| **Never reviewing allowlists** | Stale entries hide attacks | Quarterly review with expiry |
| **Using correlation for simple volume attacks** | Unnecessary complexity | Use threshold instead |
| **Ignoring baselines after environment change** | False positives from drift | Recalibrate after major changes |

## Key Takeaways Table

| Technique | Summary |
|---|---|
| **Correlation** | Use when attack = **sequence** of events (A then B) |
| **Threshold** | Use when attack = **volume** of a single event type |
| **Baseline** | Use when "normal" is **environment-specific** and cannot be a fixed number |
| **Allowlist** | Use when activity is **confirmed benign** and will remain so |



### Golden Rules
* Correlation = sequence. Threshold = count. Baseline = context.
Allowlist = known-good suppression.
* Combine techniques for layered detection.
* Allowlists create blind spots — review quarterly and set expiry dates.
* Baselines require historical data and recalibration as the environment changes.
* Thresholds must be measured, not guessed — use the 95th percentile of normal.
* Every technique has trade-offs — no single technique is sufficient alone.
* Test with simulated attacks before deploying any rule.
* Document everything — why the rule exists, who approved it, when to review.

### Decision Cheat Sheet

Attack = sequence?           → Correlation rule
Attack = high volume?        → Threshold
Attack = abnormal behavior?  → Baseline
Activity = confirmed benign? → Allowlist
Multiple signals?            → Combine all four

