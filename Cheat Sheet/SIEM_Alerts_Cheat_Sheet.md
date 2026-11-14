# SIEM Alert Triage Cheat Sheet

| SIEM Alert | Likely Cause | First Check |
|---|---|---|
| **Multiple Failed Logins** (Brute Force) | Automated password guessing (Hydra, etc.) or user typo. | Check if the source IP is internal/external. Look for a successful login afterwards. |
| **New User Created** (Event 4720) | IT Admin creating a valid account OR Attacker creating persistence. | Verify with the IT team. Check who executed the creation process. |
| **PowerShell Execution** (Sysmon 1) | Legitimate admin script OR Malicious execution (C2, download payload). | Check the CommandLine. Look for encoded commands (`-enc`). |
| **Port Scan Detected** | Vulnerability scanner (Authorized) OR Attackers doing Recon. | Check if the source IP belongs to the IT scanning team or an unknown external IP. |
| **Unusual Parent-Child Process** | Malware (e.g., Word spawning CMD) OR legitimate installer. | Check the process tree. Is MS Word supposed to run `cmd.exe`? |
| **Malware Detected (EDR)** | Real virus/ransomware OR False positive from an update. | Check the file path and hash on VirusTotal. Look at recent changes. |
| **Suspicious USB Insertion** | Insider threat (data theft) OR Employee plugging in a mouse. | Check what files were copied after the USB was inserted. |
| **DNS Tunneling Alert** | Data exfiltration over DNS OR bad application config. | Check the domain being queried. Is it known? Are there massive TXT records? |
| **Account Lockout** (Event 4740) | User forgot password OR Brute force attack. | Check the source IP. Is the account a privileged user? |
| **File Integrity Change** (FIM) | System update, config change OR Malware modification. | Check which file changed. Was it a system file (e.g., `hosts`)? |