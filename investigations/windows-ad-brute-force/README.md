# Windows / Active Directory Brute Force Detection and Investigation

## 1. Overview

This investigation documents a controlled brute-force simulation against a domain account in the homelab's Active Directory environment.

The objective was to generate repeated Windows authentication failures over SMB, collect the resulting security events on the Domain Controller, and answer the key SOC question: **did the attacker eventually get in?**

Unlike the [SSH brute force case](../brute-force/README.md), this exercise focused on Windows Security events (4625 / 4624), the failure-to-success pattern, and building a custom Wazuh correlation rule.

## 2. Lab Environment

| Component | Description |
|---|---|
| SIEM | Wazuh 4.14.10 (manager `LAB2`) |
| Attack Machine | Kali Linux (`192.168.56.114`, workstation name `KALI`) |
| Target System | Windows Server — Domain Controller `DC-01.corp.local` (Wazuh agent `AD`, `192.168.56.107`) |
| Domain | `CORP.LOCAL` (NetBIOS `CORP`) |
| Target Account | `CORP\joao.teste` (lab test user) |
| Protocol | SMB (TCP 445), NTLM authentication |
| Network | VirtualBox Host-Only (`192.168.56.0/24`) |

## 3. Attack Simulation

Repeated SMB authentication attempts with an invalid password were executed from Kali Linux using `smbclient`:

```bash
for i in $(seq 12); do
  smbclient -L //192.168.56.107 -U 'CORP.LOCAL\joao.teste%wrongpass'
done
```

Each attempt returned `NT_STATUS_LOGON_FAILURE`. A final attempt with the valid password succeeded and listed the DC shares (`ADMIN$`, `C$`, `IPC$`, `NETLOGON`, `SYSVOL`):

```bash
smbclient -L //192.168.56.107 -U 'CORP.LOCAL\joao.teste'
```

## 4. Detection and Monitoring

The Wazuh agent on the Domain Controller forwarded the Windows Security events to the manager:

- Each failed logon (Event ID **4625**) was identified by the native Wazuh rule **60122** (group `authentication_failed`).
- The successful logon (Event ID **4624**) was decoded and indexed as `AUDIT_SUCCESS`.
- A correlation alert (identified by the `previous_output` field) was generated during the burst of failures.

A custom correlation rule was also developed: [`detections/wazuh/100100-windows-brute-force.xml`](../../detections/wazuh/100100-windows-brute-force.xml). It **did not fire** during the test — most likely because a native Wazuh correlation rule matching the same events takes precedence. Tuning is in progress (see Troubleshooting below).

**Evidence:** [`evidence/01-wazuh-4625-failures-from-kali.png`](evidence/01-wazuh-4625-failures-from-kali.png)

## 5. Event Analysis

### Failed logons (4625)

| Field | Observed Value |
|---|---|
| Events | 30 |
| Source IP (`ipAddress`) | 192.168.56.114 |
| Workstation name | KALI |
| Target account | joao.teste |
| Logon type | 3 (network) |
| Authentication package | NTLM |
| Status / SubStatus | `0xC000006D` / `0xC000006A` (bad password) |
| Time window | ~17:12:40 – 17:13:28 (UTC-3) |

### Successful logon (4624)

| Field | Observed Value |
|---|---|
| Time | 17:14:06 (UTC-3) — **38 seconds after the last failure** |
| Source IP (`ipAddress`) | 192.168.56.114 |
| Workstation name | KALI |
| Account | CORP\joao.teste |
| Logon type | 3 (network) |
| Authentication package | NTLM (NTLM V2) |
| Logon ID | `0x29B029` |

**Evidence:**
- [`evidence/02-wazuh-4624-success-joao-teste.png`](evidence/02-wazuh-4624-success-joao-teste.png)
- [`evidence/03-wazuh-4624-success-full-log.png`](evidence/03-wazuh-4624-success-full-log.png)

### Separating signal from noise

During the same window, several 4624 events for `Administrador` appeared on the DC. They were **not related to the attack**: Kerberos authentication, source address IPv6 link-local (`fe80::…`), no workstation name, typical of the DC's own activity and the administrator session.

The attacker's logon is distinguishable by: **NTLM + source 192.168.56.114 + workstation KALI + the same account that had just failed 30 times**.

**Evidence:** [`evidence/04-noise-administrator-kerberos-ipv6.png`](evidence/04-noise-administrator-kerberos-ipv6.png)

## 6. Troubleshooting and Lessons Learned

1. **Hydra over RDP failed** (`all children were disabled due to too many connection errors`). RDP has Network Level Authentication (NLA) enabled by default, and Hydra's `rdp` module doesn't handle it well. **Fix:** switched the attack vector to SMB.
2. **Ports 445/3389 showed as `filtered` from Kali.** Windows Firewall was dropping inbound connections from the attacker. RDP from the physical host worked, which showed that firewall decisions depend on source and network profile — access from one host does not prove access from another.
3. **NetExec timed out** even with port 445 open. `smbclient` worked under the same conditions, isolating the problem to the tool rather than the lab.
4. **Wrong domain name assumption.** Logons failed even with the correct password because the domain prefix was assumed incorrectly. The correct one (`CORP.LOCAL\joao.teste`) was confirmed through the `smbclient` prompt.
5. **The custom correlation rule did not fire,** although all underlying events reached Wazuh. Native correlation rules can take precedence over custom rules matching the same events. Next step: confirm which rule generated the correlation alert and attach the custom rule as a child of it.

## 7. False Positives and Tuning

| Possible false positive | Tuning idea |
|---|---|
| Service account or scheduled task retrying with an outdated password | Allow-list known service accounts |
| User mistyping their password several times | Combine with logon type and number of failures |
| NAT/proxy concentrating many users behind one IP | Count distinct target accounts per source (also separates password spraying from brute force) |

## 8. Investigation Findings

- **30 failed logons followed by a successful logon from the same source, for the same account, within 38 seconds.**
- In a real environment this sequence indicates a **likely compromised credential**, and the alert should be escalated, not just closed as "brute force attempt".
- Recommended response: disable or reset `joao.teste`, review its activity after logon ID `0x29B029`, block `192.168.56.114`, and check whether the same source targeted other accounts.

## 9. MITRE ATT&CK Mapping

- **T1110.001 – Brute Force: Password Guessing**
- **T1078.002 – Valid Accounts: Domain Accounts** (successful logon after the guessing)

## 10. Conclusion

This exercise covered the full detection workflow on Windows: controlled attack, event collection from a Domain Controller, event analysis, noise separation and rule development.

The most important outcome was not the alert itself but the finding: **the failures were followed by a success**, which turns a "brute force attempt" into a probable account compromise.

All activities were performed in an isolated and authorized lab environment.
