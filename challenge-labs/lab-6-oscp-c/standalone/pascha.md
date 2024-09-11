---
description: Standalone Machine
---

# PASCHA

Writeup for standalone machine hosted at `192.168.xxx.155`.

## Enumeration

Output from Nmap scan [run earlier](./#nmap):

{% code title="standalone-tcp_stealth-all.nmap" %}
```
Nmap scan report for pascha.oscp.exam (192.168.202.155)
Host is up (0.053s latency).
Not shown: 65531 filtered tcp ports (no-response)
PORT      STATE SERVICE VERSION
80/tcp    open  http    Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: IIS Windows
9099/tcp  open  unknown
| fingerprint-strings: 
|   FourOhFourRequest, GetRequest: 
|     HTTP/1.0 200 OK 
|     Server: Mobile Mouse Server 
|     Content-Type: text/html 
|     Content-Length: 321
|_    <HTML><HEAD><TITLE>Success!</TITLE><meta name="viewport" content="width=device-width,user-scalable=no" /></HEAD><BODY BGCOLOR=#000000><br><br><p style="font:12pt arial,geneva,sans-serif; text-align:center; color:green; font-weight:bold;" >The server running on "OSCP" was able to receive your request.</p></BODY></HTML>
9999/tcp  open  abyss?
35913/tcp open  unknown
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port9099-TCP:V=7.94SVN%I=7%D=9/8%Time=66DE06C1%P=x86_64-pc-linux-gnu%r(
SF:GetRequest,1A2,"HTTP/1\.0\x20200\x20OK\x20\r\nServer:\x20Mobile\x20Mous
SF:e\x20Server\x20\r\nContent-Type:\x20text/html\x20\r\nContent-Length:\x2
SF:0321\r\n\r\n<HTML><HEAD><TITLE>Success!</TITLE><meta\x20name=\"viewport
SF:\"\x20content=\"width=device-width,user-scalable=no\"\x20/></HEAD><BODY
SF:\x20BGCOLOR=#000000><br><br><p\x20style=\"font:12pt\x20arial,geneva,san
SF:s-serif;\x20text-align:center;\x20color:green;\x20font-weight:bold;\"\x
SF:20>The\x20server\x20running\x20on\x20\"OSCP\"\x20was\x20able\x20to\x20r
SF:eceive\x20your\x20request\.</p></BODY></HTML>\r\n")%r(FourOhFourRequest
SF:,1A2,"HTTP/1\.0\x20200\x20OK\x20\r\nServer:\x20Mobile\x20Mouse\x20Serve
SF:r\x20\r\nContent-Type:\x20text/html\x20\r\nContent-Length:\x20321\r\n\r
SF:\n<HTML><HEAD><TITLE>Success!</TITLE><meta\x20name=\"viewport\"\x20cont
SF:ent=\"width=device-width,user-scalable=no\"\x20/></HEAD><BODY\x20BGCOLO
SF:R=#000000><br><br><p\x20style=\"font:12pt\x20arial,geneva,sans-serif;\x
SF:20text-align:center;\x20color:green;\x20font-weight:bold;\"\x20>The\x20
SF:server\x20running\x20on\x20\"OSCP\"\x20was\x20able\x20to\x20receive\x20
SF:your\x20request\.</p></BODY></HTML>\r\n");
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows XP (89%)
OS CPE: cpe:/o:microsoft:windows_xp::sp3
Aggressive OS guesses: Microsoft Windows XP SP3 (89%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 4 hops
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```
{% endcode %}



## Foothold



## Privilege Escalation



There is no post-exploit required for standalone exam machines.
