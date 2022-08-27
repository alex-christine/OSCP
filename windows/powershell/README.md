---
description: Notes on the Windows PowerShell environment
---

# PowerShell

Windows PowerShell is a task-based command line shell and scripting language. It is installed by default on modern Windows platforms beginning with Windows Server 2008 R2 and Windows 7

| Windows Version               | PowerShell 5.0                                  | PowerShell 4.0                                  |                                                 |
| ----------------------------- | ----------------------------------------------- | ----------------------------------------------- | ----------------------------------------------- |
| Server 2016                   | Installed by default                            | N/A                                             | N/A                                             |
| Server 2012 R2                | Install Windows Management Framework 5.0 to run | Installed by default                            | N/A                                             |
| Server 2012                   | Install Windows Management Framework 5.0 to run | Install Windows Management Framework 4.0 to run | Installed by default                            |
| 2008 R2 with Service Pack 1   | Install Windows Management Framework 5.0 to run | Install Windows Management Framework 4.0 to run | Install Windows Management Framework 3.0 to run |
| Windows 8.1                   | Install Windows Management Framework 5.0 to run | Installed by default                            | N/A                                             |
| Windows 7 with Service Pack 1 | Install Windows Management Framework 5.0 to run | Install Windows Management Framework 4.0 to run | Install Windows Management Framework 3.0 to run |

PowerShell contains a built-in Integrated Development Environment (IDE), known as the Windows PowerShell Integrated Scripting Environment (ISE).

The ISE is a host application for Windows PowerShell that enables us to run commands, write, test, and debug scripts in a single Windows-based graphical user interface.

#### Example

```powershell
Windows PowerShell
Copyright  (C) Microsoft Corporation. All rights reserved.

PS C:\WINDOWS\system32> Set-ExecutionPolicy Unrestricted

Execution Policy Change
The execution policy helps protect you from scripts that you do not trust. Changing 
the execution policy might expose you to the security risks described in the 
about_Execution_Policies help topic at https:/go.microsoft.com/fwlink/?LinkID=135170. 
Do you want to change the execution policy?
[Y] Yes  [A] Yes to All  [N] No  [L] No to All  [S] Suspend  [?] Help (default is "N"): y

PS C:\WINDOWS\system32> Get-ExecutionPolicy
Unrestricted
```

