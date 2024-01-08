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

# Manual Enumeration

This will be an example of manually enumerating a Windows system (`CLIENTWK220`). For the purposes of the example, assume the attacker was previously able to install a bind shell on port `4444` of the machine. The shell is running as unprivileged user `dave` for the examples.

## Machine Information

### Hostname

The `whoami` command will output both the hostname and the username:

```shell
C:\Users\dave> whoami
whoami
clientwk220\dave
```

The output preceding the slash (`\`) is the hostname, and what comes after is the username.

### Operating System Information

To check the operating system, version, and architecture use the `systeminfo` command in either cmd.exe or PowerShell:

```powershell
PS C:\Users\dave> systeminfo
systeminfo

Host Name:                 CLIENTWK220
OS Name:                   Microsoft Windows 11 Pro
OS Version:                10.0.22000 N/A Build 22000
OS Manufacturer:           Microsoft Corporation
OS Configuration:          Standalone Workstation
OS Build Type:             Multiprocessor Free
Registered Owner:          offsec
Registered Organization:   
Product ID:                00331-10000-00001-AA142
Original Install Date:     6/15/2022, 10:48:14 AM
System Boot Time:          3/2/2023, 3:34:17 AM
System Manufacturer:       VMware, Inc.
System Model:              VMware7,1
System Type:               x64-based PC
Processor(s):              1 Processor(s) Installed.
                           [01]: AMD64 Family 25 Model 1 Stepping 1 AuthenticAMD ~2650 Mhz
BIOS Version:              VMware, Inc. VMW71.00V.18227214.B64.2106252220, 6/25/2021
Windows Directory:         C:\Windows
System Directory:          C:\Windows\system32
Boot Device:               \Device\HarddiskVolume1
System Locale:             en-us;English (United States)
Input Locale:              en-us;English (United States)
Time Zone:                 (UTC-08:00) Pacific Time (US & Canada)
Total Physical Memory:     4,095 MB
Available Physical Memory: 2,764 MB
Virtual Memory: Max Size:  4,799 MB
Virtual Memory: Available: 3,438 MB
Virtual Memory: In Use:    1,361 MB
Page File Location(s):     C:\pagefile.sys
Domain:                    WORKGROUP
Logon Server:              N/A
Hotfix(s):                 N/A
Network Card(s):           1 NIC(s) Installed.
                           [01]: vmxnet3 Ethernet Adapter
                                 Connection Name: Ethernet0
                                 DHCP Enabled:    No
                                 IP address(es)
                                 [01]: 192.168.236.220
                                 [02]: fe80::dc1a:de70:4134:c1f7
Hyper-V Requirements:      A hypervisor has been detected. Features required for Hyper-V will not be displayed.
```

### Architecture Information

To determine the architecture of a system, 32 vs. 64-bit, the [WMIC](https://learn.microsoft.com/en-us/windows/win32/wmisdk/wmic) command-line utility can be used (via either `cmd.exe` or PowerShell):

```powershell
wmic OS get OSArchitecture
```

When run on a system the output will be either `32-bit` or `64-bit`:

```powershell
PS C:\Users\student> wmic OS get OSArchitecture
OSArchitecture
32-bit
```

### Registry

The [Windows registry](https://learn.microsoft.com/en-us/troubleshoot/windows-server/performance/windows-registry-advanced-users) is a central hierarchical database used to store information that is necessary to configure the system for one or more users, applications, and hardware devices.

It contains information that Windows continually references during operation, such as profiles for each user, the applications installed on the computer and the types of documents that each can create, property sheet settings for folders and application icons, what hardware exists on the system, and the ports that are being used.

A registry hive is a group of keys, subkeys, and values in the registry that has a set of supporting files that contain backups of its data. The supporting files for all hives except `HKEY_CURRENT_USER` are in the `%SystemRoot%\System32\Config`.

Administrators can modify the registry by using Registry Editor (`Regedit.exe` or `Regedt32.exe`), Group Policy, System Policy, Registry (`.reg`) files, or by running scripts such as VisualBasic script files.

The registry can be a source of useful information during enumeration. It will be touched on in later sections to enumerate specific pieces of the system.

### Username

As described [above](manual-enumeration.md#hostname), the username can be obtained with the `whoami` command.

### Group Membership

Running `whoami` with the `/groups` flag will enumerate all groups of which the current user is a member:

```sh
C:\Users\dave> whoami /groups
 whoami /groups

GROUP INFORMATION
-----------------

Group Name                           Type             SID                                            Attributes                                        
==================================== ================ ============================================== ==================================================
Everyone                             Well-known group S-1-1-0                                        Mandatory group, Enabled by default, Enabled group
CLIENTWK220\helpdesk                 Alias            S-1-5-21-2309961351-4093026482-2223492918-1008 Mandatory group, Enabled by default, Enabled group
BUILTIN\Remote Desktop Users         Alias            S-1-5-32-555                                   Mandatory group, Enabled by default, Enabled group
BUILTIN\Users                        Alias            S-1-5-32-545                                   Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\BATCH                   Well-known group S-1-5-3                                        Mandatory group, Enabled by default, Enabled group
CONSOLE LOGON                        Well-known group S-1-2-1                                        Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users     Well-known group S-1-5-11                                       Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization       Well-known group S-1-5-15                                       Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Local account           Well-known group S-1-5-113                                      Mandatory group, Enabled by default, Enabled group
LOCAL                                Well-known group S-1-2-0                                        Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NTLM Authentication     Well-known group S-1-5-64-10                                    Mandatory group, Enabled by default, Enabled group
Mandatory Label\High Mandatory Level Label            S-1-16-12288                                                       
```

It is also possible to query what groups others are members of via the `net user` command (cmd or PowerShell). E.g. to see what groups the user `steve` is a member of the command would be:

```powershell
PS C:\Users\dave> net user steve

User name                    steve
Full Name                    steve
Comment                      
User's comment               
Country/region code          000 (System Default)
Account active               Yes
Account expires              Never

Password last set            6/16/2022 12:08:00 PM
Password expires             Never
Password changeable          6/16/2022 12:08:00 PM
Password required            Yes
User may change password     Yes

Workstations allowed         All
Logon script                 
User profile                 
Home directory               
Last logon                   2/13/2023 3:53:29 AM

Logon hours allowed          All

Local Group Memberships      *helpdesk             *Remote Desktop Users 
                             *Remote Management Use*Users                
Global Group memberships     *None                 
The command completed successfully.
```

### Privileges

Windows [privileges](https://learn.microsoft.com/en-us/windows/win32/secauthz/privileges) are the rights of an account to perform various system-related operations on the local computer, such as shutting down the system, loading device drivers, or changing the system time. Privileges differ from access rights in two ways:

1. Privileges control access to system resources and system-related tasks, whereas access rights control access to [securable objects](https://learn.microsoft.com/en-us/windows/win32/secauthz/securable-objects).
2. A system administrator assigns privileges to user and group accounts, whereas the system grants or denies access to a securable object based on the access rights granted in the ACEs in the object's DACL.

Each system has an account database that stores the privileges held by user and group accounts. When a user logs on, the system produces an [access token](https://learn.microsoft.com/en-us/windows/win32/secauthz/access-tokens) that contains a list of the user's privileges, including those granted to the user or to groups to which the user belongs.

In order to enumerate the current user's privileges run whoami with the /priv flag:

```
whoami /priv
```

Output from this command looks like:

```powershell
PS C:\Users\dave> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                               State   
============================= ========================================= ========
SeSecurityPrivilege           Manage auditing and security log          Disabled
SeShutdownPrivilege           Shut down the system                      Disabled
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled 
SeUndockPrivilege             Remove computer from docking station      Disabled
SeImpersonatePrivilege        Impersonate a client after authentication Enabled 
SeIncreaseWorkingSetPrivilege Increase a process working set            Disabled
SeTimeZonePrivilege           Change the time zone                      Disabled
```

The `State` column in the output can be easily misinterpreted. For example, at a glance it appears the current user does not have permission to shutdown the system (`SeShutdownPrivilege`) due to the state being `Disabled`. However, the `Disabled` state only indicates if the privilege is currently enabled for the running process. In this case, it means that `whoami` has not requested and is not currently using the `SeShutdownPrivilege` privilege. A privilege's presence in the list indicates it is available to the current user regardless of `State`.

#### Important Privileges

Non-privileged users with assigned privileges can potentially abuse those privileges to perform privilege escalation attacks. Privileges that may lead to escalation include:

* [`SeImpersonatePrivilege`](https://learn.microsoft.com/en-us/troubleshoot/windows-server/windows-security/seimpersonateprivilege-secreateglobalprivilege)
* [`SeBackupPrivilege`](https://learn.microsoft.com/en-us/windows-hardware/drivers/ifs/privileges)
* [`SeAssignPrimaryToken`](https://book.hacktricks.xyz/windows-hardening/windows-local-privilege-escalation/privilege-escalation-abusing-tokens)
* [`SeLoadDriver`](https://book.hacktricks.xyz/windows-hardening/windows-local-privilege-escalation/privilege-escalation-abusing-tokens/abuse-seloaddriverprivilege)
* [`SeDebug`](https://book.hacktricks.xyz/windows-hardening/windows-local-privilege-escalation/sedebug-+-seimpersonate-copy-token)

### Local Users

If on the DOS command prompt (`cmd.exe`) it is possible to enumerate all users with the `net user` command:

```
C:\Users\dave> net user

User accounts for \\CLIENTWK220

-------------------------------------------------------------------------------
Administrator            BackupAdmin              dave                     
daveadmin                DefaultAccount           Guest                    
offsec                   steve                    WDAGUtilityAccount       
The command completed successfully.
```

Alternatively, if PowerShell is available, the [Get-LocalUser](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.localaccounts/get-localuser?view=powershell-5.1) cmdlet will provide slightly more detail:

```powershell
PS C:\Users\dave> Get-LocalUser

Name               Enabled Description                                                                                 
----               ------- -----------                                                                                 
Administrator      False   Built-in account for administering the computer/domain                                      
BackupAdmin        True                                                                                                
dave               True    dave                                                                                        
daveadmin          True                                                                                                
DefaultAccount     False   A user account managed by the system.                                                       
Guest              False   Built-in account for guest access to the computer/domain                                    
offsec             True                                                                                                
steve              True                                                                                                
WDAGUtilityAccount False   A user account managed and used by the system for Windows Defender Application Guard scen...
```

In this instance it is interesting to note that the normal built-in `Administrator` account is disabled. However there is a `daveadmin` account that is likely the privileged account for the `dave` user. Additionally `BackupAdmin`'s name seems to indicate it may also have elevated privileges.

### Local Groups

Similarly to Local Users there is both a cmd.exe and PowerShell command to get a list of groups on the local machine.

For cmd use the `net localgroup` command:

```
C:\Users\dave> net localgroup

Aliases for \\CLIENTWK220

-------------------------------------------------------------------------------
*Access Control Assistance Operators
*Administrators
*adminteam
*Backup Operators
*BackupUsers
*Cryptographic Operators
*Device Owners
*Distributed COM Users
*Event Log Readers
*Guests
*helpdesk
*Hyper-V Administrators
*IIS_IUSRS
*Network Configuration Operators
*Performance Log Users
*Performance Monitor Users
*Power Users
*Remote Desktop Users
*Remote Management Users
*Replicator
*System Managed Accounts Group
*Users
The command completed successfully.
```

For PowerShell, use the [Get-LocalGroup](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.localaccounts/get-localgroup?view=powershell-5.1) cmdlet:

```powershell
PS C:\Users\dave> Get-LocalGroup

Name                                Description                                                                        
----                                -----------                                                                        
adminteam                           Members of this group are admins to all workstations on the second floor           
BackupUsers                                                                                                            
helpdesk                                                                                                               
Access Control Assistance Operators Members of this group can remotely query authorization attributes and permission...
Administrators                      Administrators have complete and unrestricted access to the computer/domain        
Backup Operators                    Backup Operators can override security restrictions for the sole purpose of back...
Cryptographic Operators             Members are authorized to perform cryptographic operations.                        
Device Owners                       Members of this group can change system-wide settings.                             
Distributed COM Users               Members are allowed to launch, activate and use Distributed COM objects on this ...
Event Log Readers                   Members of this group can read event logs from local machine                       
Guests                              Guests have the same access as members of the Users group by default, except for...
Hyper-V Administrators              Members of this group have complete and unrestricted access to all features of H...
IIS_IUSRS                           Built-in group used by Internet Information Services.                              
Network Configuration Operators     Members in this group can have some administrative privileges to manage configur...
Performance Log Users               Members of this group may schedule logging of performance counters, enable trace...
Performance Monitor Users           Members of this group can access performance counter data locally and remotely     
Power Users                         Power Users are included for backwards compatibility and possess limited adminis...
Remote Desktop Users                Members in this group are granted the right to logon remotely                      
Remote Management Users             Members of this group can access WMI resources over management protocols (such a...
Replicator                          Supports file replication in a domain                                              
System Managed Accounts Group       Members of this group are managed by the system.                                   
Users                               Users are prevented from making accidental or intentional system-wide changes an...
```

Some non-standard groups stand out in the initial analysis. The `adminteam` group may be of interest to attackers since it contains the string "admin" and has a custom description text, indicating the members are admins to all workstations on the second floor. However at this point this is just an interesting tidbit as the attacker has no idea of what else exists on floor 2 or its value.

Another interesting finding is the group name `BackupUsers` as with the user `BackupAdmin` before. Backup solutions often have extensive permissions to perform their operations on the file system, making both potentially valuable targets.

Beyond the non-standard groups there are several built-in groups that could be valuable, such as `Administrators`, `Backup Operators`, `Remote Desktop Users`, and `Remote Management Users`.

Members of `Backup Operators` can backup and restore all files on a computer, even those files they don't have permissions for. This is not to be confused with the non-standard `BackupUsers` group found in this particular example.

Members of `Remote Desktop Users` can access the system with RDP, while members of `Remote Management Users` can access it with `WinRM`.

A complete list of the built-in AD groups can be found [here](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/understand-security-groups#default-active-directory-security-groups).

### Users in Group

It is possible to list the users in a particular group via PowerShell's [Get-LocalGroupMember](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.localaccounts/get-localgroupmember?view=powershell-5.1) cmdlet:

```powershell
PS C:\Users\dave> Get-LocalGroupMember adminteam

ObjectClass Name                  PrincipalSource
----------- ----                  ---------------
User        CLIENTWK220\daveadmin Local

PS C:\Users\dave> Get-LocalGroupMember -Group "Administrators"

ObjectClass Name                      PrincipalSource
----------- ----                      ---------------
User        CLIENTWK220\Administrator Local          
User        CLIENTWK220\BackupAdmin   Local          
User        CLIENTWK220\daveadmin     Local          
User        CLIENTWK220\offsec        Local
```

In this example, the members of the `adminteam` group (found [above](manual-enumeration.md#local-groups)) are determined to only be the `daveadmin` user. The `Administrators` group contains several members including `daveadmin`.

### Installed Applications

Two [registry](manual-enumeration.md#registry) keys can be queried via PowerShell to list installed applications. The keys to query are the [uninstall registry keys](https://learn.microsoft.com/en-us/windows/win32/msi/uninstall-registry-key?redirectedfrom=MSDN):

1. `HKLM:\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*`
2. `HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*`

1 will generate a list of all installed 32-bit applications and 2 will generate a list of 64-bit applications. Both can be queried via the [Get-ItemProperty](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.management/get-itemproperty?view=powershell-7.3) cmdlet:

```powershell
Get-ItemProperty HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\*
```

This will result in a full list of all values under the uninstall registry key for each application. The properties for a single installed application are listed in the documentation linked above. An example is provided here for 7Zip from the target:

```powershell
PS C:\Users\dave> Get-ItemProperty HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*

DisplayName          : 7-Zip 21.07 (x64)
DisplayVersion       : 21.07
DisplayIcon          : C:\Program Files\7-Zip\7zFM.exe
InstallLocation      : C:\Program Files\7-Zip\
UninstallString      : "C:\Program Files\7-Zip\Uninstall.exe"
QuietUninstallString : "C:\Program Files\7-Zip\Uninstall.exe" /S
NoModify             : 1
NoRepair             : 1
EstimatedSize        : 5443
VersionMajor         : 21
VersionMinor         : 7
Publisher            : Igor Pavlov
PSPath               : Microsoft.PowerShell.Core\Registry::HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\7-Zip
PSParentPath         : Microsoft.PowerShell.Core\Registry::HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall
PSChildName          : 7-Zip
PSDrive              : HKLM
PSProvider           : Microsoft.PowerShell.Core\Registry
```

The output can then be piped to the [Select-Object](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/select-object?view=powershell-7.3) cmdlet to obtain certain properties of the object. In this case the name, version, publisher, and installation date:

{% code overflow="wrap" %}
```powershell
Get-ItemProperty HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\* |  Select-Object DisplayName, DisplayVersion, Publisher, InstallDate
```
{% endcode %}

For additional display cleaning, this could be piped to the [Format-Table](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/format-table?view=powershell-7.3) cmdlet with the `-AutoSize` flag enabled.

#### 32-Bit Applications

The full one-line command to display all 32-bit applications:

{% code overflow="wrap" %}
```powershell
Get-ItemProperty HKLM:\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\* | Select-Object DisplayName, DisplayVersion, Publisher, InstallDate | Format-Table -AutoSize
```
{% endcode %}

#### 64-Bit Applications

The full one-line command to display all 64-bit applications:

{% code overflow="wrap" %}
```powershell
Get-ItemProperty HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\* |  Select-Object DisplayName, DisplayVersion, Publisher, InstallDate | Format-Table -AutoSize
```
{% endcode %}

#### Example Output

Running these commands on the example target generates the following lists:

```powershell
PS C:\Users\dave> Get-ItemProperty HKLM:\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\* | Select-Object DisplayName, DisplayVersion, Publisher, InstallDate | Format-Table -AutoSize

DisplayName                                                        DisplayVersion Publisher             InstallDate
-----------                                                        -------------- ---------             -----------
                                                                                                                   
KeePass Password Safe 2.51.1                                       2.51.1         Dominik Reichl        20220616   
Microsoft Edge                                                     110.0.1587.57  Microsoft Corporation 20230301   
Microsoft Edge Update                                              1.3.173.45                                      
Microsoft Edge WebView2 Runtime                                    109.0.1518.78  Microsoft Corporation 20230210   
                                                                                                                   
Microsoft Visual C++ 2015-2019 Redistributable (x86) - 14.28.29913 14.28.29913.0  Microsoft Corporation            
Microsoft Visual C++ 2019 X86 Additional Runtime - 14.28.29913     14.28.29913    Microsoft Corporation 20220615   
Microsoft Visual C++ 2019 X86 Minimum Runtime - 14.28.29913        14.28.29913    Microsoft Corporation 20220615   
Microsoft Visual C++ 2015-2019 Redistributable (x64) - 14.28.29913 14.28.29913.0  Microsoft Corporation            


PS C:\Users\dave> Get-ItemProperty HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\* |  Select-Object DisplayName, DisplayVersion, Publisher, InstallDate | Format-Table -AutoSize

DisplayName                                                    DisplayVersion  Publisher             InstallDate
-----------                                                    --------------  ---------             -----------
7-Zip 21.07 (x64)                                              21.07           Igor Pavlov                      
                                                                                                                
                                                                                                                
                                                                                                                
XAMPP                                                          7.4.29-1        Bitnami               20220616   
VMware Tools                                                   11.3.0.18090558 VMware, Inc.          20220615   
Microsoft Visual C++ 2019 X64 Additional Runtime - 14.28.29913 14.28.29913     Microsoft Corporation 20220615   
Microsoft Update Health Tools                                  4.67.0.0        Microsoft Corporation 20220720   
Microsoft Visual C++ 2019 X64 Minimum Runtime - 14.28.29913    14.28.29913     Microsoft Corporation 20220615 
```

Several bits are interesting here. XAMPP is present indicating the machine may be hosting a web server. KeePass was also found indicating the software used by this user (potentially the whole organization) for storing credentials.

#### Additional Checks

The list of applications from above may not be complete. For example, this could be due to an incomplete or flawed installation process. Therefore, one should always check 32-bit and 64-bit `Program Files` directories located in `C:\`. Additionally, one should review the contents of the `Downloads` directory of our user to find more potential programs.

* `C:\Program Files` contains 32-bit applications
* `C:\Program Files (x86)` contains 64-bit applications
* `C:\Users\<user>\Downloads` is the default download directory for any given `<user>`

Folders can be searched with the [Get-ChildItem](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.management/get-childitem?view=powershell-7.3) cmdlet:

```powershell
Get-ChildItem -Path "C:\Program Files" -Force -ErrorAction SilentlyContinue
```

* `-Path` tells the cmdlet where to start the search
* `-ErrorAction SilentlyContinue` is functionally equivalent to `2>/dev/null` on Linux in that it prevents error output from appearing or interrupting execution
* `-Force` allows the cmdlet to get items that otherwise can't be accessed by the user, such as hidden or system files (equivalent to `ls -a` on Linux)
* If needed, the `-Recurse` flag can be used to tell the cmdlet to recursively enter each directory found under `-Path`

On the example machine enumerating these additional directories provides the following output:

```powershell
PS C:\Users\dave> Get-ChildItem -Path "C:\Program Files" -Force -ErrorAction SilentlyContinue


    Directory: C:\Program Files


Mode                 LastWriteTime         Length Name                                                                 
----                 -------------         ------ ----                                                                 
d-----         6/16/2022   9:01 AM                7-Zip                                                                
d-----         6/15/2022   1:19 PM                Common Files                                                         
d-----          7/4/2022   7:05 AM                Enterprise Apps                                                      
d-----         7/20/2022   2:21 AM                Internet Explorer                                                    
d-----         6/16/2022   9:05 AM                KeePass Password Safe 2                                              
d-----         7/20/2022   1:14 AM                Microsoft Update Health Tools                                        
d-----          6/5/2021   5:10 AM                ModifiableWindowsApps                                                
d-----         6/15/2022   1:19 PM                VMware                                                               
d-----         7/20/2022   2:21 AM                Windows Defender                                                     
d-----         7/20/2022   2:21 AM                Windows Defender Advanced Threat Protection                          
d-----         7/20/2022   2:21 AM                Windows Mail                                                         
d-----         7/20/2022   2:21 AM                Windows Media Player                                                 
d-----          6/5/2021   7:21 AM                Windows NT                                                           
d-----         7/20/2022   2:21 AM                Windows Photo Viewer                                                 
d-----          6/5/2021   5:10 AM                WindowsPowerShell 
```

In this case, there were some additional programs found though their usefulness is still to be determined.

### Running Processes

Running processes can be enumerated using the [Get-Process](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.management/get-process?view=powershell-7.3) cmdlet. Without parameters, this cmdlet gets all of the processes on the local computer. One can also specify a particular process by process name or process ID (PID) or pass a process object through the pipeline to this cmdlet.

```powershell
PS C:\Users\dave> Get-Process

Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName                                                  
-------  ------    -----      -----     ------     --  -- -----------                                                  
     53      13      536       1552       0.00   2668   0 access                                                       
     91       6      992       5376              3592   0 AggregatorHost                                               
    191      12    12144      12796              2620   0 BetaServ                                                     
     76       6     2400        744       0.00   2068   0 cmd                                                          
    ...                                                 
    183      29     9520      19788              2608   0 httpd                                                        
    481      49    16424      23020              3664   0 httpd                                                        
    ...                                                      
    177      16   210492      28704              2832   0 mysqld                                                       
    631      27   114964      14112       1.20   5068   0 powershell                                                   
      0       7     3396      69860               100   0 Registry                                                     
    ...                                                      
    148       9     1744      22176              6264   0 svchost                                                      
    116       8     1232      13772              6996   0 svchost                                                      
    277      17     7896      50364              7100   0 svchost                                                      
   1874       0       36        140                 4   0 System                                                       
    ...                                                     
    215      17     5612      17116              6232   0 xampp-control 
```

The same cmdlet can also be used to get more detail on a singular process:

```powershell
Get-Process xampp-control
```

A process can also be specified by PID.

The output can be formatted cleanly with the [Format-List](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/format-list?view=powershell-7.3) cmdlet:

```powershell
PS C:\Users\dave> Get-Process xampp-control | Format-List

Id      : 6232
Handles : 215
CPU     : 
SI      : 0
Name    : xampp-control
```

### Services

Enumeration of services is covered in the [Windows Services section](../../core-concepts/windows-services.md#service-enumeration).

### Scheduled Tasks

Enumeration of scheduled tasks is covered in the [Task Scheduler section](../../core-concepts/task-scheduler.md#enumerating-scheduled-tasks).

### Sensitive Files

The [Get-ChildItem](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.management/get-childitem?view=powershell-7.3) cmdlet has been mentioned before, but it can also be used to search for sensitive files that are not adequately protected.

#### Search By File Types

The `-Include` flag can be used with a wildcard and extension to search for all files of a particular type.

For example, [earlier](manual-enumeration.md#installed-applications) it was determined that the machine has KeePass installed to manage passwords. Research indicates that the application stores its blob as a `.kdbx` file. In order to search the machine for that file type:

{% code overflow="wrap" %}
```powershell
Get-ChildItem -Path "C:\" -Include "*.kdbx" -Recurse -File -ErrorAction SilentlyContinue
```
{% endcode %}

Unfortunately, the target does not contain any exposed database files but it was worth checking.

A second useful check could be to look for password or config files in the XAMPP directory given that it was also found on the machine in an [earlier section](manual-enumeration.md#installed-applications). The passwords would be stored in `.txt` files and XAMPP config files end in `.ini`. Therefore the command to search for them would be:

{% code overflow="wrap" %}
```powershell
Get-ChildItem -Path "C:\XAMPP" -Include "*.txt","*.ini" -Recurse -File -ErrorAction SilentlyContinue
```
{% endcode %}

Which does find some useful output on the target machine:

```powershell
PS C:\Users\dave> Get-ChildItem -Path C:\xampp -Include *.txt,*.ini -File -Recurse -ErrorAction SilentlyContinue

...
Directory: C:\xampp\mysql\bin

Mode                 LastWriteTime         Length Name                                               
----                 -------------         ------ ----                                               
-a----         6/16/2022   1:42 PM           5786 my.ini
...
Directory: C:\xampp

Mode                 LastWriteTime         Length Name                                              
----                 -------------         ------ ----                                                                 
-a----         3/13/2017   4:04 AM            824 passwords.txt
-a----         6/16/2022  10:22 AM            792 properties.ini     
-a----         5/16/2022  12:21 AM           7498 readme_de.txt 
-a----         5/16/2022  12:21 AM           7368 readme_en.txt     
-a----         6/16/2022   1:17 PM           1200 xampp-control.ini 
```

These are simplistic examples but it shows how `Get-ChildItem` can be used to search for files and directories on a target.

### Folder Access

Often techniques will depend on whether a particular user has access to a particular folder. In order to check, the [`Get-Acl`](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.security/get-acl?view=powershell-7.4) cmdlet can be used with the `-Path` flag (techincally the flag is optional and just calling `Get-Acl` with a path will accomplish the same thing):

```powershell
PS C:\Users\steve> Get-Acl "C:\Users\steve"


    Directory: C:\Users


Path  Owner                  Access
----  -----                  ------
steve BUILTIN\Administrators NT AUTHORITY\SYSTEM Allow  FullControl...
```

This output is not especially helpful. A slightly more helpful move is to use the cmdlet and request the `.Access` property:

```powershell
(Get-Acl $Folder).Access
```

* `$Folder` shoud be a string that contains the path to the folder in question (it can also just be replaced with a string in quotations)

```powershell
PS C:\Users\steve> $Folder = "C:\Users\steve"
PS C:\Users\steve> (Get-Acl $Folder).Access


FileSystemRights  : FullControl
AccessControlType : Allow
IdentityReference : NT AUTHORITY\SYSTEM
IsInherited       : False
InheritanceFlags  : ContainerInherit, ObjectInherit
PropagationFlags  : None

FileSystemRights  : FullControl
AccessControlType : Allow
IdentityReference : BUILTIN\Administrators
IsInherited       : False
InheritanceFlags  : ContainerInherit, ObjectInherit
PropagationFlags  : None

FileSystemRights  : FullControl
AccessControlType : Allow
IdentityReference : CLIENTWK220\offsec
IsInherited       : False
InheritanceFlags  : ContainerInherit, ObjectInherit
PropagationFlags  : None

FileSystemRights  : FullControl
AccessControlType : Allow
IdentityReference : CLIENTWK220\steve
IsInherited       : False
InheritanceFlags  : ContainerInherit, ObjectInherit
PropagationFlags  : None
```

* This also could have been called `(Get-Acl "C:\Users\steve").Access` with no variable used

The output above shows each group on the machine and what type of access they have to the folder in question. This can be cross-referenced with `whoami /groups` or `net localgroup <groupname>` outputs.

#### Checking a Specific User and Folder

While the above output is helpful, it can be tedious to cross-reference outputs of multiple commands. Instead the following little script ([source](https://community.spiceworks.com/topic/1982988-powershell-command-to-check-if-user-has-permissions-to-a-folder)) can be used to check a specific `$Folder` and `$User` combination:

{% code overflow="wrap" %}
```powershell
(Get-Acl $Folder).Access | ?{$_.IdentityReference -match $User} | Select IdentityReference,FileSystemRights
```
{% endcode %}

Running this on an example machine:

```powershell
PS C:\Users\steve> $Folder = "C:\Users\steve"
PS C:\Users\steve> $User = "steve"
PS C:\Users\steve> (Get-Acl $Folder).Access | ?{$_.IdentityReference -match $User} | Select IdentityReference,FileSystemRights

IdentityReference FileSystemRights
----------------- ----------------
CLIENTWK220\steve      FullControl
```

If the `$User` in question does not have access to the folder, nothing is returned:

```powershell
PS C:\Users\steve> $User = "dave"
PS C:\Users\steve> (Get-Acl $Folder).Access | ?{$_.IdentityReference -match $User} | Select IdentityReference,FileSystemRights
PS C:\Users\steve>
```

### icacls

[icacls](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/icacls) displays or modifies discretionary access control lists (DACLs) on specified files, and applies stored DACLs to files in specified directories. It is usable in both PowerShell and Windows Command Line.

It displays permissions masks for a file or directory. The most common permissions masks are listed below. A more complete list can be found in the Windows documentation linked above. A refresher on file/folder permissions can be found [here](https://www.2brightsparks.com/resources/articles/a-basic-introduction-to-ntfs-permissions.html).

<table><thead><tr><th width="114">Mask</th><th>Name</th></tr></thead><tbody><tr><td><code>F</code></td><td>Full access</td></tr><tr><td><code>M</code></td><td>Modify access</td></tr><tr><td><code>RX</code></td><td>Read and execute access</td></tr><tr><td><code>R</code></td><td>Read-only access</td></tr><tr><td><code>W</code></td><td>Write-only access</td></tr></tbody></table>

`icacls` is called with the following command structure:

```powershell
icacls $Item
```

* `$Item` is a path (string) to the file or directory in question

The output from the command looks something like this

```powershell
PS C:\Users\alex> icacls "C:\Services\"
C:\Services\ BUILTIN\Administrators:(I)(OI)(CI)(F)
             NT AUTHORITY\SYSTEM:(I)(OI)(CI)(F)
             BUILTIN\Users:(I)(OI)(CI)(RX)
             NT AUTHORITY\Authenticated Users:(I)(M)
             NT AUTHORITY\Authenticated Users:(I)(OI)(CI)(IO)(M)
```

This utility can come in handy for checking permissions on service binaries, DLLs, or just folders.

## Network Information

### Interfaces

To list all network interfaces use the `ipconfig` command with the `/all` flag (cmd.exe or PowerShell):

```powershell
PS C:\Users\dave> ipconfig /all

Windows IP Configuration

   Host Name . . . . . . . . . . . . : clientwk220
   Primary Dns Suffix  . . . . . . . : 
   Node Type . . . . . . . . . . . . : Hybrid
   IP Routing Enabled. . . . . . . . : No
   WINS Proxy Enabled. . . . . . . . : No

Ethernet adapter Ethernet0:

   Connection-specific DNS Suffix  . : 
   Description . . . . . . . . . . . : vmxnet3 Ethernet Adapter
   Physical Address. . . . . . . . . : 00-50-56-86-01-3C
   DHCP Enabled. . . . . . . . . . . : No
   Autoconfiguration Enabled . . . . : Yes
   Link-local IPv6 Address . . . . . : fe80::dc1a:de70:4134:c1f7%6(Preferred) 
   IPv4 Address. . . . . . . . . . . : 192.168.236.220(Preferred) 
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . : 192.168.236.254
   DHCPv6 IAID . . . . . . . . . . . : 234901590
   DHCPv6 Client DUID. . . . . . . . : 00-01-00-01-2B-92-42-1C-00-50-56-86-66-4B
   DNS Servers . . . . . . . . . . . : 192.168.236.254
   NetBIOS over Tcpip. . . . . . . . : Enabled
```

In this case it is worth noting that the client was not configured to receive an IP via DHCP, rather one is manually assigned. It also contains the IP of the DNS server, the subnet mask, gateway, and MAC address.

### Routing Table

The `route print` command (cmd.exe or PowerShell) can be used to display the routing table for the machine. This can be helpful in understanding the topography of the network when moving to other machines.

```powershell
PS C:\Users\dave> route print

===========================================================================
Interface List
  6...00 50 56 86 01 3c ......vmxnet3 Ethernet Adapter
  1...........................Software Loopback Interface 1
===========================================================================

IPv4 Route Table
===========================================================================
Active Routes:
Network Destination        Netmask          Gateway       Interface  Metric
          0.0.0.0          0.0.0.0  192.168.236.254  192.168.236.220     16
        127.0.0.0        255.0.0.0         On-link         127.0.0.1    331
        127.0.0.1  255.255.255.255         On-link         127.0.0.1    331
  127.255.255.255  255.255.255.255         On-link         127.0.0.1    331
    192.168.236.0    255.255.255.0         On-link   192.168.236.220    271
  192.168.236.220  255.255.255.255         On-link   192.168.236.220    271
  192.168.236.255  255.255.255.255         On-link   192.168.236.220    271
        224.0.0.0        240.0.0.0         On-link         127.0.0.1    331
        224.0.0.0        240.0.0.0         On-link   192.168.236.220    271
  255.255.255.255  255.255.255.255         On-link         127.0.0.1    331
  255.255.255.255  255.255.255.255         On-link   192.168.236.220    271
===========================================================================
Persistent Routes:
  Network Address          Netmask  Gateway Address  Metric
          0.0.0.0          0.0.0.0  192.168.236.254       1
===========================================================================

IPv6 Route Table
===========================================================================
Active Routes:
 If Metric Network Destination      Gateway
  1    331 ::1/128                  On-link
  6    271 fe80::/64                On-link
  6    271 fe80::dc1a:de70:4134:c1f7/128
                                    On-link
  1    331 ff00::/8                 On-link
  6    271 ff00::/8                 On-link
===========================================================================
Persistent Routes:
  None
```

In this case, no unknown networks were discovered but it is always worth checking the routing table on a machine.

### Active Connections

To see connections to the active machine [netstat](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/netstat) can be used via cmd or PowerShell:

```
PS C:\Users\dave> netstat -ano
netstat -ano

Active Connections

  Proto  Local Address          Foreign Address        State           PID
  TCP    0.0.0.0:80             0.0.0.0:0              LISTENING       2608
  TCP    0.0.0.0:135            0.0.0.0:0              LISTENING       932
  TCP    0.0.0.0:443            0.0.0.0:0              LISTENING       2608
  TCP    0.0.0.0:3306           0.0.0.0:0              LISTENING       2832
  TCP    0.0.0.0:3389           0.0.0.0:0              LISTENING       696
  ...
  TCP    192.168.236.220:139    0.0.0.0:0              LISTENING       4
  TCP    192.168.50.220:3389    192.168.119.4:33060    ESTABLISHED     696
  TCP    192.168.236.220:4444   192.168.45.205:42768   ESTABLISHED     2548
  TCP    192.168.236.220:54364  72.21.81.240:80        SYN_SENT        2644
  TCP    192.168.236.220:54365  40.68.123.157:443      SYN_SENT        3772
  TCP    [::]:80                [::]:0                 LISTENING       2608
  TCP    [::]:135               [::]:0                 LISTENING       932
  TCP    [::]:443               [::]:0                 LISTENING       2608
  TCP    [::]:3306              [::]:0                 LISTENING       2832
  TCP    [::]:3389              [::]:0                 LISTENING       696
  ...
  UDP    0.0.0.0:123            *:*                                    2908
  UDP    0.0.0.0:500            *:*                                    2716
  ...
  UDP    127.0.0.1:1900         *:*                                    5568
  UDP    127.0.0.1:56630        *:*                                    5568
  UDP    127.0.0.1:65291        127.0.0.1:65291                        2756
  UDP    192.168.236.220:137    *:*                                    4
  UDP    192.168.236.220:138    *:*                                    4
  UDP    192.168.236.220:1900   *:*                                    5568
  UDP    192.168.236.220:56629  *:*                                    5568
  UDP    [::]:123               *:*                                    2908
  UDP    [::]:500               *:*                                    2716
  ...
```

* `-a` lists all TCP connections as well as the TCP and UDP ports on which the machine is listening
* `-n` displays connections with numeric IPs with no attempt to resolve to a name
* `-o` includes the PID for each connection

The output shows the machine is listening on ports `80` and `443` indicating a web server is likely running here. Additionally the open port of `3306` likely indicates a MySQL server instance.

There are also some active connections visible. One is the connection from the attacking machine to the bind shell at port `4444`. There also appears to be an open RDP connection from `192.168.119.4` to port `3389` on the machine.

## PowerShell History

Over the last decade, IT security awareness for the average enterprise user has tremendously improved through training, IT policies, and the prevalent threat of cyber attacks painted by media. Fortunately, this has led to less sensitive information being stored in notes or text files.

Because of the growing threat of cyber attacks, more defensive measures were developed and implemented for clients and servers alike. One of these measures is to collect and record more data on systems about executed commands and operations, which allows the IT staff to review and respond accordingly to threats. One important source of this information is PowerShell, since it is a vital resource for attackers.

With default settings, Windows only logs a small amount of information on PowerShell usage, which is not sufficient for enterprise environments. Therefore, attackers will often find PowerShell logging mechanisms enabled on Windows clients and servers. Two important logging mechanisms for PowerShell are:

1. PowerShell Transcription
2. PowerShell Script Block Logging

### Standard History

The [Get-History](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/get-history?view=powershell-7.3) cmdlet can be used to get the session history. As noted, the information logged by Windows is very limited and this will often result in an empty history:

```powershell
PS C:\Users\dave> Get-History

PS C:\Users\dave>
```

Most Administrators use the [Clear-History](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/clear-history?view=powershell-7.3) command to clear the PowerShell history. But this Cmdlet is only clearing PowerShell's own history, which can be retrieved with `Get-History`.

### Logs

When logs are enabled, PowerShell [logs](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about\_logging\_windows?view=powershell-7.2) details about PowerShell operations, such as starting and stopping the engine and providers, and executing PowerShell commands.

Logs can be found in [Windows Event Viewer](https://www.howtogeek.com/123646/htg-explains-what-the-windows-event-viewer-is-and-how-you-can-use-it/).

### PSReadline History

Starting with PowerShell v5, v5.1, and v7, a module named [PSReadline](https://learn.microsoft.com/en-us/powershell/module/psreadline/?view=powershell-7.2) is included, which is used for line-editing and command history functionality.

Interestingly, `Clear-History` does not clear the command history recorded by `PSReadline`. Therefore, it is possible to check if the user in the example misunderstood the `Clear-History` Cmdlet to clear all traces of previous commands.

To check the PSReadline history, first the location of the log must be found. This can be done with the [Get-PSReadLineOption](https://learn.microsoft.com/en-us/powershell/module/psreadline/get-psreadlineoption?view=powershell-7.3) Cmdlet and requesting its `HistorySavePath` property via:

```powershell
(Get-PSReadlineOption).HistorySavePath
```

On the example machine the file is revealed to be at:

```powershell
PS C:\Users\dave> (Get-PSReadlineOption).HistorySavePath

C:\Users\dave\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
```

From here the file can be read to see what commands have been run:

```powershell
PS C:\Users\dave> type C:\Users\dave\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt

whoami
ls
$PSVersionTable
Register-SecretVault -Name pwmanager -ModuleName SecretManagement.keepass -VaultParameters $VaultParams
Set-Secret -Name "Server02 Admin PW" -Secret "paperEarMonitor33@" -Vault pwmanager
cd C:\
ls
cd C:\xampp
ls
type passwords.txt
Clear-History
Start-Transcript -Path "C:\Users\Public\Transcripts\transcript01.txt"
Enter-PSSession -ComputerName CLIENTWK220 -Credential $cred
exit
Stop-Transcript
```

#### Prevention

Administrators can prevent PSReadline from recording commands by setting the `-HistorySaveStyle` option to `SaveNothing` with the [Set-PSReadlineOptions](https://learn.microsoft.com/en-us/powershell/module/psreadline/set-psreadlineoption?view=powershell-7.2) Cmdlet. Alternatively, they can clear the history file manually.

### Transcription

[PowerShell Transcription](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.host/start-transcript?view=powershell-7.3) creates a record of all or part of a PowerShell session to a text file. The transcript includes all command that the user types and all output that appears on the console.

Transcription is often referred to as "over-the-shoulder-transcription", because, when enabled, the logged information is equal to what a person would obtain from looking over the shoulder of a user entering commands in PowerShell. The information is stored in **transcript files**, which are often saved in the home directories of users, a central directory for all users of a machine, or a network share collecting the files from all configured machines.

In this example the path to the transcript file was found&#x20;

[earlier](manual-enumeration.md#psreadline-history), however, if it had not been it is possible to search the machine for files with "transcript" in the title via the following PowerShell command:

{% code overflow="wrap" %}
```powershell
Get-ChildItem -Path C:\ -Include "*transcript*" -Recurse -Force -File -ErrorAction SilentlyContinue
```
{% endcode %}

Whichever way it is found the transcript file can be read at `C:\Users\Public\Transcripts\transcript01.txt`:

{% code title="transcript01.txt" %}
```
**********************
Windows PowerShell transcript start
Start time: 20220623081143
Username: CLIENTWK220\dave
RunAs User: CLIENTWK220\dave
Configuration Name: 
Machine: CLIENTWK220 (Microsoft Windows NT 10.0.22000.0)
Host Application: C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
Process ID: 10336
PSVersion: 5.1.22000.282
PSEdition: Desktop
PSCompatibleVersions: 1.0, 2.0, 3.0, 4.0, 5.0, 5.1.22000.282
BuildVersion: 10.0.22000.282
CLRVersion: 4.0.30319.42000
WSManStackVersion: 3.0
PSRemotingProtocolVersion: 2.3
SerializationVersion: 1.1.0.1
**********************
Transcript started, output file is C:\Users\Public\Transcripts\transcript01.txt
PS C:\Users\dave> $password = ConvertTo-SecureString "qwertqwertqwert123!!" -AsPlainText -Force
PS C:\Users\dave> $cred = New-Object System.Management.Automation.PSCredential("daveadmin", $password)
PS C:\Users\dave> Enter-PSSession -ComputerName CLIENTWK220 -Credential $cred
PS C:\Users\dave> Stop-Transcript
**********************
Windows PowerShell transcript end
End time: 20220623081221
**********************
```
{% endcode %}

### Script Block Logging

[PowerShell Script Block Logging](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about\_logging\_windows?view=powershell-7.2#enabling-script-block-logging) records commands and blocks of script code as events while executing. This results in a much broader logging of information because it records the full content of code and commands as they are executed. This means such an event also contains the original representation of encoded code or commands.&#x20;

This is basically an enhanced form of [logging](manual-enumeration.md#logs). Like standard logs, these logs can be viewed in Event Viewer.
