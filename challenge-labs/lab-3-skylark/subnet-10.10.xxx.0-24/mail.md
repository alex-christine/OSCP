---
description: Domain-connected Windows Machine
---

# MAIL

## Enumeration

Machine is hosted at `10.10.XXX.13`. Output from Nmap scan run [here](./#nmap):

```
Nmap scan report for MAIL.skylark.com (10.10.175.13)
Host is up (0.091s latency).
Not shown: 65517 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
25/tcp    open  smtp          hMailServer smtpd
| smtp-commands: mail.skylark.com, SIZE 20480000, AUTH LOGIN, HELP
|_ 211 DATA HELO EHLO MAIL NOOP QUIT RCPT RSET SAML TURN VRFY
80/tcp    open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: IIS Windows Server
110/tcp   open  pop3          hMailServer pop3d
|_pop3-capabilities: USER UIDL TOP
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
143/tcp   open  imap          hMailServer imapd
|_imap-capabilities: CHILDREN IDLE IMAP4 SORT completed IMAP4rev1 OK RIGHTS=texkA0001 ACL CAPABILITY QUOTA NAMESPACE
445/tcp   open  microsoft-ds?
587/tcp   open  smtp          hMailServer smtpd
| smtp-commands: mail.skylark.com, SIZE 20480000, AUTH LOGIN, HELP
|_ 211 DATA HELO EHLO MAIL NOOP QUIT RCPT RSET SAML TURN VRFY
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
49670/tcp open  msrpc         Microsoft Windows RPC
49671/tcp open  msrpc         Microsoft Windows RPC
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): IBM z/OS 1.11.X (85%)
OS CPE: cpe:/o:ibm:zos:1.11
Aggressive OS guesses: IBM z/OS 1.11 (85%)
No exact OS matches for host (test conditions non-ideal).
Service Info: Host: mail.skylark.com; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2024-08-28T01:09:14
|_  start_date: N/A
|_nbstat: NetBIOS name: MAIL, NetBIOS user: <unknown>, NetBIOS MAC: 00:50:56:bf:ad:a4 (VMware)
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
```

## Foothold

Before machine enumeration is even started, initial access is [provided](./#credential-spray) as Administrator via the `SKYLARK\backup_service` account. I can use Impacket's PsExec to access the machine:

{% code overflow="wrap" %}
```bash
impacket-psexec SKYLARK/backup_service:It4Server@mail.skylark.com
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/SL-MAIL-SystemPsExec.png" alt=""><figcaption><p>SYSTEM access on MAIL</p></figcaption></figure>

`SYSTEM` access achieved.

## Post Exploit

One of the first things to note is that this machine can reach the `10.20.XXX.0/24` subnet.

