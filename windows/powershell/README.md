---
description: Notes on the Windows PowerShell environment
---

# PowerShell

Windows PowerShell is a task-based command line shell and scripting language. It is installed by default on modern Windows platforms beginning with Windows Server 2008 R2 and Windows 7

<table><thead><tr><th width="198">Windows Version</th><th>PowerShell 5.0</th><th>PowerShell 4.0</th><th></th></tr></thead><tbody><tr><td>Server 2016</td><td>Installed by default</td><td>N/A</td><td>N/A</td></tr><tr><td>Server 2012 R2</td><td>Install Windows Management Framework 5.0 to run</td><td>Installed by default</td><td>N/A</td></tr><tr><td>Server 2012</td><td>Install Windows Management Framework 5.0 to run</td><td>Install Windows Management Framework 4.0 to run</td><td>Installed by default</td></tr><tr><td>2008 R2 with Service Pack 1</td><td>Install Windows Management Framework 5.0 to run</td><td>Install Windows Management Framework 4.0 to run</td><td>Install Windows Management Framework 3.0 to run</td></tr><tr><td>Windows 8.1</td><td>Install Windows Management Framework 5.0 to run</td><td>Installed by default</td><td>N/A</td></tr><tr><td>Windows 7 with Service Pack 1</td><td>Install Windows Management Framework 5.0 to run</td><td>Install Windows Management Framework 4.0 to run</td><td>Install Windows Management Framework 3.0 to run</td></tr></tbody></table>

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

