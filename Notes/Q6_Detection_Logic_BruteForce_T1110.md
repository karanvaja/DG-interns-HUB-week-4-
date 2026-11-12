# Q6 — Detection Logic from a TTP: SSH / Windows Brute Force (MITRE T1110)

## Overview

This note covers how to translate a **TTP (Tactic, Technique, Procedure)** into
working detection logic. Using **brute force** as the example, we build detection
logic for both SSH (Linux) and Windows, then map it to **MITRE ATT&CK T1110**.

**Detection goal:** Alert when **5 or more failed logons occur from one source
within 5 minutes** — a classic brute force pattern.

---

## MITRE T1110 — Brute Force

| Aspect | Detail |
|---|---|
| **Technique ID** | T1110 |
| **Tactic** | Credential Access (TA0006) |
| **Description** | Adversaries may use brute force techniques to gain access to accounts when passwords are unknown or when password hashes are obtained |
| **Common sub-techniques** | T1110.001 (Password Guessing), T1110.002 (Password Cracking), T1110.003 (Password Spraying), T1110.004 (Credential Stuffing) |

### Sub-Technique Mapping

| Attack Pattern | MITRE Sub-Technique |
|---|---|
| Single account targeted repeatedly | **T1110.001** — Password Guessing |
| Many accounts targeted from one source | **T1110.003** — Password Spraying |
| Credentials from breach lists tried | **T1110.004** — Credential Stuffing |
| Offline hash cracking | **T1110.002** — Password Cracking |

---

## Detection Condition (Abstract Logic)

  ```
  IF failed_logon events
  FROM same source (IP or user)
  COUNT >= 5
  WITHIN 5 minutes
  THEN generate alert (brute force)
```
 
 
**Components:**
- **Event type:** failed authentication (SSH `Failed password` / Windows Event 4625)
- **Threshold:** 5 failures
- **Time window:** 5 minutes (300 seconds)
- **Scope:** same source IP (or same target user)
- **Action:** alert with MITRE T1110 mapping

---

## SSH Brute Force Detection

### Data Source
- Linux authentication log: `/var/log/auth.log` or `/var/log/secure`
- Splunk sourcetype: `linux_secure`
- Wazuh: built-in decoder `sshd`

### Splunk SPL

```
    index=linux sourcetype=linux_secure "Failed password"
   | bucket _time span=5m
   | stats count as failures by _time,  src_ip, user
   | where failures >= 5
```

**Step-by-step:**
| Step | Command | What it does |
|---|---|---|
| 1 | `index=linux sourcetype=linux_secure "Failed password"` | Base search: find failed SSH logins |
| 2 | `\| bucket _time span=5m` | Group events into 5-minute windows |
| 3 | `\| stats count as failures by _time, src_ip, user` | Count failures per window, IP, and user |
| 4 | `\| where failures >= 5` | Keep only windows with 5+ failures |

### Wazuh Correlation Rule

```xml
<rule id="100200" level="10" frequency="5" timeframe="300">
  <if_matched_sid>5716</if_matched_sid>   <!-- sshd auth failed -->
  <same_source_ip />
  <description>SSH brute force: 5+ failed logins from same IP in 5 min</description>
  <mitre>
    <id>T1110</id>
    <id>T1110.001</id>
  </mitre>
 </rule>
```
### Field breakdown:
 This table explains each field in the SSH brute force correlation rule
(rule ID 100200) and what it controls.

| Field | Value | Meaning |
|---|---|---|
| **id** | 100200 | Custom rule number (100000+ for custom rules) |
| **level** | 10 | Triage priority (0–15 scale) |
| **frequency** | 5 | Must match 5 times before firing |
| **timeframe** | 300 | Within 300 seconds (5 minutes) |
| **if_matched_sid** | 5716 | Parent rule = sshd auth failed |
| **same_source_ip** | (present) | All failures must come from same IP |
| **mitre** | T1110, T1110.001 | Maps to Brute Force / Password Guessing |


### Parent rule reference (built-in Wazuh rule 5716):
  ```xml 
  <rule id="5716" level="5">
  <decoded_as>sshd</decoded_as>
  <match>Failed password for</match>
  <description>sshd: authentication failed</description>
 </rule>
```
### Windows Brute Force Detection
  *Data Source
  *Windows Security Event Log
  *Event ID 4625 = failed logon
  *Splunk sourcetype: wineventlog (or WinEventLog:Security)
  *Wazuh: built-in decoder for Windows Event Log

