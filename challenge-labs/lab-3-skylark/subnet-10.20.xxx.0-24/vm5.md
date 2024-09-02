---
description: Linux machine on second-level internal network
---

# VM5

## Enumeration

Output from `nmap` command run [here](./#nmap):

```
Nmap scan report for vm5.skylark (10.20.111.14)
Host is up (0.039s latency).
Not shown: 65531 filtered tcp ports (no-response)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 68:e3:f4:77:ec:9b:94:91:cb:54:ad:58:19:70:8e:b5 (RSA)
|_  256 b6:84:c4:6c:2a:91:ec:10:45:09:f7:73:62:46:5a:75 (ED25519)
80/tcp   open  http    nginx
|_http-trane-info: Problem with XML parsing of /evox/about
| http-title: Sign in \xC2\xB7 GitLab
|_Requested resource was http://vm5.skylark/users/sign_in
| http-robots.txt: 57 disallowed entries (15 shown)
| / /autocomplete/users /autocomplete/projects /search 
| /admin /profile /dashboard /users /api/v* /help /s/ /-/profile 
|_/-/ide/ /-/experiment /*/new
8060/tcp open  http    nginx 1.20.2
|_http-server-header: nginx/1.20.2
|_http-title: 404 Not Found
9094/tcp open  unknown
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
No OS matches for host
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```



## Foothold



## Privilege Escalation



## Post-Exploit

