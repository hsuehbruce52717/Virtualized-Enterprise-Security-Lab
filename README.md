# 🛡️ Virtualized Enterprise Security Lab
### SOC Simulation Environment — Based on Project Security: [Enterprise 101](https://docs.projectsecurity.io/e101/overview/)

A hands-on, multi-VM enterprise security lab built in VirtualBox to develop and validate real-world SOC analyst skills. The lab simulates a realistic business network, executes a full end-to-end cyber attack, and practices detection engineering, alert correlation, and incident response using Wazuh SIEM.

---

## 🎯 Objectives

- Build and configure a virtualized enterprise network from the ground up
- Deploy a SIEM with endpoint agents, custom detection rules, and log ingestion
- Simulate a realistic cyber attack across multiple stages of the kill chain
- Detect and investigate each attack stage through Wazuh alert correlation
- Map findings to the MITRE ATT&CK framework

---

## 🏗️ Lab Architecture

**Network:** VirtualBox NAT Network (project-x-nat) — 10.0.0.0/24

| VM | OS | Role | IP |
|---|---|---|---|
| project-x-dc | Windows Server 2025 | Domain Controller (AD, DNS, DHCP) | 10.0.0.5 |
| project-x-win-client | Windows 11 Enterprise | Windows Workstation | 10.0.0.100 |
| project-x-linux-client | Ubuntu Desktop 22.04.5 | Linux Workstation | 10.0.0.101 |
| project-x-corp-svr | Ubuntu Desktop 22.04.5 | Jumpbox & Email Server | 10.0.0.8 |
| project-x-sec-box | Ubuntu Desktop 22.04.5 | Security Server (Wazuh) | 10.0.0.10 |
| project-x-sec-work | Security Onion | Security Playground (provisioned, not used in simulation) | 10.0.0.103 |
| project-x-attacker | Kali Linux 2025.2 | Attacker Machine | dynamic |

---

## 🔧 Lab Setup Summary

### Active Directory
Configured Windows Server 2025 as the domain controller for `corp.project-x-dc.com`. Set up DNS, DHCP, and provisioned domain user accounts. Joined all Windows and Linux machines to the domain — Linux machines connected via Samba Winbind.

### Corporate Server — Jumpbox & Email
Set up `project-x-corp-svr` as a Jumpbox running Docker Engine. Deployed **MailHog** as a Docker container using Docker Compose to simulate a corporate SMTP email server. Deployed a Bash email poller script on the Linux workstation to simulate an employee email inbox.

### Wazuh SIEM
Deployed the full Wazuh stack (Indexer, Server, Dashboard) on the dedicated security server. Installed agents on the Windows workstation, Domain Controller, and Linux workstation. Organized agents into Windows and Linux groups and configured custom log ingestion via `agent.conf` for Windows Event logs and Linux auth/audit logs.

### Vulnerable Environment
Deliberately misconfigured the environment to simulate realistic attack conditions — enabling SSH with password authentication, WinRM, and RDP across endpoints. Planted a sensitive file on the Domain Controller for exfiltration simulation. Intentionally left `project-x-corp-svr` without a Wazuh agent to demonstrate a detection blind spot.

### Custom Detection Rules & Alerts
Implemented detection coverage across three attack surfaces:

| Alert | Trigger | Rule |
|---|---|---|
| 3 Failed SSH Attempts | Repeated SSH authentication failures | Wazuh Rule `5760` |
| WinRM Logon | Successful WinRM authentication via Kerberos | Rule `60106` / Event ID `4624` |
| Sensitive File Accessed | Modification to `secrets.txt` | Custom Rule `100002` (`local_rules.xml`) |

Configured **File Integrity Monitoring (FIM)** on the sensitive directory with real-time monitoring and change reporting.

---

## ⚔️ Cyber Attack Simulation

Executed a full attack chain from initial access to persistence, playing the role of a financially motivated threat actor targeting sensitive data.

```
Reconnaissance → Brute Force SSH → Post-Access Recon → Phishing
→ Credential Harvest → Lateral Movement → Privilege Escalation
→ RDP to Domain Controller → Data Exfiltration → Persistence
```

**Reconnaissance** — Scanned the network with Nmap to identify open ports and services on the corporate jumpbox.

**Initial Access** — Brute-forced SSH credentials on the jumpbox using Hydra. After gaining access, discovered the MailHog email service and identified an internal user account.

**Phishing** — Deployed a credential-harvesting site on the attacker machine, sent a spear-phishing email via MailHog, and captured the victim's credentials.

**Lateral Movement** — Used harvested credentials to SSH into the Linux workstation. Discovered WinRM running on the Windows workstation, performed password spraying with NetExec, and gained a shell via Evil-WinRM.

**Privilege Escalation & Lateral Movement 2.0** — From the Windows workstation, used xFreeRDP to move laterally into the Domain Controller using Administrator credentials.

**Data Exfiltration** — Located and exfiltrated the sensitive file from the Domain Controller to the attacker machine via SCP.

**Persistence** — Created a rogue domain admin account and deployed a PowerShell reverse shell via a Windows Scheduled Task, caught with a Netcat listener on the attacker machine.

---

## 🗺️ MITRE ATT&CK Mapping

| Tactic | Technique | Tool | Detection |
|---|---|---|---|
| Reconnaissance | T1595 – Active Scanning | Nmap | Network traffic |
| Credential Access | T1110 – Brute Force | Hydra | Wazuh Rule 5760 |
| Initial Access | T1566 – Phishing | Apache2, MailHog | Email delivery |
| Lateral Movement | T1021.004 – SSH | SSH | Wazuh auth.log |
| Lateral Movement | T1021.006 – WinRM | NetExec, Evil-WinRM | Wazuh Rule 60106 |
| Lateral Movement | T1021.001 – RDP | xFreeRDP | Wazuh Rule 92653 |
| Exfiltration | T1048 – Exfil Over Alt Protocol | SCP | Custom Rule 100002 |
| Persistence | T1136 – Create Account | net user | AD account audit |
| Persistence | T1053.005 – Scheduled Task | schtasks, PowerShell | Windows Event logs |
| C2 | T1059.001 – PowerShell | reverse.ps1, Netcat | Process creation log |

---

## 🛠️ Tools & Technologies

**Defense:** `Wazuh` `Active Directory` `Windows Server 2025` `MailHog` `Docker` `Docker Compose` `Samba Winbind`

**Offense:** `Nmap` `Hydra` `Apache2` `NetExec` `Evil-WinRM` `xFreeRDP` `Netcat` `SCP`

**Infrastructure:** `VirtualBox` `Windows 11 Enterprise` `Ubuntu Desktop 22.04` `Kali Linux` `Python` `Bash` `PowerShell`

---

## 🔍 Skills Demonstrated

- SIEM deployment, agent management, and centralized log configuration
- Custom detection rule creation and alert tuning in Wazuh
- File Integrity Monitoring on sensitive directories
- Active Directory setup — DNS, DHCP, domain join (Windows & Linux)
- Docker-based service deployment
- End-to-end offensive simulation — recon, phishing, lateral movement, exfiltration, persistence
- MITRE ATT&CK framework mapping and kill chain analysis

---

## 📚 Reference

Built following the **Project Security — Enterprise 101** curriculum:
🔗 https://docs.projectsecurity.io/e101/overview/

---

## 📌 Notes

- No real credentials, IPs, or sensitive data are included in this repository
- VM images are not included due to file size
- Security Onion was provisioned as part of the lab build but is reserved for the NA101 curriculum
- Built as part of a self-directed transition into cybersecurity from a background as an organic chemist
