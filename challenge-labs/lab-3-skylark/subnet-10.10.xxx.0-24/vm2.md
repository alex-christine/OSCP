---
description: Linux Machine
---

# VM2

## Enumeration

Machine is hosted at `10.10.XXX.10`. Output from Nmap scan run [here](./#nmap):

```
Nmap scan report for vm2.skylark (10.10.175.10)
Host is up (0.060s latency).
Not shown: 65533 filtered tcp ports (no-response)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 2d:a1:4f:70:8a:b5:2e:c4:5c:3d:63:4f:e9:fc:00:48 (RSA)
|   256 91:e1:f7:c2:3f:81:2a:4f:0d:f4:73:ed:fa:3e:7d:4a (ECDSA)
|_  256 c8:b8:b6:de:87:aa:86:8d:75:a2:6b:ac:7a:95:a3:19 (ED25519)
5901/tcp open  vnc     VNC (protocol 3.8)
| vnc-info: 
|   Protocol version: 3.8
|   Security types: 
|     VeNCrypt (19)
|     VNC Authentication (2)
|   VeNCrypt auth subtypes: 
|     Unknown security type (2)
|_    VNC auth, Anonymous TLS (258)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): IBM z/OS 2.1.X (85%)
OS CPE: cpe:/o:ibm:zos:2.1
Aggressive OS guesses: IBM z/OS 2.1 (85%)
No exact OS matches for host (test conditions non-ideal).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

