---
description: Domain-connected Windows machine on second-level internal network
---

# CLIENT01

## Enumeration

Output from `nmap` command run [here](./#nmap):

```
Nmap scan report for vm7.skylark (10.20.111.110)
Host is up (0.041s latency).
Not shown: 65522 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: SKYLARK
|   NetBIOS_Domain_Name: SKYLARK
|   NetBIOS_Computer_Name: CLIENT01
|   DNS_Domain_Name: SKYLARK.com
|   DNS_Computer_Name: client01.SKYLARK.com
|   DNS_Tree_Name: SKYLARK.com
|   Product_Version: 10.0.22000
|_  System_Time: 2024-09-02T00:02:10+00:00
| ssl-cert: Subject: commonName=client01.SKYLARK.com
| Not valid before: 2024-08-31T23:23:05
|_Not valid after:  2025-03-02T23:23:05
|_ssl-date: 2024-09-02T00:02:51+00:00; 0s from scanner time.
5040/tcp  open  unknown
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
49670/tcp open  msrpc         Microsoft Windows RPC
49672/tcp open  msrpc         Microsoft Windows RPC
49673/tcp open  msrpc         Microsoft Windows RPC
49702/tcp open  msrpc         Microsoft Windows RPC
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): IBM z/OS 2.1.X (85%)
OS CPE: cpe:/o:ibm:zos:2.1
Aggressive OS guesses: IBM z/OS 2.1 (85%)
No exact OS matches for host (test conditions non-ideal).
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
|_nbstat: NetBIOS name: CLIENT01, NetBIOS user: <unknown>, NetBIOS MAC: 00:50:56:bf:f2:4d (VMware)
| smb2-time: 
|   date: 2024-09-02T00:02:08
|_  start_date: N/A
```

This machine is probably accessible with something from the compromised [`DC`](../subnet-10.10.xxx.0-24/dc.md#dc-sync) machine.

## Foothold



## Privilege Escalation



## Post-Exploit

