# File Transfer

Due to the fact that Powercat must itself first be transferred onto a system via some file transfer mechanism, it may not always be thought of as a file transfer tool despite possessing the capability.

## Basic File Transfer

In this example a Windows machine will be using Powercat to transfer a file to a waiting Kali machine (listening with netcat).

### Receiving Machine

Receiving machine must first set up a listener:

```bash
kali@kali:~$ sudo nc -lnvp 443 > receiving_powercat.txt
listening on [any] 443 ...
connect to [10.11.0.4] from (UNKNOWN) [10.11.0.22] 63661
```

### Host Machine

The machine making the transfer will then invoke Powercat (from a PowerShell prompt) to initiate the file transfer:

{% code overflow="wrap" %}
```powershell
PS C:\Users\Offsec> powercat -c 10.11.0.4 -p 443 -i C:\Users\Offsec\example.txt
```
{% endcode %}

<table><thead><tr><th width="158">Flag</th><th>Description</th></tr></thead><tbody><tr><td><code>-c {IP}</code></td><td>Specifies client mode and sets the listening IP address (transfer recipient)</td></tr><tr><td><code>-p {PORT}</code></td><td>Specifies the port number for connection</td></tr><tr><td><code>-i {PATH}</code></td><td>Indicates the local file for transfer</td></tr></tbody></table>
