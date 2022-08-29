---
description: Utilizing tcpdump for analysis
---

# Tcpdump

Tcpdump is a text-based network sniffer that is streamlined, powerful, and flexible despite the lack of a graphical interface.

It is by far the most commonly-used command-line packet analyzer and can be found on most Unix and Linux operating systems, but local user permissions determine the ability to capture network traffic.

Tcpdump can both capture traffic from the network and read existing capture files.

## Important Flags

| Flag         | Description                                     |
| ------------ | ----------------------------------------------- |
| `-i {IF}`    | Specifies interface  to `{IF}` (E.g. `-i eth0`) |
| `-n`         | Disable DNS look-ups                            |
| `-r {FILE}`  | Read from `{FILE}`                              |
| `-s {BYTES}` | Limits packet size (default is 65536 bytes)     |
| `-A`         | Print packet capture data as ASCII              |
| `-X`         | Print packet capture data in hex                |
