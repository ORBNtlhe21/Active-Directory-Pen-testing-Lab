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

### Nmap Scan

> **Tool:** `nmap`  
> **Purpose:** Identify open ports and running services on the target system.

```bash
nmap -sCV -p- 192.168.50.129

PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP
445/tcp   open  microsoft-ds  Windows SMB file sharing
464/tcp   open  kpasswd5      Kerberos password change
593/tcp   open  ncacn_http    RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Global Catalog LDAP
3269/tcp  open  tcpwrapped
5357/tcp  open  http          Microsoft HTTPAPI httpd 2.0
5985/tcp  open  http          Windows Remote Management (WinRM)
9389/tcp  open  mc-nmf        .NET Message Framing
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0
```
> **Purpose:** Identify open ports and running services on the target system.

## 2. Enumeration

###LDAP Enumeration with NetExec (nxc)
> **Tool:** `nxc`
> **Purpose:** Perform anonymous LDAP queries to extract usernames and domain structure

```bash
nxc ldap 192.168.50.129 -u '' -p '' --users

[*] Total records returned: 67
DC=orb,DC=corp
CN=Administrator,CN=Users,DC=orb,DC=corp
CN=Guest,CN=Users,DC=orb,DC=corp
CN=krbtgt,CN=Users,DC=orb,DC=corp
CN=alice wonderland,CN=Users,DC=orb,DC=corp  
CN=bob builder,CN=Users,DC=orb,DC=corp
CN=jerry cantrell,CN=Users,DC=orb,DC=corp
CN=kurt cobain,CN=Users,DC=orb,DC=corp 
CN=mssql_svc,CN=Users,DC=orb,DC=corp
CN=http_svc,CN=Users,DC=orb,DC=corp
CN=Floor,CN=Users,DC=orb,DC=corp
CN=IT,CN=Users,DC=orb,DC=corp
CN=HR,CN=Users,DC=orb,DC=corp
CN=Management,CN=Users,DC=orb,DC=corp
```
> **Success:** Anonymous access to LDAP allowed us to enumerate domain users and service accounts without credentials. 
