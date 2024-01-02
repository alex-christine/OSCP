---
description: Techniques for enumerating SMTP from a Windows client
---

# Enumerating in Windows

Basic SMTP information can be obtained from the Test-NetConnection cmdlet:

```powershell
PS C:\Users\student> Test-NetConnection -Port 25 192.168.50.8

ComputerName     : 192.168.50.8
RemoteAddress    : 192.168.50.8
RemotePort       : 25
InterfaceAlias   : Ethernet0
SourceAddress    : 192.168.50.152
TcpTestSucceeded : True
```

Unfortunately this cmdlet does not allow full interaction with SMTP.

## Telnet

One alternative is Telnet. Assuming it is available, Telnet can be leveraged to interact with the SMTP servers:

```
C:\Windows\system32>telnet 192.168.50.8 25
220 mail ESMTP Postfix (Ubuntu)
VRFY goofy
550 5.1.1 <goofy>: Recipient address rejected: User unknown in local recipient table
VRFY root
252 2.0.0 root
```

If Telnet is not available but the tester is running as an administrator on the Windows machine, it is possible to install Telnet using [DISM](../../windows/core-concepts/dism.md).
