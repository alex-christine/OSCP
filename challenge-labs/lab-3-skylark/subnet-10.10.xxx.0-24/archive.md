# ARCHIVE

## Enumeration

Machine is hosted at `10.10.XXX.12`. Output from Nmap scan run [here](./#nmap):

```
Nmap scan report for ARCHIVE.skylark.com (10.10.175.12)
Host is up (0.058s latency).
Not shown: 65531 filtered tcp ports (no-response)
PORT     STATE SERVICE     VERSION
22/tcp   open  ssh         OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|_  3072 a2:20:e1:9e:43:98:72:b4:13:62:d2:df:77:ed:39:30 (RSA)
139/tcp  open  netbios-ssn Samba smbd 4.6.2
445/tcp  open  netbios-ssn Samba smbd 4.6.2
8080/tcp open  http-proxy
|_http-title: File Browser
|_http-open-proxy: Proxy might be redirecting requests
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.0 200 OK
|     Cache-Control: no-cache, no-store, must-revalidate
|     Content-Type: text/html; charset=utf-8
|     X-Xss-Protection: 1; mode=block
|     Date: Wed, 28 Aug 2024 00:57:28 GMT
|     <!DOCTYPE html><html lang="en"><head><meta charset="utf-8"><meta http-equiv="X-UA-Compatible" content="IE=edge"><meta name="viewport" content="width=device-width,initial-scale=1,user-scalable=no"><title>File Browser</title><link rel="icon" type="image/png" sizes="32x32" href="/static/img/icons/favicon-32x32.png"><link rel="icon" type="image/png" sizes="16x16" href="/static/img/icons/favicon-16x16.png"><link rel="manifest" id="manifestPlaceholder" crossorigin="use-credentials"><meta name="theme-color" content="#2979ff"><meta name="apple-mobile-web-app-capable" content="yes"><meta name="apple-mobile-web-app-status-bar-style" content="black"><meta name="apple-mobile-web-app-title" content="assets"><link rel="appl
|   GetRequest: 
|     HTTP/1.0 200 OK
|     Cache-Control: no-cache, no-store, must-revalidate
|     Content-Type: text/html; charset=utf-8
|     X-Xss-Protection: 1; mode=block
|     Date: Wed, 28 Aug 2024 00:57:27 GMT
|     <!DOCTYPE html><html lang="en"><head><meta charset="utf-8"><meta http-equiv="X-UA-Compatible" content="IE=edge"><meta name="viewport" content="width=device-width,initial-scale=1,user-scalable=no"><title>File Browser</title><link rel="icon" type="image/png" sizes="32x32" href="/static/img/icons/favicon-32x32.png"><link rel="icon" type="image/png" sizes="16x16" href="/static/img/icons/favicon-16x16.png"><link rel="manifest" id="manifestPlaceholder" crossorigin="use-credentials"><meta name="theme-color" content="#2979ff"><meta name="apple-mobile-web-app-capable" content="yes"><meta name="apple-mobile-web-app-status-bar-style" content="black"><meta name="apple-mobile-web-app-title" content="assets"><link rel="appl
|   HTTPOptions: 
|     HTTP/1.0 404 Not Found
|     Cache-Control: no-cache, no-store, must-revalidate
|     Content-Type: text/plain; charset=utf-8
|     X-Content-Type-Options: nosniff
|     Date: Wed, 28 Aug 2024 00:57:28 GMT
|     Content-Length: 14
|     Found
|   RTSPRequest: 
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|_    Request
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port8080-TCP:V=7.94SVN%I=7%D=8/27%Time=66CE75F7%P=x86_64-pc-linux-gnu%r
SF:(GetRequest,11E6,"HTTP/1\.0\x20200\x20OK\r\nCache-Control:\x20no-cache,
...
SF:me=\"apple-mobile-web-app-status-bar-style\"\x20content=\"black\"><meta
SF:\x20name=\"apple-mobile-web-app-title\"\x20content=\"assets\"><link\x20
SF:rel=\"appl");

Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): IBM z/OS 1.11.X (85%)
OS CPE: cpe:/o:ibm:zos:1.11
Aggressive OS guesses: IBM z/OS 1.11 (85%)
No exact OS matches for host (test conditions non-ideal).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

### Port 8080

The landing page on port 8080 is some sort of sign in portal. I try some basic guesses but no luck:

<figure><img src="../../../.gitbook/assets/SL-ARCHIVE-8080_Landing.png" alt=""><figcaption><p>Landing page on port 8080</p></figcaption></figure>

I move on to other machines for now.

## Foothold

Eventually I [find some credentials](../subnet-10.20.xxx.0-24/preprod.md#manual-enumeration) that reference this machine on `PREPROD` in the `10.20.XXX.0/24` subnet. I first decide to try them on port 8080 and they log me in to some sort of file server:

<figure><img src="../../../.gitbook/assets/SL-ARCHIVE-8080_LoggedIn.png" alt=""><figcaption><p>Port 8080 after successful login</p></figcaption></figure>

As I start poking around the only interesting file is `file2` which seems to contain some credentials:

<figure><img src="../../../.gitbook/assets/SL-ARCHIVE-8080_File2_Credentials.png" alt=""><figcaption><p>Potential credentials in file2</p></figcaption></figure>

I was hoping one of them would work on SSH for this machine but none did:

<figure><img src="../../../.gitbook/assets/SL-ARCHIVE-FailedSshLogins.png" alt=""><figcaption><p>Failed SSH logins with the discovered credentials</p></figcaption></figure>

Regardless I file them in `creds.txt`:

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
admin           Complex__1__Password!   Found in C:\inetpub\TODO.txt on PREPROD - Work on ARCHIVE port 8080
lance           circus5                 Found in file2 on ARCHIVE port 8080
baop_user       CSHrckxgVskAuVEwB0gZ    Found in file2 on ARCHIVE port 8080
s.ahmed         WelcomeToSkyl4rk!       Found in file2 on ARCHIVE port 8080
```
{% endcode %}

One thing of interest is `s.ahmed`'s password of `WelcomeToSkyl4rk!`. That looks like it could be a standard password for new users at the company. I should keep this in mind in case I want to do any password spraying later.
