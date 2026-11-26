# SOC-HOMELAB
Multi-VM Cybersecurity Homelab focused on SOC Operations, Wazuh SIEM, Active Directory, Network Security, Threat Detection, and Blue Team practices.
# 🛡️ Cybersecurity Homelab
### SOC Operations | Blue Team | Network Security | Threat Hunting

![Status](https://img.shields.io/badge/Status-In%20Progress-yellow)
![SIEM](https://img.shields.io/badge/SIEM-Wazuh-blue)
![Platform](https://img.shields.io/badge/Platform-VirtualBox-orange)
![Focus](https://img.shields.io/badge/Focus-Blue%20Team-green)

---

## 📌 About the Project

Welcome to my Cybersecurity Homelab!

This project is a multi-VM virtualized environment designed to simulate an enterprise IT infrastructure and develop practical cybersecurity skills.

The lab focuses on Security Operations Center (SOC) activities, Blue Team operations, security monitoring, threat detection, network administration, and incident investigation.

My goal is to gain hands-on experience with real-world security tools, understand attack behaviors, improve detection capabilities, and document practical security investigations.

## 🏗️ Lab Infrastructure

| Virtual Machine | Operating System | Purpose |
|---|---|---|
| LAB2 | Ubuntu Server 22.04 | Wazuh SIEM |
| Kali | Kali Linux | Security Testing |
| Windows 10 | Windows 10 | Endpoint Monitoring |
| Network Lab | Ubuntu Server 22.04 | Routing, Switching, SSH and Firewall |
| AD Server | Windows Server | Active Directory and Domain Services |
| Windows 10 2 | Windows 10 |  Windows Monitoring |

## 🖥️ Lab Architecture

The environment is composed of interconnected virtual machines responsible for different infrastructure and security functions.

<img width="1672" height="941" alt="image" src="https://github.com/user-attachments/assets/676110a8-96fe-413c-a46d-439a985607a9" />


## 🔍 Security Monitoring & SIEM

Centralized security monitoring using Wazuh.

Main activities:

- SIEM deployment and configuration.
- Endpoint integration.
- Log collection and analysis.
- Security alert investigation.
- Custom detection rules.
- File Integrity Monitoring (FIM).
- Threat Hunting.

### Connected Agents

 <img width="956" height="735" alt="Dispositivos" src="https://github.com/user-attachments/assets/80569654-03b4-4bb0-b1a8-4b48af9f8fef" />


## 🪟 Windows Security

Windows security monitoring and Active Directory administration.

- Windows Event Log analysis.
- Sysmon configuration.
- PowerShell logging.
- Advanced Audit Policies.
- Active Directory administration.
- User and Group Management.
- Group Policy Objects (GPO).

## 🌐 Network Infrastructure & Security

Practical network administration and security configuration.

- Routing and Switching.
- VLANs and Network Segmentation.
- Inter-VLAN Routing.
- SSH Administration.
- Firewall Configuration.
- Network Traffic Analysis.
- TCP/IP, DNS and DHCP.

## 🐉 Security Testing

Kali Linux is used for controlled security testing and attack simulation.

- Network Reconnaissance.
- Service Enumeration.
- Authentication Testing.
- Controlled Attack Simulations.
- Detection Validation.

## 🎯 Blue Team & Threat Hunting

- Security Event Analysis.
- Log Correlation.
- MITRE ATT&CK Mapping.
- Detection Engineering.
- Incident Investigation.
- Threat Hunting.
- Security Monitoring Improvements.

## 🧰 Technologies & Tools

- Wazuh SIEM
- Ubuntu Server 22.04
- Kali Linux
- Windows 10
- Windows Server
- Sysmon
- Active Directory
- PowerShell
- Wireshark
- SSH
- VirtualBox

## 🔎 Investigations

| Case | Target | Key finding | ATT&CK |
|---|---|---|---|
| [SSH Brute Force](investigations/brute-force/README.md) | Ubuntu Server (SSH) | 3,144 authentication failures from a single source, detected by native Wazuh rules | T1110.001 |
| [Windows / AD Brute Force](investigations/windows-ad-brute-force/README.md) | Domain Controller (SMB) | 30 failed logons followed by a successful logon from the same source 38 seconds later — likely compromised credential | T1110.001, T1078.002 |

## 🛠️ Detections

Custom Wazuh rules and their status: [detections/README.md](detections/README.md)

## 📂 Repository Structure

```text
SOC-HOMELAB/
├── detections/
│   ├── README.md          # Rule index (ATT&CK, status, investigation)
│   └── wazuh/             # Custom Wazuh rules (local_rules.xml)
├── investigations/
│   ├── brute-force/              # SSH brute force case
│   └── windows-ad-brute-force/   # Windows / AD brute force case
│       ├── README.md
│       ├── evidence/      # Screenshots
│       └── logs/          # Exported alerts
└── README.md
```


## ⚠️ Disclaimer

This project is intended for educational purposes and authorized security testing only.

All attack simulations and security experiments are conducted within a controlled environment using systems under my administration.

---

**Continuous Learning | Hands-on Practice | Cybersecurity Engineering**
