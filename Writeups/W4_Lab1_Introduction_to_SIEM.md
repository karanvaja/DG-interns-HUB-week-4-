# Writeup: Introduction to SIEM (TryHackMe)

**Room Link:** [https://tryhackme.com/room/introtosiem](https://tryhackme.com/room/introtosiem)
**Difficulty:** Easy
**Status:** Completed (100%)

---

## Overview
This room introduces the fundamentals of Security Information and Event Management (SIEM). It covers how different devices generate logs, the challenges of isolated logs, and how a centralized SIEM solution solves these problems through collection, normalization, correlation, and alerting.

---

## Task 1: Introduction
**Objective:** Understand the basic definition of a SIEM.

*   **Question:** What does SIEM stand for?
*   **Answer:** `Security Information and Event Management system`

---

## Task 2: Logs Everywhere, Answers Nowhere
**Objective:** Differentiate between Host-Centric and Network-Centric log sources. Host-centric logs capture events within a host (e.g., file access, PowerShell execution). Network-centric logs capture communication between devices (e.g., SSH, VPN, firewall traffic).

*   **Question:** Is Registry-related activity host-centric or network-centric?
*   **Answer:** `host-centric`
*   **Question:** Is VPN-related activity host-centric or network-centric?
*   **Answer:** `network-centric`

---

## Task 3: Why SIEM?
**Objective:** Understand the core features of a SIEM solution.
*   **Centralized Log Collection:** Gathers logs from all sources into one place.
*   **Normalization:** Parses raw logs into a consistent format.
*   **Correlation:** Links individual events to identify malicious patterns.
*   **Real-time Alerting:** Triggers alerts based on detection rules.
*   **Dashboards and Reporting:** Visualizes data for analysts.

*   **Question:** Read the task above.
*   **Answer:** `No answer needed`

---

## Task 4: Log Sources and Ingestion
**Objective:** Identify where logs are stored in different systems and how they are ingested into the SIEM.

*   **Windows:** Logs are viewed via Event Viewer and assigned unique Event IDs.
*   **Linux:** Common logs include `/var/log/httpd` (HTTP), `/var/log/cron` (Cron jobs), `/var/log/auth.log` (Authentication), and `/var/log/kern` (Kernel).
*   **Ingestion Methods:** Agent/Forwarder, Syslog, Manual Upload, Port-Forwarding.

*   **Question:** In which location within a Linux environment are HTTP logs stored?
*   **Answer:** `/var/log/httpd`

---

## Task 5: Alerting Process and Analysis
**Objective:** Understand how detection rules are created and how alerts are investigated.

*   **Use-Case 1 (Log Clearing):** Adversaries clear logs to hide tracks. The rule is: If Log Source is WinEventLog AND EventID is 104 → Trigger Alert.
*   **Use-Case 2 (Command Execution):** The rule is: If Log Source is WinEventLog AND EventCode is 4688 AND NewProcessName contains `whoami` → Trigger Alert.
*   **Triage:** Analysts must determine if an alert is a True Positive (needs investigation) or False Positive (needs tuning).

*   **Question:** Which Event ID is generated when event logs are removed?
*   **Answer:** `104`
*   **Question:** What type of alert may require tuning?
*   **Answer:** `False Positive`

---

## Task 6: Lab Work (Static Site)
**Objective:** Analyze a triggered alert in the provided static dashboard.

*   **Question:** After clicking on the Start Suspicious Activity button, which process caused the alert?
*   **Answer:** `cudominer.exe` 
    *(Note: This is a known cryptocurrency miner)*
*   **Question:** Find the event that caused the alert and identify the user responsible for the process execution.
*   **Answer:** `chris`
*   **Question:** What is the hostname of the suspect user?
*   **Answer:** `HR_02`
*   **Question:** Examine the rule and the suspicious process; which term matched the rule that caused the alert?
*   **Answer:** `miner`
*   **Question:** Which option best represents the event? Choose from the following: False Positive / True Positive
*   **Answer:** `True Positive`
*   **Question:** Selecting the right ACTION will display the FLAG. What is the FLAG?
*   **Answer:** `THM{000_SIEM_INTRO}`

### Triage Note Example (Task 6)
> **Who:** User `chris`
> **What:** Process execution of `cudominer.exe` (Cryptominer)
> **When:** (Timestamp from the lab dashboard)
> **Where:** Host `HR_02`
> **Why:** The detection rule triggered because the process name matched the keyword "miner". This is malicious activity.
> **Action:** True Positive. Isolate the host and escalate to L2.

---

## Task 7: Conclusion
**Objective:** Complete the room and review next steps.

*   **Question:** Complete this room.
*   **Answer:** `No answer needed`