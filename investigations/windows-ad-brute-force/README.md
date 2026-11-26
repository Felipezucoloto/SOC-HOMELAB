# Windows / Active Directory Brute Force Detection and Investigation

## 1. Overview

This investigation documents a controlled brute-force simulation against a domain account in the homelab's Active Directory environment.

The objective was to generate repeated Windows authentication failures over SMB, collect the resulting security events (Event ID 4625), and validate a **custom Wazuh correlation rule** that detects multiple failed logons from the same source.

Unlike the [SSH brute force case](../brute-force/README.md), which relied on native Wazuh rules, this exercise involved writing, troubleshooting and tuning a custom detection.

## 2. Lab Environment

| Component | Description |
|---|---|
| SIEM | Wazuh 4.14.10 |
| Attack Machine | Kali Linux (`192.168.56.114`) |
| Target System | Windows Server — Domain Controller `DESKTOP-VR0EOHC` (`192.168.56.107`) |
| Domain | `CORP.LOCAL` |
| Target Account | `CORP.LOCAL\joao.teste` (lab test user) |
| Protocol | SMB (TCP 445) |
| Network | VirtualBox Host-Only (`192.168.56.0/24`) |

## 3. Attack Simulation

Repeated SMB authentication attempts with an invalid password were executed from Kali Linux using `smbclient`:

```bash
for i in $(seq 12); do
  smbclient -L //192.168.56.107 -U 'CORP.LOCAL\joao.teste%wrongpass'
done
```

Each attempt returned `NT_STATUS_LOGON_FAILURE` and generated a Windows Security Event ID **4625** (failed logon, logon type 3) on the Domain Controller.

A final attempt with the valid password succeeded and listed the DC shares (`ADMIN$`, `C$`, `IPC$`, `NETLOGON`, `SYSVOL`), generating Event ID **4624** (successful logon).

**Evidence (to be added):** `evidence/`

## 4. Detection and Monitoring

The Wazuh agent on the Domain Controller forwarded the Windows Security events to the manager.

Each failed logon was identified by the native Wazuh rule:

- Rule ID: 60122
- Description: Logon failure — unknown user or bad password
- Group: `authentication_failed`

Repeated failures were then correlated by a **custom rule**:

- Rule ID: **100100** — [`detections/wazuh/100100-windows-brute-force.xml`](../../detections/wazuh/100100-windows-brute-force.xml)
- Logic: 8 or more events from the `authentication_failed` group within 120 seconds, from the same `win.eventdata.ipAddress`
- Severity: Level 10

The rule fired as expected during the simulation.

**Evidence (to be added):** `evidence/`

## 5. Event Analysis

| Indicator | Observed Value |
|---|---|
| Source IP | 192.168.56.114 |
| Target host | DESKTOP-VR0EOHC (Domain Controller) |
| Target account | joao.teste |
| Windows Event ID | 4625 (failures) / 4624 (success) |
| Logon type | 3 (network) |
| Native rule | 60122 |
| Custom correlation rule | 100100 |

**Logs (to be added):** `logs/`

## 6. Troubleshooting and Lessons Learned

Building this detection required solving several issues, all documented here because they reflect real SOC troubleshooting:

1. **Hydra over RDP failed** (`all children were disabled due to too many connection errors`). RDP on Windows has Network Level Authentication (NLA) enabled by default, and Hydra's `rdp` module doesn't handle it well. **Fix:** switched the attack vector to SMB.
2. **Ports 445/3389 showed as `filtered` from Kali.** Windows Firewall was dropping inbound connections from the attacker. RDP from the physical host worked, which showed that firewall decisions depend on source and network profile — access from one host does not prove access from another.
3. **NetExec timed out** even with port 445 open. `smbclient` worked under the same conditions, isolating the problem to the tool rather than the lab.
4. **Wrong domain name assumption.** Logons failed even with the correct password because the NetBIOS domain was assumed incorrectly. The correct prefix (`CORP.LOCAL\joao.teste`) was confirmed through the `smbclient` prompt.
5. **The first version of rule 100100 never fired.** It relied on `if_matched_sid 60122`. Rewriting it to use `if_matched_group authentication_failed` made it independent of a specific rule ID and it fired as expected.

## 7. False Positives and Tuning

| Possible false positive | Tuning idea |
|---|---|
| Service account or scheduled task retrying with an outdated password | Allow-list known service accounts |
| User mistyping their password several times | Raise threshold or combine with logon type |
| NAT/proxy concentrating many users behind one IP | Count distinct target accounts per source (also separates password spraying from brute force) |

## 8. MITRE ATT&CK Mapping

- **T1110 – Brute Force**
- **T1110.001 – Password Guessing**

## 9. Conclusion

This exercise demonstrated a complete detection engineering workflow for Windows authentication: controlled attack simulation, event collection from a Domain Controller, custom correlation rule development, troubleshooting and validation.

**Next step:** rule [`100101`](../../detections/wazuh/100101-logon-after-failures.xml) — detecting a **successful logon after multiple failures** from the same source (4625 → 4624), a stronger indicator of a compromised credential (T1078 – Valid Accounts).

All activities were performed in an isolated and authorized lab environment.
