# VM6

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



## Foothold



## Privilege Escalation



## Post-Exploit

