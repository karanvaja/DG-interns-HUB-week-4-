# Triage Note: Vulnerability Scanner Detection

- **Who:** Source IP `10.0.5.100` (Internal Scanner).
- **What:** Nmap scan detected against the internal web server.
- **When:** November 14, 2026, at 02:00 AM.
- **Where:** Web Server `WEB-01`.
- **Why:** The alert fired due to multiple port connection attempts. However, the source IP belongs to the authorized Nessus vulnerability scanner used by the IT team.
- **Verdict:** **False Positive** (Authorized activity).
- **Next Step:** Add the IP `10.0.5.100` to the Splunk/Wazuh allowlist so it doesn't trigger alerts in the future.