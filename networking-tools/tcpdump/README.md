---
description: Utilizing tcpdump for analysis
---

# Tcpdump

Tcpdump is a text-based network sniffer that is streamlined, powerful, and flexible despite the lack of a graphical interface.

It is by far the most commonly-used command-line packet analyzer and can be found on most Unix and Linux operating systems, but local user permissions determine the ability to capture network traffic.

Tcpdump can both capture traffic from the network and read existing capture files.

## Important Flags

<table><thead><tr><th width="165">Flag</th><th>Description</th></tr></thead><tbody><tr><td><code>-i {IF}</code></td><td>Specifies interface  to <code>{IF}</code> (E.g. <code>-i eth0</code>)</td></tr><tr><td><code>-n</code></td><td>Disable DNS look-ups</td></tr><tr><td><code>-r {FILE}</code></td><td>Read from <code>{FILE}</code></td></tr><tr><td><code>-s {BYTES}</code></td><td>Limits packet size (default is 65536 bytes)</td></tr><tr><td><code>-A</code></td><td>Print packet capture data as ASCII</td></tr><tr><td><code>-X</code></td><td>Print packet capture data in hex</td></tr></tbody></table>
