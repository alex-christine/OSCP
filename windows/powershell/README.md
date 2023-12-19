---
description: Notes on the Windows PowerShell environment
---

# PowerShell

Windows PowerShell is a task-based command line shell and scripting language. PowerShell was designed to extend the capabilities of the Command shell to run PowerShell commands called cmdlets. Cmdlets are similar to Windows Commands but provide a more extensible scripting language.

It is installed by default on modern Windows platforms beginning with Windows Server 2008 R2 and Windows 7

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

## PowerShell Execution Policy

[PowerShell Execution Policy](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about\_execution\_policies?view=powershell-7.4) is a safety feature that controls the conditions under which PowerShell loads configuration files and runs scripts. This feature helps prevent the execution of malicious scripts.

On a Windows computer one can set an execution policy for the local computer, for the current user, or for a particular session. It is also possible to use a Group Policy setting to set execution policies for computers and users.

### Policy Settings

Users can check their execution policy with the [Get-ExecutionPolicy](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.security/get-executionpolicy?view=powershell-7.4) cmdlet:

```powershell
PS C:\Users\dave> Get-ExecutionPolicy
Restricted
```

The execution policy can be set to the following levels:

#### AllSigned <a href="#allsigned" id="allsigned"></a>

* Scripts can run.
* Requires that all scripts and configuration files be signed by a trusted publisher, including scripts that you write on the local computer.
* Prompts you before running scripts from publishers that you haven't yet classified as trusted or untrusted.
* Risks running signed, but malicious, scripts.

#### Bypass <a href="#bypass" id="bypass"></a>

* Nothing is blocked and there are no warnings or prompts.
* This execution policy is designed for configurations in which a PowerShell script is built into a larger application or for configurations in which PowerShell is the foundation for a program that has its own security model.

#### Default <a href="#default" id="default"></a>

* Sets the default execution policy.
* **Restricted** for Windows clients.
* **RemoteSigned** for Windows servers.

#### RemoteSigned <a href="#remotesigned" id="remotesigned"></a>

* The default execution policy for Windows server computers.
* Scripts can run.
* Requires a digital signature from a trusted publisher on scripts and configuration files that are downloaded from the internet which includes email and instant messaging programs.
* Doesn't require digital signatures on scripts that are written on the local computer and not downloaded from the internet.
* Runs scripts that are downloaded from the internet and not signed, if the scripts are unblocked, such as by using the `Unblock-File` cmdlet.
* Risks running unsigned scripts from sources other than the internet and signed scripts that could be malicious.

#### Restricted <a href="#restricted" id="restricted"></a>

* The default execution policy for Windows client computers.
* Permits individual commands, but does not allow scripts.
* Prevents running of all script files, including formatting and configuration files (`.ps1xml`), module script files (`.psm1`), and PowerShell profiles (`.ps1`).

#### Undefined <a href="#undefined" id="undefined"></a>

* There is no execution policy set in the current scope.
* If the execution policy in all scopes is **Undefined**, the effective execution policy is **Restricted** for Windows clients and **RemoteSigned** for Windows Server.

#### Unrestricted <a href="#unrestricted" id="unrestricted"></a>

* The default execution policy for non-Windows computers and cannot be changed.
* Unsigned scripts can run. There is a risk of running malicious scripts.
* Warns the user before running scripts and configuration files that are not from the local intranet zone.

### Bypassing Execution Policy

Often attackers will find scripting disabled on a particular machine resulting in an error like:

```powershell
PS C:\Users\dave> .\winPEAS.ps1
.\PowerUp.ps1 : File C:\Users\dave\winPEAS.ps1 cannot be loaded because running scripts is disabled on this system.
For more information, see about_Execution_Policies at https:/go.microsoft.com/fwlink/?LinkID=135170.
At line:1 char:1
+ .\winPEAS.ps1
+ ~~~~~~~~~~~~~
    + CategoryInfo          : SecurityError: (:) [], PSSecurityException
    + FullyQualifiedErrorId : UnauthorizedAccess
```

In these instances it is sometimes possible to ignore this policy by running `PowerShell -ExecutionPolicy Bypass`or `powershell -ep bypass`:

```powershell
PS C:\Users\dave> PowerShell -ExecutionPolicy Bypass 

PS C:\Users\dave> .\winPEAS.ps1 > winPEAS_scan.txt


PS C:\Users\dave>
```

Based on the output, the script from above (`.\winPEAS.ps1`) was able to run after overwriting the execution policy.

This is one of the simplest ways to bypass Execution Policy, though more can be found [here](https://www.netspi.com/blog/technical/network-penetration-testing/15-ways-to-bypass-the-powershell-execution-policy/).
