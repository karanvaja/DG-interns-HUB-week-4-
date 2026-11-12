# Q2 — Log Ingestion Methods: Splunk Universal Forwarder vs Wazuh Agent

## Core Conclusion
- **Splunk Universal Forwarder (UF)** = a pure log transport tool. It only collects logs
  and forwards them to the Splunk indexer. All parsing, detection, and analysis happens
  downstream in Splunk.
- **Wazuh Agent** = a security sensor. It forwards logs AND performs file integrity
  monitoring (FIM), vulnerability scanning, security configuration assessment (SCA),
  rootkit detection, and active response directly on the endpoint.
- They are both called "agents," but they solve different problems:
  UF = **shipping**, Wazuh Agent = **shipping + detecting + responding**.

---

## Common Log Ingestion Methods

| Method | How It Works | Best For |
|---|---|---|
| **Agent** | Software installed on the host collects and forwards logs/telemetry | Servers, workstations needing continuous monitoring |
| **Syslog** | Network devices actively push logs over UDP/TCP 514 | Firewalls, routers, switches |
| **API** | SIEM periodically pulls events from cloud/SaaS platforms via REST | Microsoft 365, AWS CloudTrail, Okta |
| **File Monitoring** | Agent watches specific files/directories for changes | Application logs, audit logs, custom log files |

---

## Splunk Universal Forwarder vs Wazuh Agent

| Dimension | Splunk Universal Forwarder (UF) | Wazuh Agent |
|---|---|---|
| **Design Positioning** | Pure log forwarder — a "transport truck" | Security monitoring agent with built-in detection and response |
| **Endpoint Processing** | Only collects and forwards; no analysis on endpoint | Runs modules locally: FIM, SCA, malware detection, inventory |
| **Host-Based Detection** | **None** — detection runs on Splunk indexer/search head | **Core capability** — built-in HIDS, rootkit detection, vulnerability scanning |
| **Active Response** | Not supported | Supported — can block connections, kill processes, delete files |
| **Data Processing** | Cannot parse/route/transform; needs a heavy forwarder for that | Decodes and enriches data locally, then sends to manager for rule matching |
| **Resource Footprint** | Extremely low (no UI, forwarding components only) | Approx. 35 MB RAM (lightweight but active) |
| **Configuration Files** | `inputs.conf` (what to collect), `outputs.conf` (where to send) | `ossec.conf`, remotely managed by the Wazuh manager |
| **Communication Security** | SSL encryption and compression supported | Encrypted and authenticated channel (TCP/UDP) with flow control |
| **Best For** | Environments already using Splunk as the analytics platform, only needing log centralization | Teams needing endpoint security telemetry (FIM, SCA, vuln scan) + SIEM integration |

---

## Decision Guide

| If you need… | Choose |
|---|---|
| Centralized log search in Splunk only | Splunk Universal Forwarder |
| Endpoint-level detection (FIM, rootkit, vuln scan) | Wazuh Agent |
| Active response on the host | Wazuh Agent |
| Minimal footprint log shipping | Splunk Universal Forwarder |
| One agent that both forwards logs and detects threats | Wazuh Agent |
| Integration with an existing Splunk deployment | Splunk Universal Forwarder |

---

## Key Takeaways
- **UF does not detect** — it only ships. Detection happens in Splunk.
- **Wazuh Agent detects and responds locally**, then reports to the manager.
- Ingestion method choice (agent vs syslog vs API vs file monitoring) depends on the
  data source and whether you need host-level security telemetry.
- Both agents support encrypted communication, but only Wazuh Agent provides
  built-in security modules beyond log forwarding.