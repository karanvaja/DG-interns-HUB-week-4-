# Q4 — Wazuh Architecture, Decoders, Rules, and Rule Levels 0–15

## Wazuh Architecture Overview

Wazuh is an open-source SIEM/XDR platform built on four core components that work
together to collect, analyze, store, and visualize security data.

| Component | Role | Key Functions |
|---|---|---|
| **Wazuh Agent** | Endpoint sensor installed on monitored hosts | Collects logs, file integrity monitoring (FIM), vulnerability data, security configuration assessment (SCA), system inventory, malware detection; forwards to manager |
| **Wazuh Manager** | Central analysis engine | Receives agent data; decodes raw logs; applies rules; generates alerts; forwards to indexer |
| **Wazuh Indexer** | Storage and search engine (OpenSearch-based) | Stores alerts and events; provides full-text search and analytics |
| **Wazuh Dashboard** | Web UI | Configuration, visualization, investigation, compliance reporting |

### Data Flow
  Endpoint (Agent)
→ collects logs, FIM, SCA, vuln data
→ forwards to Manager
→ Manager decodes raw logs into structured fields
→ Manager applies rules (atomic + correlation)
→ Manager generates alerts
→ forwards to Indexer
→ Indexer stores and indexes alerts/events
→ Dashboard queries Indexer for visualization and investigation



---

## Component Details

### 1. Wazuh Agent

| Aspect | Detail |
|---|---|
| **Deployment** | Installed on servers, workstations, containers, cloud instances |
| **Collection modules** | Log collector, FIM (file integrity monitoring), SCA (security configuration assessment), vulnerability detector, malware detection, system inventory, active response |
| **Communication** | Encrypted and authenticated channel to manager (TCP/UDP); supports flow control |
| **Resource footprint** | Lightweight (~35 MB RAM typical) |
| **Configuration** | `ossec.conf`, remotely managed by manager |
| **Active response** | Can execute scripts to block connections, kill processes, delete malicious files |

### 2. Wazuh Manager

| Aspect | Detail |
|---|---|
| **Role** | Central brain — receives, decodes, correlates, alerts |
| **Decoders** | Parse raw log lines into structured fields |
| **Rules** | Match decoded fields to determine security relevance |
| **Correlation** | Composite rules link multiple events (frequency, timeframe, same_source_ip) |
| **Alert output** | Forwards alerts to indexer via Filebeat or direct integration |
| **Configuration** | `ossec.conf`, `decoder.xml`, `rules.xml` (custom rules in `local_rules.xml`) |

### 3. Wazuh Indexer

| Aspect | Detail |
|---|---|
| **Technology** | OpenSearch-based (fork of Elasticsearch) |
| **Role** | Stores alerts and events; provides full-text search |
| **Indexing** | Alerts indexed by time, rule ID, severity, agent, etc. |
| **Retention** | Configurable index lifecycle management (ILM) policies |
| **Query** | Dashboard and API query the indexer for investigation |

### 4. Wazuh Dashboard

| Aspect | Detail |
|---|---|
| **Role** | Web UI for the entire platform |
| **Functions** | Configuration management, alert visualization, compliance dashboards (PCI DSS, HIPAA, GDPR), agent management, rule/decoder testing |
| **Visualization** | Time-series charts, tables, maps, custom dashboards |
| **Investigation** | Search alerts, drill into events, view raw logs |

---

## How Decoders Work

### Decoders parse raw log lines into structured fields.

Without a decoder, the SIEM sees only a text string. With a decoder, it sees
structured fields it can match against rules.

### Example raw log line: 
    Failed password for invalid user admin from 192.168.1.50 port 22 ssh2


### Decoder extracts:
| Field | Value |
|---|---|
| `srcuser` | admin |
| `srcip` | 192.168.1.50 |
| `srcport` | 22 |
| `program_name` | sshd |

### Types of decoders:
- **Built-in decoders** — ship with Wazuh for common sources (SSH, Apache, Windows Event Log, syslog, etc.)
- **Custom decoders** — written for proprietary or environment-specific log formats
- **Parent-child decoders** — parent decoder identifies the log type; child decoders extract specific fields

 ### Decoder example (concept):