## Splunk SPL
  
```xml 
   index=wineventlog EventCode=4625
  | bucket _time span=5m
  | stats count as failures by _time,      src_ip, TargetUserName
  | where failures >= 5
```
# Step-by-step:
 
| Step | Command | What it does |
|---|---|---|
| **1** | `index=wineventlog EventCode=4625` | Base search: find failed Windows logons |
| **2** | `\| bucket _time span=5m` | Group events into 5-minute windows |
| **3** | `\| stats count as failures by _time, src_ip, TargetUserName` | Count failures per window, IP, and target user |
| **4** | `\| where failures >= 5` | Keep only windows with 5+ failures |

### Wazuh Correlation Rule
 
  <rule id="100201" level="10" frequency="5" timeframe="300">
  <if_matched_sid>60122</if_matched_sid>   <!-- Windows failed logon -->
  <same_source_ip />
  <description>Windows brute force: 5+ failed logons in 5 min</description>
  <mitre>
    <id>T1110</id>
    <id>T1110.001</id>
  </mitre>
  </rule>

* Note: Rule ID 60122 is Wazuh's built-in Windows failed logon rule. The exact
ID may vary by Wazuh version — verify in your ruleset/rules/ directory. 

### Tuning Considerations
 
 | Consideration | Action | Reason |
|---|---|---|
| **Known scanner IPs** | Allowlist them | Vulnerability scanners legitimately fail logins |
| **Environment baseline** | Adjust threshold to observed normal | Some environments have more legitimate failures |
| **Follow-up success** | Escalate severity if success follows failures | Indicates likely compromise |
| **Time window** | Adjust based on attack speed | Faster attacks need shorter windows |
| **Account scope** | `same_user` vs `same_source_ip` | Choose based on attack pattern (spraying vs guessing) |
| **Service accounts** | Exclude known service accounts with old cached creds | Prevents noise from stale credentials |


## Example: Follow-Up Success Correlation

index=linux sourcetype=linux_secure
| transaction src_ip maxspan=5m
| search "Failed password" AND "Accepted password"
| table _time, src_ip, user

 * This detects the pattern: failures followed by a success from the same IP —
a strong indicator of successful brute force.

### MITRE ATT&CK Mapping Summary

| Detection | MITRE Technique | Sub-Technique | Tactic |
|---|---|---|---|
| **SSH brute force (single user)** | T1110 | T1110.001 (Password Guessing) | Credential Access |
| **SSH brute force (many users, one IP)** | T1110 | T1110.003 (Password Spraying) | Credential Access |
| **Windows brute force (single user)** | T1110 | T1110.001 (Password Guessing) | Credential Access |
| **Windows brute force (many users, one IP)** | T1110 | T1110.003 (Password Spraying) | Credential Access |
| **Credentials from breach list** | T1110 | T1110.004 (Credential Stuffing) | Credential Access |


### Complete Detection Logic Flow
| Stage | What Happens | Tools |
|---|---|---|
| **1. Collect** | Raw logs are gathered from endpoints | Wazuh Agent, Splunk UF, syslog |
| **2. Decode** | Raw text is parsed into structured fields | Wazuh decoders, Splunk props.conf |
| **3. Aggregate** | Events are grouped by time window and source | Splunk `bucket`/`stats`, Wazuh `frequency`/`timeframe` |
| **4. Threshold** | Count is compared to the rule threshold | Splunk `where`, Wazuh rule level |
| **5. Enrich** | Context is added (asset, user, MITRE) | Lookup tables, Wazuh `mitre` tag |
| **6. Alert & Triage** | Alert is generated and routed to analyst | Splunk alert actions, Wazuh dashboard |



### Key Takeaways

* Detection logic = threshold + time window + scope (same IP or same user).

* SSH: search "Failed password" in linux_secure; Windows: search EventCode=4625.

* Wazuh correlation rules use frequency, timeframe, if_matched_sid, and
same_source_ip to implement the same logic.

* Map every brute-force rule to T1110 (and the appropriate sub-technique:
.001 for guessing, .003 for spraying, .004 for stuffing).

* Tune thresholds to environment baseline; allowlist known scanners.

* Escalate when failures are followed by a success — that's likely a real
compromise, not just noise.