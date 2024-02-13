---
description: Notes on moving files using PowerShell
---

# File Transfers

Because of the versatility (i.e. complexity) of PowerShell the file transfer mechanisms are more complex than they would be with netcat or socat for example.

#### Example

In this example a copy of `wget.exe` will be downloaded from the machine at `10.11.0.4` via HTTP. This command is executed from the standard Windows Command prompt (`cmd.exe`).

{% code overflow="wrap" lineNumbers="true" %}
```shell
C:\Users\offsec> powershell -c "(new-object System.Net.WebClient).DownloadFile('http://10.11.0.4/wget.exe','C:\Users\offsec\Desktop\wget.exe')"

C:\Users\offsec\Desktop> wget.exe -V
GNU Wget 1.9.1
```
{% endcode %}

<table><thead><tr><th width="308">Component</th><th>Description</th></tr></thead><tbody><tr><td><code>-c</code></td><td>Execute the supplied command (wrapped in double-quotes) as if it were typed at the PowerShell prompt.</td></tr><tr><td><code>New-Object</code></td><td>Cmdlet to instantiate either a .Net Framework or a COM object.</td></tr><tr><td><code>System.Net.WebClient</code></td><td>Object class used to access web resources via URI</td></tr><tr><td><code>DownloadFile({URL}, {LOC})</code></td><td>Download file from <code>{URL}</code> and store it on the local machine at <code>{LOC}</code></td></tr></tbody></table>
