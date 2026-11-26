# Detections

Custom detection rules developed and tested in the homelab.

## Wazuh

Rules are deployed to `/var/ossec/etc/rules/local_rules.xml` on the Wazuh manager, followed by `sudo systemctl restart wazuh-manager`.

| ID | Detection | Log source | ATT&CK | Status | Investigation |
|---|---|---|---|---|---|
| [100100](wazuh/100100-windows-brute-force.xml) | Multiple Windows logon failures from the same source | Windows Security 4625 | T1110 | 🔧 Tuning — events collected, rule not firing yet | [Windows / AD Brute Force](../investigations/windows-ad-brute-force/README.md) |
| [100101](wazuh/100101-logon-after-failures.xml) | Successful logon after multiple failures from the same source | Windows Security 4625 → 4624 | T1110, T1078 | 🔄 In development (depends on 100100) | [Windows / AD Brute Force](../investigations/windows-ad-brute-force/README.md) |
