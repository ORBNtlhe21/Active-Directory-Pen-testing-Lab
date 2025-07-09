# Active-Directory-Pen-testing-Lab
This project simulates a real-world internal penetration test in a controlled lab environment. The goal is to compromise a Windows Server 2022 Domain Controller using a Kali Linux attacker machine through enumeration, exploitation, and post-exploitation techniques.

Project Summary

- **Attacker Machine:** Kali Linux
- **Target Machine:** Windows Server 2022 (Active Directory Domain Controller)
- **Domain Name:** `orb.corp0`
- **Objective:** Gain initial access, enumerate the domain, escalate privileges, and extract sensitive data (e.g., password hashes, domain admin credentials).
- **Scope:** Internal black-box test — attacker has direct network access but no initial credentials.

Lab Architecture

| Machine          | Role                            | IP Address     |
|------------------|---------------------------------|----------------|
| Kali Linux       | Attacker                        | 192.168.50.10  |
| Windows Server   | Domain Controller (DC)          | 192.168.50.129 |
| Windows 10       | Windows client                  |                |

## 1. Information Gathering

### 🔹 Nmap Scan

> **Tool:** `nmap`  
> **Purpose:** Identify open ports and running services on the target system.

```bash
nmap -sCV -p- 192.168.50.129
