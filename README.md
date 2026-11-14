# Week 4 Internship Report: SIEM in Practice
## Splunk, Wazuh & Log Ingestion Engineering

**Intern Name:** Vaja Karan  
**Domain:** SOC Analyst  
**Report Period:** Week 4 (November 2026)  
**Document Status:** Completed

---

## 1. Executive Summary
During Week 4 of the SOC Analyst internship, operational focus shifted to core Security Information and Event Management (SIEM) engineering, centralized log ingestion, custom detection authoring, and incident triage. 

This report details the architectural principles and practical deployment of two industry-standard platforms: **Splunk Enterprise** and **Wazuh HIDS/SIEM**. Practical activities included establishing log ingestion pipelines, executing attack simulations (Nmap port scanning and Hydra SSH brute-force attacks), engineering custom detection logic using Splunk Search Processing Language (SPL) and XML-based Wazuh rules, constructing operational dashboards, and conducting structured alert triage based on standard investigative methodology.

---

## 2. Theoretical Architecture & Concepts

### 2.1 SIEM Operational Ecosystem
A robust Defensive Security Architecture integrates multiple operational technologies. The table below delineates the core differences across the enterprise security ecosystem:

| Platform | Primary Function | Core Telemetry Source | Focus & Capability |
|---|---|---|---|
| **SIEM** | Log centralization, correlation, threat detection | Syslog, Event Logs, API streams | Real-time correlation across disparate sources |
| **Log Management** | Long-term log storage, archiving, basic search | Raw text files, system logs | Data retention, compliance auditing, basic retrieval |
| **SOAR** | Orchestration, automated playbook execution | SIEM alerts, Threat Intelligence feeds | Automated incident response and workflow mitigation |
| **EDR** | Endpoint-level threat detection, process tracking | Kernel telemetry, process trees | Deep endpoint visibility, memory analysis, isolation |

### 2.2 Log Ingestion Pipelines: Splunk UF vs. Wazuh Agent
Ingesting event telemetry efficiently into a central indexer is critical. While both Splunk Universal Forwarders (UF) and Wazuh Agents run as host endpoint daemons, their operational architecture differs fundamentally.

| Ingestion Metric | Splunk Universal Forwarder (UF) | Wazuh Agent |
|---|---|---|
| **Primary Design** | Transport shipper ("data truck") | Security monitoring & response sensor |
| **Endpoint Processing** | Minimal (collection & forwarding only) | Active (FIM, SCA, local decoding, vuln scans) |
| **Host-Based Detection** | None (executed on Search Head) | Built-in HIDS & rootkit detection engine |
| **Active Mitigation** | Requires add-on automated actions | Native active response (IP blocking) |
| **Configuration** | inputs.conf & outputs.conf | ossec.conf (managed centrally via Manager) |

### 2.3 Search Processing Language (SPL) Core Mechanics
Splunk processes unstructured and structured event data using its proprietary SPL. Efficient SPL queries follow a structured data flow pipeline:
1. **Search & Filter:** Retrieve events matching criteria (`index=lab sourcetype=syslog`).
2. **Field Extraction:** Filter data dynamically using `where` or `rex`.
3. **Transformation:** Compute statistical metrics using `stats`, `chart`, `timechart`.
4. **Formatting:** Structure the final visualization using `table`, `eval`, `rename`, `sort`.

### 2.4 Wazuh Architecture & Rule Severity Matrix
Wazuh processes telemetry through a two-stage evaluation engine:
- **Decoders:** Extract key-value pairs (e.g., `srcip`, `dstuser`, `action`) from raw log strings.
- **Rules Engine:** Compares extracted decoders against hierarchical XML rules.

| Rule Level | Classification | Operational Meaning | Action Required |
|---|---|---|---|
| **0 - 2** | Ignored / Informational | Operational events, status checks | Ignored by default |
| **3 - 5** | Low Severity | Routine system events, successful logons | Stored for audit trail |
| **6 - 9** | Medium Severity | Invalid login attempts, config changes | Monitored by L1 SOC Analysts |
| **10 - 13** | High Severity | Multiple failed logins, suspected brute force | Immediate triage & verification |
| **14 - 15** | Critical Severity | Proven rootkit detection, active malware | Immediate escalation & active response |

