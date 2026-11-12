# Q1 — What is a SIEM?

## Definition
- **SIEM = Security Information and Event Management.**
- It is a central platform that **collects** security logs from many sources,
  **normalizes** them into a common format, **correlates** related events,
  **detects** suspicious patterns, and **alerts** analysts.
- In short: it turns scattered raw logs into actionable security signals.

## Components

| Component | What it does |
|---|---|
| **Log Collection** | Agents, syslog, APIs, file monitors pull events from endpoints, firewalls, identity systems, cloud |
| **Parsing / Normalization** | Converts different log formats into consistent fields (src_ip, user, event_type) |
| **Correlation Engine** | Links events across sources using rules, thresholds, sequences, or behaviour logic |
| **Alerting** | Fires notifications when detection logic matches |
| **Storage / Indexing** | Stores events for fast search and retention (compliance) |
| **Search & Investigation** | Analysts query data to reconstruct incidents |
| **Dashboards & Reporting** | Visual summaries, compliance reports, trend views |

## Data Flow
1. **Source** produces a log (endpoint, firewall, cloud, app).
2. **Collector** (agent/syslog/API) forwards it.
3. **Parser/Normalizer** maps it to a common schema.
4. **Correlation/Detection** engine evaluates rules.
5. **Alert** is generated if logic matches.
6. **Analyst triage** → investigation → response → tuning feedback loop.

## SIEM vs Log Management vs SOAR vs EDR

| Capability | Primary Purpose | Focus | Typical Examples |
|---|---|---|---|
| **SIEM** | Centralized detection, correlation, investigation | Broad: endpoints + network + identity + cloud | Splunk ES, Wazuh, Microsoft Sentinel |
| **Log Management** | Collect, store, retain logs for audit/troubleshooting | Broad but **passive** — no security correlation | Graylog, rsyslog, CloudWatch Logs |
| **SOAR** | Orchestrate & automate response workflows | **Automation/process**, not detection | Splunk SOAR, Palo Alto XSOAR, Tines |
| **EDR** | Detect & respond on individual endpoints | **Endpoint-only**, deep host telemetry + containment | CrowdStrike, SentinelOne, Defender for Endpoint |

## Key Takeaways
- SIEM = **visibility + correlation** across the environment.
- Log management = **storage + search**, no detection logic.
- SOAR = **does the response work automatically**.
- EDR = **deep endpoint sensor**; feeds SIEM but does not replace it.
- They **complement** each other in a layered SOC.
