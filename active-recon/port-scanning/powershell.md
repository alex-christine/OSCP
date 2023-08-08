---
description: Living-off-the-land techniques leveraging PowerShell
---

# PowerShell

If conducting initial network enumeration from a Windows laptop with no internet access, testers are prevented from installing any extra tools that might help us, (E.g. the Windows Nmap version). In such a limited scenario, they are forced to pursue **living off the land (LotL)** techniques. Such techniques involve leveraging native Windows functionality in place of purpose-built tools.

This section will focus on built-in PowerShell utilities that can be used to approximate some host/port scanning functionality.

## Test-NetConnection

The `Test-NetConnection` function checks if an IP responds to ICMP and whether a specified TCP port on the target host is open.

A simple check to see if the SMB port (445) was open on a particular machine (`192.168.50.51`) would look like:

```powershell
PS C:\Users\student> Test-NetConnection -Port 445 192.168.50.151

ComputerName     : 192.168.50.151
RemoteAddress    : 192.168.50.151
RemotePort       : 445
InterfaceAlias   : Ethernet0
SourceAddress    : 192.168.50.152
TcpTestSucceeded : True
```

The returned value in the **TcpTestSucceeded** parameter indicates that port 445 is open.

### Automation

The testing process can be scripted with a PowerShell one liner such as:

{% code overflow="wrap" %}
```powershell
PS C:\Users\student> 1..1024 | % {echo ((New-Object Net.Sockets.TcpClient).Connect("192.168.50.151", $_)) "TCP port $_ is open"} 2>$null
```
{% endcode %}

The above will scan ports 1-1024 of `192.168.50.151` and print the open ports:

```powershell
PS C:\Users\student> 1..1024 | % {echo ((New-Object Net.Sockets.TcpClient).Connect("192.168.50.151", $_)) "TCP port $_ is open"} 2>$null
TCP port 53 is open
TCP port 88 is open
TCP port 135 is open
TCP port 139 is open
TCP port 389 is open
...
```
