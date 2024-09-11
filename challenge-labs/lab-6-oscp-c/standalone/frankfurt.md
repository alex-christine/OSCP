---
description: Standalone Machine
---

# FRANKFURT

Writeup for standalone machine hosted at `192.168.xxx.156`.

## Enumeration

Output from Nmap scan [run earlier](./#nmap):

{% code title="standalone-tcp_stealth-all.nmap" %}
```
Nmap scan report for frankfurt.oscp.exam (192.168.202.156)
Host is up (0.052s latency).
Not shown: 65519 closed tcp ports (reset)
PORT     STATE SERVICE  VERSION
21/tcp   open  ftp      vsftpd 3.0.3
| ssl-cert: Subject: commonName=oscp.example.com/organizationName=Vesta Control Panel/stateOrProvinceName=California/countryName=US
| Not valid before: 2022-11-08T08:16:51
|_Not valid after:  2023-11-08T08:16:51
|_ssl-date: TLS randomness does not represent time
22/tcp   open  ssh      OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 7e:62:fd:92:52:6f:64:b1:34:48:8d:1e:52:f1:74:c6 (RSA)
|   256 1b:f7:0c:c7:1b:05:12:a9:c5:c5:78:b7:2a:54:d2:83 (ECDSA)
|_  256 ee:d4:a1:1a:07:b4:9f:d9:e5:2d:f6:b8:8d:dd:bf:d7 (ED25519)
25/tcp   open  smtp     Exim smtpd 4.90_1
| smtp-commands: oscp.exam Hello frankfurt.oscp.exam [192.168.45.157], SIZE 52428800, 8BITMIME, PIPELINING, AUTH PLAIN LOGIN, CHUNKING, STARTTLS, HELP
|_ Commands supported: AUTH STARTTLS HELO EHLO MAIL RCPT DATA BDAT NOOP QUIT RSET HELP
| ssl-cert: Subject: commonName=oscp.example.com/organizationName=Vesta Control Panel/stateOrProvinceName=California/countryName=US
| Not valid before: 2022-11-08T08:16:51
|_Not valid after:  2023-11-08T08:16:51
|_ssl-date: 2024-09-08T20:22:59+00:00; +33s from scanner time.
53/tcp   open  domain   ISC BIND 9.11.3-1ubuntu1.18 (Ubuntu Linux)
| dns-nsid: 
|_  bind.version: 9.11.3-1ubuntu1.18-Ubuntu
80/tcp   open  http     nginx
|_http-title: oscp.exam &mdash; Coming Soon
| http-methods: 
|_  Potentially risky methods: TRACE
110/tcp  open  pop3     Dovecot pop3d
| ssl-cert: Subject: commonName=oscp.example.com/organizationName=Vesta Control Panel/stateOrProvinceName=California/countryName=US
| Not valid before: 2022-11-08T08:16:51
|_Not valid after:  2023-11-08T08:16:51
|_ssl-date: TLS randomness does not represent time
|_pop3-capabilities: RESP-CODES STLS TOP CAPA AUTH-RESP-CODE SASL(PLAIN LOGIN) UIDL USER PIPELINING
143/tcp  open  imap     Dovecot imapd (Ubuntu)
| ssl-cert: Subject: commonName=oscp.example.com/organizationName=Vesta Control Panel/stateOrProvinceName=California/countryName=US
| Not valid before: 2022-11-08T08:16:51
|_Not valid after:  2023-11-08T08:16:51
|_ssl-date: TLS randomness does not represent time
|_imap-capabilities: LOGIN-REFERRALS STARTTLS LITERAL+ AUTH=PLAIN ENABLE post-login IDLE AUTH=LOGINA0001 have SASL-IR more listed capabilities OK ID IMAP4rev1 Pre-login
465/tcp  open  ssl/smtp Exim smtpd 4.90_1
|_ssl-date: 2024-09-08T20:22:13+00:00; -12s from scanner time.
| smtp-commands: oscp.exam Hello frankfurt.oscp.exam [192.168.45.157], SIZE 52428800, 8BITMIME, PIPELINING, AUTH PLAIN LOGIN, CHUNKING, HELP
|_ Commands supported: AUTH HELO EHLO MAIL RCPT DATA BDAT NOOP QUIT RSET HELP
| ssl-cert: Subject: commonName=oscp.example.com/organizationName=Vesta Control Panel/stateOrProvinceName=California/countryName=US
| Not valid before: 2022-11-08T08:16:51
|_Not valid after:  2023-11-08T08:16:51
587/tcp  open  smtp     Exim smtpd 4.90_1
| ssl-cert: Subject: commonName=oscp.example.com/organizationName=Vesta Control Panel/stateOrProvinceName=California/countryName=US
| Not valid before: 2022-11-08T08:16:51
|_Not valid after:  2023-11-08T08:16:51
|_ssl-date: 2024-09-08T20:22:30+00:00; +3s from scanner time.
| smtp-commands: oscp.exam Hello frankfurt.oscp.exam [192.168.45.157], SIZE 52428800, 8BITMIME, PIPELINING, AUTH PLAIN LOGIN, CHUNKING, STARTTLS, HELP
|_ Commands supported: AUTH STARTTLS HELO EHLO MAIL RCPT DATA BDAT NOOP QUIT RSET HELP
993/tcp  open  ssl/imap Dovecot imapd (Ubuntu)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=oscp.example.com/organizationName=Vesta Control Panel/stateOrProvinceName=California/countryName=US
| Not valid before: 2022-11-08T08:16:51
|_Not valid after:  2023-11-08T08:16:51
|_imap-capabilities: LOGIN-REFERRALS have SASL-IR more post-login LITERAL+ IMAP4rev1 AUTH=PLAIN listed ENABLE OK IDLE capabilities ID AUTH=LOGINA0001 Pre-login
995/tcp  open  ssl/pop3 Dovecot pop3d
|_ssl-date: TLS randomness does not represent time
|_pop3-capabilities: CAPA RESP-CODES AUTH-RESP-CODE SASL(PLAIN LOGIN) USER UIDL TOP PIPELINING
| ssl-cert: Subject: commonName=oscp.example.com/organizationName=Vesta Control Panel/stateOrProvinceName=California/countryName=US
| Not valid before: 2022-11-08T08:16:51
|_Not valid after:  2023-11-08T08:16:51
2525/tcp open  smtp     Exim smtpd 4.90_1
| smtp-commands: oscp.exam Hello frankfurt.oscp.exam [192.168.45.157], SIZE 52428800, 8BITMIME, PIPELINING, AUTH PLAIN LOGIN, CHUNKING, STARTTLS, HELP
|_ Commands supported: AUTH STARTTLS HELO EHLO MAIL RCPT DATA BDAT NOOP QUIT RSET HELP
|_ssl-date: 2024-09-08T20:22:33+00:00; +6s from scanner time.
| ssl-cert: Subject: commonName=oscp.example.com/organizationName=Vesta Control Panel/stateOrProvinceName=California/countryName=US
| Not valid before: 2022-11-08T08:16:51
|_Not valid after:  2023-11-08T08:16:51
3306/tcp open  mysql    MySQL 5.7.40-0ubuntu0.18.04.1
| ssl-cert: Subject: commonName=MySQL_Server_5.7.40_Auto_Generated_Server_Certificate
| Not valid before: 2022-11-08T08:15:37
|_Not valid after:  2032-11-05T08:15:37
| mysql-info: 
|   Protocol: 10
|   Version: 5.7.40-0ubuntu0.18.04.1
|   Thread ID: 41
|   Capabilities flags: 65535
|   Some Capabilities: InteractiveClient, Speaks41ProtocolNew, SupportsCompression, IgnoreSigpipes, Speaks41ProtocolOld, IgnoreSpaceBeforeParenthesis, ODBCClient, DontAllowDatabaseTableColumn, SupportsLoadDataLocal, LongPassword, Support41Auth, SwitchToSSLAfterHandshake, FoundRows, SupportsTransactions, ConnectWithDatabase, LongColumnFlag, SupportsMultipleStatments, SupportsAuthPlugins, SupportsMultipleResults
|   Status: Autocommit
|   Salt: 6"\x19V3OtiLcsq	1 as\x1D'"
|_  Auth Plugin Name: mysql_native_password
|_ssl-date: TLS randomness does not represent time
8080/tcp open  http     Apache httpd 2.4.29 ((Ubuntu) mod_fcgid/2.3.9 OpenSSL/1.1.1)
|_http-server-header: Apache/2.4.29 (Ubuntu) mod_fcgid/2.3.9 OpenSSL/1.1.1
|_http-title: oscp.exam &mdash; Coming Soon
| http-methods: 
|_  Potentially risky methods: TRACE
8083/tcp open  http     nginx
|_http-title: Did not follow redirect to https://frankfurt.oscp.exam:8083/
8443/tcp open  http     Apache httpd 2.4.29 ((Ubuntu) mod_fcgid/2.3.9 OpenSSL/1.1.1)
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.29 (Ubuntu) mod_fcgid/2.3.9 OpenSSL/1.1.1
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.94SVN%E=4%D=9/8%OT=21%CT=1%CU=43296%PV=Y%DS=4%DC=T%G=Y%TM=66DE0
OS:786%P=x86_64-pc-linux-gnu)SEQ(SP=104%GCD=1%ISR=10C%TI=Z%II=I%TS=A)OPS(O1
OS:=M551ST11NW7%O2=M551ST11NW7%O3=M551NNT11NW7%O4=M551ST11NW7%O5=M551ST11NW
OS:7%O6=M551ST11)WIN(W1=7120%W2=7120%W3=7120%W4=7120%W5=7120%W6=7120)ECN(R=
OS:Y%DF=Y%T=40%W=7210%O=M551NNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=AS%R
OS:D=0%Q=)T2(R=N)T3(R=N)T4(R=N)T5(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q
OS:=)T6(R=N)T7(R=N)U1(R=Y%DF=N%T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=
OS:834E%RUD=G)IE(R=Y%DFI=N%T=40%CD=S)

Network Distance: 4 hops
Service Info: Host: oscp.exam; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 7s, deviation: 18s, median: 2s
```
{% endcode %}



## Foothold



## Privilege Escalation



There is no post-exploit required for standalone exam machines.
