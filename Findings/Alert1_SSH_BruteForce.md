# Triage Note: SSH Brute Force Alert

- **Who:** The attacker IP `192.168.1.50` targeting the `root` user.
- **What:** Multiple failed SSH login attempts were detected (more than 20 attempts in a short time).
- **When:** November 14, 2026, at 15:00 PM.
- **Where:** Host `kali` (Linux Server).
- **Why:** The alert fired because the threshold of failed logins was exceeded, indicating an automated brute-force attack.
- **Verdict:** **True Positive** (It was a simulated attack, but in a real scenario, this is malicious).
- **Next Step:** Block the source IP `192.168.1.50` at the firewall and ensure the `root` account has a strong password and SSH root login is disabled.