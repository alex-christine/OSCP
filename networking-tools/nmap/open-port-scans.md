---
description: Techniques for determining which ports on a host machine are open
---

# Open Port Scans

## Flags

<table><thead><tr><th width="116">Flag</th><th>Command</th></tr></thead><tbody><tr><td><code>-sA</code></td><td><a href="open-port-scans.md#tcp-ack-scan">TCP ACK Scan</a></td></tr><tr><td><code>-sF</code></td><td><a href="open-port-scans.md#tcp-fin-scan">TCP FIN Scan</a></td></tr><tr><td><code>-sI</code></td><td><a href="open-port-scans.md#zombie-scan">Zombie Scan</a></td></tr><tr><td><code>-sM</code></td><td><a href="open-port-scans.md#maimon-scan">Maimon Scan</a></td></tr><tr><td><code>-sN</code></td><td><a href="open-port-scans.md#null-scan">Null Scan</a></td></tr><tr><td><code>-sS</code></td><td><a href="open-port-scans.md#tcp-syn-scan">TCP SYN Scan</a> (Default scan mode for root-level users)</td></tr><tr><td><code>-sT</code></td><td><a href="open-port-scans.md#tcp-connect-scan">TCP Connect Scan</a> (Default scan mode for regular users)</td></tr><tr><td><code>-sU</code></td><td><a href="open-port-scans.md#undefined">UDP Scan</a></td></tr><tr><td><code>-sW</code></td><td><a href="open-port-scans.md#window-scan">Window Scan</a></td></tr><tr><td><code>-sX</code></td><td><a href="open-port-scans.md#xmas-scan">XMAS Scan</a></td></tr><tr><td>custom</td><td>Triggered <code>--scanflags=&#x3C;FLAGS></code> where <code>URG</code>, <code>ACK</code>, <code>PSH</code>, <code>RST</code>, <code>SYN</code>, <code>FIN</code> can be used to specify flags. Multiple can be used with no spaces (e.g. <code>SYNRST</code> would send a packet with the SYN and RST flags set</td></tr></tbody></table>

## Port States

Nmap considers 6 different port states in reporting.

1. **Open** - A service is listening on the specified port&#x20;
2. **Closed** - No service is listening on the specified port, although the port is accessible
   * By accessible, we mean that it is reachable and is not blocked by a firewall or other security appliances/programs
3. **Filtered** - Nmap cannot determine if the port is open or closed because the port is not accessible.
   * This state is usually due to a firewall preventing Nmap from reaching that port
     * Nmap’s packets may be blocked from reaching the port
     * Alternatively, the responses are blocked from reaching Nmap’s host
4. **Unfiltered** - Nmap cannot determine if the port is open or closed, although the port is accessible
   * This state is encountered when using an ACK scan `-sA`
5. Open|Filtered - Nmap cannot determine whether the port is open or filtered
6. Closed|Filtered - Nmap cannot decide whether a port is closed or filtered

## TCP SYN Scan

Default method of operation for nmap. TCP three-way handshake is not completed prior to connection tear-down.

1. Nmap sends SYN packet
2. Open port responds SYN/ACK
3. Nmap replies RST (as opposed to ACK) to close connection

A closed TCP port responds to a SYN packet with RST/ACK to indicate that it is not open

## TCP Connect Scan

Attempts 3-way TCP handshake with target port(s)

* Open port allows handshake completion
* Closed TCP port responds to a SYN packet with RST/ACK to indicate that it is not open

TCP 3-way handshake is followed immediately by a RST/ACK packet to tear down the connection.

Unprivileged users are limited to TCP connect scan

## UDP Scan

Sends UDP packet to port

* Open port **not** expected to respond
  * Nmap cannot determine for sure if port is open or filtered
  * Also cannot guarantee that a service listening on a UDP port would respond to nmap's packets, further muddying the waters concerning open vs. filtered ports
* Closed port should reply with ICMP port unreachable (Type 3 Code 3) packet

Scan can be combined with a TCP scan.

## TCP ACK Scan

Sends TCP packet with ACK flag set

* Target should respond with RST for **both open and closed ports**
* No response indicates filtered port

**Helpful for targets with a firewall in front of them** as it tells attacker which ports are blocked or allowed by the firewall.

## TCP FIN Scan

Sends TCP packet with FIN flag set

* No response if the TCP port is open (or filtered)
  * Nmap cannot determine for sure if port is open or filtered (will report as open/filtered)
* Closed port responds with RST,ACK packet

## Zombie Scan

Requires an idle system connected to the network that can be communicated with.

Nmap will make each probe appear as if coming from the idle (zombie) host (by spoofing IP address), then it will check for indicators whether the idle (zombie) host received any response to the spoofed probe.&#x20;

Accomplished by checking the IP identification (IP ID) value in the IP header.

`nmap -sI <ZOMBIE_IP> <TARGET_IP>`

### Operation

1. Trigger the idle host to respond so that you can record the current IP ID on the idle host
   * Attacker sends idle host SYN,ACK packet to which the idle host responds with a RST from which attacker can check IP ID&#x20;
2. Send a SYN packet to a TCP port on the target. The packet should be spoofed to appear as if it was coming from the idle host (zombie) IP address
   * **Closed TCP port** the target machine responds to the idle host with an RST packet. The idle host does not respond; hence its IP ID is not incremented
     * **Firewall filtered port** will yield the same result as closed port (the idle host won’t increase the IP ID)&#x20;
   * **Open TCP port** the target machine responds with a SYN/ACK to the idle host (zombie). The idle host responds to this unexpected packet with an RST packet, thus incrementing its IP ID
3. Trigger the idle machine again to respond (send SYN,ACK to idle host and it responds with RST) so that nmap can compare the new IP ID with the one received earlier
   * If the difference is 1, it means the port on the target machine was closed or filtered - If the difference is 2, it means that the port on the target was open

## Maimon Scan

Sends TCP packet with FIN and ACK flags set

* Target should send an RST packet as a response whether **port is open or closed**
  * Certain BSD-derived systems drop the packet if it is an open port
* No response from **filtered ports**

**Will not work on most targets encountered in modern networks** but it is worth knowing about

## Null Scan

Sends TCP packet with none of the 6 flag bits set

* Open port does not respond
  * Nmap cannot determine for sure if it is open or filtered
* Closed port responds with a RST,ACK packet

## Window Scan

Almost the same as TCP ACK scan (-sA) except it also examines the window field of the TCP RST packet received in responses

* On some specific systems this can reveal that the port is open (as opposed to open|filtered)

## XMAS Scan

Sends TCP packet with FIN, PSH, and URG flags set.

Gets name because packet is "lit up like a Christmas tree"

* Open port does not respond
  * Nmap cannot determine for sure if port is open or filtered (will report as open|filtered)
* Closed port responds with RST,ACK
