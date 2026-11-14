# SPL Detection Searches

Here are the 5 SPL searches created for threat detection:

### 1. Failed logons per user (EventCode 4625)
- **SPL:** `index=* EventCode=4625 | stats count by TargetUserName | sort - count`
- **MITRE:** T1110 (Brute Force)

### 2. PowerShell process creation (Sysmon Event ID 1)
- **SPL:** `index=* source="*Sysmon*" EventCode=1 Image="*\\powershell.exe" | table _time, host, User, CommandLine`
- **MITRE:** T1059.001 (PowerShell)

### 3. New user account creation (EventCode 4720)
- **SPL:** `index=* EventCode=4720 | table _time, host, SubjectUserName, NewTargetUserName`
- **MITRE:** T1136 (Create Account)

### 4. SSH failures per source IP
- **SPL:** `index=lab "Failed password" | rex field=_raw "from (?<attacker_ip>\d+\.\d+\.\d+\.\d+)" | stats count by attacker_ip | sort - count`
- **MITRE:** T1110 (Brute Force)

### 5. Rare parent-child process pairs
- **SPL:** `index=* source="*Sysmon*" EventCode=1 | stats count by ParentImage, Image | where count < 5`
- **MITRE:** T1055 (Process Injection)