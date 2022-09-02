---
description: Netcat's client-side functionality
---

# Client

Can use client mode to connect to any TCP/UDP port, allowing us to:

* Check if a port is open or closed.
* Read a banner from the service listening on a port.
* Connect to a network service manually

```bash
kali@kali:~$ nc -nv 10.11.0.22 110
(UNKNOWN) [10.11.0.22] 110 (pop3) open
+OK POP3 server lab ready <00003.1277944@lab>
```

In this example the target machine is apparently running POP3 on port 110. The flag `-n` is used to disable DNS look ups and `-v` adds verbosity.
