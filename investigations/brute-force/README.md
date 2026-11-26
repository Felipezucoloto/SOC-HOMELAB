# SSH Brute Force Attack Detection and Investigation

## 1. Overview

This investigation documents a controlled SSH brute-force simulation performed in a cybersecurity homelab environment.

The objective was to generate repeated SSH authentication failures, collect the resulting security events, and validate the detection capabilities of Wazuh SIEM.

## 2. Lab Environment

| Component | Description |
|---|---|
| SIEM | Wazuh 4.14.10 |
| Attack Machine | Kali Linux |
| Target System | Ubuntu Server 22.04 |
| Protocol | SSH |
| Network | VirtualBox Host-Only |

## 3. Attack Simulation

A controlled SSH brute-force simulation was executed from Kali Linux against the Ubuntu Server target.

The simulation generated repeated authentication failures, producing SSH security events on the target system.

**Evidence:** [`evidence/evidence_bruteforcessh_kali.png`](evidence/evidence_bruteforcessh_kali.png) — Hydra execution against `ssh://192.168.56.112`

## 4. Detection and Monitoring

Wazuh collected and analyzed the SSH authentication events.

The initial authentication failures were identified by the native Wazuh rule:

- Rule ID: 5760
- Description: sshd: authentication failed.
- Severity: Level 5

The Wazuh ruleset also correlated repeated authentication failures and generated a brute-force detection alert.

**Evidence:** [`evidence/evidence_bruteforcessh_wazuh.png`](evidence/evidence_bruteforcessh_wazuh.png) — rule 5760 alert on agent `LINUX-SSH`

## 5. Event Analysis

A total of 3,144 authentication failure alerts associated with the simulation source IP were identified in the Wazuh alert log.

| Indicator | Observed Value |
|---|---|
| Source IP | 192.168.56.113 |
| Authentication failures | 3,144 |
| Protocol | SSH |
| Detection source | Wazuh SIEM |
| Initial detection rule | 5760 |

The source IP was identified through JSON log analysis using command-line tools.

**Logs:** [`logs/ssh-authentication-sample.json`](logs/ssh-authentication-sample.json) — sample of the Wazuh alerts used in the analysis

## 6. Investigation Findings

The investigation demonstrated that Wazuh successfully collected SSH authentication telemetry, identified repeated authentication failures, and generated a correlated brute-force detection alert.

The exercise validated the visibility of authentication activity and the use of native Wazuh detection rules for identifying suspicious SSH access attempts.

## 7. MITRE ATT&CK Mapping

- **T1110 – Brute Force**
- **T1110.001 – Password Guessing**

## 8. Conclusion

This exercise demonstrated a complete detection workflow, from controlled attack simulation to SIEM alert generation and event analysis.

The results provide practical evidence of SSH monitoring, authentication event analysis, and brute-force detection capabilities within the homelab.

All activities were performed in an isolated and authorized lab environment.
