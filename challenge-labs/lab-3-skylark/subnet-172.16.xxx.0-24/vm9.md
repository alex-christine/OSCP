# VM9

## Enumeration

Host at `172.16.XXX.30`. Output from nmap command run [here](./#nmap-scan):

```
Nmap scan report for vm9.skylark.com (172.16.151.30)
Host is up (0.11s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT     STATE SERVICE       VERSION
22/tcp   open  ssh           OpenSSH 7.4p1 Debian 10+deb9u7 (protocol 2.0)
| ssh-hostkey: 
|   2048 6a:a9:ed:5f:86:93:7f:3b:32:bc:4f:b4:c9:a7:69:08 (RSA)
|   256 90:f4:42:50:ad:84:51:a7:d3:63:ba:79:9a:70:f5:6e (ECDSA)
|_  256 ba:20:37:38:37:ea:09:60:5c:db:b4:a7:e5:5e:24:29 (ED25519)
3390/tcp open  ms-wbt-server xrdp
OS fingerprint not ideal because: Didn't receive UDP response. Please try again with -sSU
No OS matches for host
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```



## Foothold



## Privilege Escalation



## Post-Exploit

