---
description: Alternative to netcat for networking utilities
---

# Socat

Command-line utility that establishes two bidirectional byte streams and transfers data between them. For penetration testing, it is similar to Netcat but has additional useful features.

## Usage

### Netcat vs. Socat

```bash
kali@kali:~$ nc <remote server's ip address> 80
```

```bash
kali@kali:~$ socat - TCP4:<remote server's ip address>:80
```

Note that the syntax is similar, but socat requires the - to transfer data between `STDIO` and the remote host (allowing our keyboard interaction with the shell) and protocol (TCP4). The protocol, options, and port number are colon-delimited.

Because root privileges are required to bind a listener to ports below 1024, we need to use `sudo` when starting a listener on port 443.

```bash
kali@kali:~$ sudo nc -lvp localhost 443
```

```bash
kali@kali:~$ sudo socat TCP4-LISTEN:443 STDOUT
```

Notice the required addition of both the protocol for the listener (TCP4-LISTEN) and the `STDOUT` argument, which redirects standard output.

