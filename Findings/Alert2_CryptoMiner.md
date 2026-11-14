# Triage Note: Suspicious Process (Crypto Miner)

- **Who:** User `chris` on host `HR_02`.
- **What:** Execution of the process `cudominer.exe` was detected.
- **When:** November 14, 2026, at 10:15 AM.
- **Where:** Windows Workstation `HR_02`.
- **Why:** The rule matched the keyword "miner", and the process is a known cryptocurrency mining tool, which consumes system resources for malicious profit.
- **Verdict:** **True Positive**.
- **Next Step:** Isolate the host `HR_02` from the network immediately, kill the process, and run a full antivirus/EDR scan to find how it was installed.