```xml
<decoder name="sshd">
  <program_name>sshd</program_name>
</decoder>

<decoder name="sshd-failed">
  <parent>sshd</parent>
  <match>Failed password for</match>
  <regex>from (\S+) port (\d+)</regex>
  <order>srcip, srcport</order>
</decoder>
```

### How Rules Work : 
Rules match decoded fields to determine if an event is security-relevant.

Rules are evaluated after decoding. If a rule matches, an alert is generated with
the rule's severity level.

### Rule Structure

| Field | Purpose |
|---|---|
| **id** | Unique rule number (built-in rules: 1–99999; custom rules: 100000+) |
| **level** | Severity 0–15 (0 = ignored, 15 = critical) |
| **frequency** | How many times the rule must match before firing |
| **timeframe** | Within how many seconds the matches must occur |
| **if_matched_sid** | Parent rule ID to correlate against (used in composite rules) |
| **same_source_ip** / **same_user** | Scope of correlation — all matched events must share the same source IP or user |
| **description** | Human-readable alert text shown to analysts |
| **mitre** | MITRE ATT&CK technique mapping (e.g., T1110 for brute force) |

## Example Rule Using These Fields

```xml
<rule id="100200" level="10" frequency="5" timeframe="300">
  <if_matched_sid>5716</if_matched_sid>
  <same_source_ip />
  <description>SSH brute force: 5+ failed logins in 5 minutes</description>
  <mitre><id>T1110</id></mitre>
</rule>
```


Fires on a single event.
Level 5 = low-impact user error.


 ### Composite/Correlation Rule (Multiple Events)   

 <rule id="100200" level="10" frequency="5" timeframe="300">
  <if_matched_sid>5716</if_matched_sid>   <!-- sshd auth failed -->
  <same_source_ip />
  <description>SSH brute force: 5+ failed logins in 5 minutes</description>
  <mitre><id>T1110</id></mitre>
</rule>
````

Fires when the parent rule (5716) matches 5 times within 300 seconds.
same_source_ip ensures all failures come from the same IP.
Level 10 = triage priority.

### Rule Levels 0–15
| Level | Meaning | Handling | Example |
|---|---|---|---|
| 0 | Ignored or grouping rule; never alerts | Used as parent for suppression | A rule that groups events but doesn't alert |
| 2–4 | Low-relevance system notifications or successful events | Context only | Successful login, system startup |
| 5–7 | User errors, first-time or low-impact events | Watch for patterns | Failed login for non-existent user |
| 8–10 | First-time-seen events, repeated errors, possible attacks | Triage | SSH brute force threshold reached |
| 11–12 | Important security/integrity events | Prioritize | File integrity change on critical binary |
| 13–15 | High-importance events; likely successful attacks | Immediate response | Rootkit detected, malware execution |


 ### Important Caveats

Level is a generic severity estimate from the rule author — not a guarantee
of actual risk.

Asset criticality matters more than raw level: a level 10 alert on a critical
database server outranks a level 12 alert on a dev laptop.

Event sequence matters: 5 failures (level 5 each) followed by a success
(level 3) is far more significant than any single event's level.

Custom rules should be tuned to your environment's baseline.



### — Key Takeaways: Wazuh Architecture

| Concept | Summary |
|---|---|
| **Agent** | Endpoint sensor; collects and forwards logs, FIM, SCA, vulnerability data; can actively respond |
| **Manager** | Central brain; receives data, decodes logs, correlates events, generates alerts |
| **Indexer** | Stores and indexes alerts/events for fast search and analytics |
| **Dashboard** | Web UI for configuration, visualization, investigation, and compliance reporting |
| **Decoder** | Parses raw log text → structured fields (e.g., srcip, srcuser) |
| **Rule** | Matches structured fields → alert or no alert |
| **Atomic rule** | Matches a single event (e.g., one failed login) |
| **Composite rule** | Correlates multiple events using frequency + timeframe + scope (same_source_ip/same_user) |
| **Levels 0–15** | Severity indicator; context (asset criticality, event sequence) matters more than raw level |
