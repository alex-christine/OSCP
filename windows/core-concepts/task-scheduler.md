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

# Task Scheduler

Windows uses its [Task Scheduler](https://learn.microsoft.com/en-us/windows/win32/taskschd/task-scheduler-reference) to execute various automated tasks, such as clean-up activities or update management.

On Windows, they are called **Scheduled Tasks**, or **Tasks**, and are defined with one or more triggers. A trigger is used as a condition, causing one or more actions to be executed when met. For example, a trigger can be set to a specific time and date, at startup, at log on, or on a Windows _event_. An action specifies which program or script to execute. There are various other possible configurations for a task, categorized in the _Conditions_, _Settings_, and _General_ menu tabs of a task's property.

## Enumerating Scheduled Tasks

### PowerShell

PowerShell's [`Get-ScheduledTask`](https://learn.microsoft.com/en-us/powershell/module/scheduledtasks/get-scheduledtask?view=windowsserver2022-ps) cmdlet can be used to list scheduled tasks on a machine:

```powershell
PS C:\Users\steve> Get-ScheduledTask

TaskPath                                       TaskName                          State
--------                                       --------                          -----
\                                              OneDrive Reporting Task-S-1-5-... Ready
\                                              OneDrive Standalone Update Tas... Ready
\Microsoft\                                    CacheCleanup                      Ready
\Microsoft\Windows\.NET Framework\             .NET Framework NGEN v4.0.30319    Ready
\Microsoft\Windows\.NET Framework\             .NET Framework NGEN v4.0.30319 64 Ready
\Microsoft\Windows\.NET Framework\             .NET Framework NGEN v4.0.30319... Disabled
\Microsoft\Windows\.NET Framework\             .NET Framework NGEN v4.0.30319... Disabled
\Microsoft\Windows\Active Directory Rights ... AD RMS Rights Policy Template ... Disabled
\Microsoft\Windows\Active Directory Rights ... AD RMS Rights Policy Template ... Ready
\Microsoft\Windows\AppID\                      PolicyConverter                   Disabled
\Microsoft\Windows\AppID\                      VerifiedPublisherCertStoreCheck   Disabled
\Microsoft\Windows\Application Experience\     Microsoft Compatibility Appraiser Running
\Microsoft\Windows\Application Experience\     PcaPatchDbTask                    Ready
\Microsoft\Windows\Application Experience\     ProgramDataUpdater                Ready
\Microsoft\Windows\Application Experience\     StartupAppTask                    Ready
...
```

The possible states listed in the last column are:

* Ready
* Disabled
* Running

### schtasks

[schtasks](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/schtasks) schedules commands and programs to run periodically or at a specific time, adds and removes tasks from the schedule, starts and stops tasks on demand, and displays and changes scheduled tasks.

Users must be a member of the Administrators group in order to make changes to tasks via schtasks, however it can be used to query existing tasks by users.

#### schtasks query

[schtasks query](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/schtasks-query) can be used to list all scheduled tasks on the current machine:

```
schtasks /query /fo LIST /v
```

* `/query`: indicates a query (technically this flag is optional)
* `/fo`: controls output format. Valid options are `TABLE`, `LIST`, or `CSV`
* `/v`: adds advanced properties of the task to output displayed

When run on a sample machine the output looks like:

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

The most "interesting" fields from an attacker's standpoint are the `Author`, `TaskName`, `Task To Run`, `Run As User`, and `Next Run Time` fields.

#### Refining schtasks Search

The output from `schtasks /query /v` can be quite overwhelming. It is often helpful to run it through PowerShell allowing the output to be searched easier (the following was inspired by the answers in [this forum post](https://community.spiceworks.com/topic/256331-get-user-info-that-scheduled-task-runs-as)):

```powershell
schtasks /query /fo CSV /v | ConvertFrom-Csv
```

* The `schtasks` output is formatted as CSV and then converted to a PowerShell object via the [`ConvertFrom-Csv`](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/convertfrom-csv?view=powershell-7.4) cmdlet

From here the output can be passed to [`Where-Object`](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/where-object?view=powershell-7.4) which can be used to select on various properties. E.g. to find all jobs that have the "Run As User" field set to "Administrator" the command would look like:

{% code overflow="wrap" %}
```powershell
schtasks /query /fo CSV /v | ConvertFrom-Csv | Where-Object "Run As User" -EQ "Administrator"
```
{% endcode %}