### 2.5 SOC Operational Framework
**Detection Evaluation Matrix:**
- **True Positive (TP):** Alert fires on actual malicious activity. (Action: Incident Response).
- **False Positive (FP):** Alert fires on benign activity. (Action: Tune baseline rule).
- **True Negative (TN):** Benign activity occurs, no alert fires.
- **False Negative (FN):** Malicious activity occurs but evades detection. (Action: Enhance logic).

**The 5 Triage Questions:** Who (Identity), What (Action), When (Timestamp), Where (IP/Host), Why (Intent).

**SOC Escalation Tiers:**
- **L1:** Continuous monitoring, alert validation, initial triage, baseline tuning.
- **L2:** Advanced investigation, threat hunting, root-cause analysis, containment.
- **L3:** Deep forensic analysis, custom signature development, threat hunting.

---

## 3. TryHackMe Labs
- **Introduction to SIEM:** Differentiated host-centric telemetry (Windows Event Logs, Linux Syslog) from network-centric telemetry (Firewall logs, NetFlow). Analyzed SIEM normalization workflows.
- **Splunk: Exploring SPL:** Mastered practical search filtering, dataset formatting, and statistical aggregation. Applied search pipelines and constructed time-series visualizations.

---

## 4. Hands-On Technical Execution

### 4.1 Task 1 & 2: Splunk Installation & Log Ingestion
- **Environment Setup:** Installed Splunk Enterprise on Kali Linux.
- **Index Creation:** Created a dedicated index designated as `lab`.
- **File Monitoring:** Configured local file monitoring to stream `/var/log/auth.log` into the `lab` index.
- **Validation Query:** `index=lab sourcetype=syslog | stats count by process, host` (Verified successful ingestion).

### 4.2 Task 3: Wazuh Deployment & File Integrity Monitoring (FIM)
- **Deployment:** Deployed Wazuh Manager and enrolled a Windows 11 endpoint agent.
- **FIM Configuration:** Modified `ossec.conf` to monitor `C:\Users\Public\karan`.
- **Verification:** Created and modified test files. Verified real-time telemetry generation on the dashboard (Rule IDs 550, 554, 553).

### 4.3 Task 4: Attack Simulation & Telemetry Analysis
Two attack scenarios were simulated from an attacker host (Kali Linux):
1. **Reconnaissance (Nmap Scan):** Executed a TCP port sweep against target services.
2. **Credential Access (Hydra SSH Brute Force):** Executed automated dictionary attack: `hydra -l root -P pass.txt ssh://127.0.0.1`
- **Telemetry Verification:** Observed a massive spike in failed authentication logs in Splunk (Failed password for invalid user...).

### 4.4 Task 5: Detection Engineering, Alerts & Dashboards

**5 SPL Detection Searches Engineered:**
1. **SSH Brute Force:** `index=lab "Failed password" | stats count by src_ip, dst_user | where count > 10`
2. **PowerShell Execution:** `index=lab EventCode=4688 Image="*powershell.exe" CommandLine="*-enc*"`
3. **Local Account Creation:** `index=lab (EventCode=4720 OR "useradd") | table _time, host, user`
4. **Web Reconnaissance:** `index=lab sourcetype=access_combined (status=404 OR status=403) | stats count by src_ip, uri_path`
5. **Failed Logon Summary:** `index=lab "Failed password" | timechart span=5m count by host`

**Scheduled Alerts Configured:**
- **Alert 1 (SSH Brute Force):** Triggers when failed logon attempts exceed 15 within 5 minutes. Severity: High.
- **Alert 2 (Unauthorized User Creation):** Triggers immediately upon local account creation.

**SOC Overview Dashboard:**
Built a 4-panel dashboard containing: Total Authentication Failures (Single Value), Failed Logons Trended (Timechart), Failures by Target Username (Pie Chart), and Top 10 Attacking IPs (Table).

**Custom Wazuh Rule (ID 100051):**
```xml
<group name="syslog, sshd, ">
  <rule id="100051" level="10">
    <if_matched_sid>5716</if_matched_sid>
    <same_source_ip />
    <frequency>6</frequency>
    <timeframe>120</timeframe>
    <description>Possible SSH Brute Force Attack Detected</description>
    <mitre><id>T1110</id></mitre>
  </rule>
</group>