---
description: Methods for detecting open ports and their services
---

# Port Scanning

Port scanning is the process of inspecting TCP or UDP ports on a remote machine with the intention of detecting what services are running on the target and what potential attack vectors may exist.

It is essential to understand the implication of port scanning. Due to the amount of traffic some scans can generate, along with their intrusive nature, **running port scans blindly can have adverse effects** on target systems or the client network such as overloading servers and network links or triggering IDS.

## Netcat

Netcat is **not** a port scanner, however it can serve as a rudimentary one in a pinch. The main reason this could serve a purpose is the near-ubiquity of Netcat on Linux systems.

### TCP Scanning

The simplest TCP port scanning technique, usually called CONNECT scanning, relies on the three-way TCP handshake mechanism. To perform a TCP handshake (with no data sent), the `-z` flag is used.

```bash
kali@kali:~$ nc -nvv -w 1 -z 10.11.1.220 3388-3390
(UNKNOWN) [10.11.1.220] 3390 (?) : Connection refused
(UNKNOWN) [10.11.1.220] 3389 (?) open
(UNKNOWN) [10.11.1.220] 3388 (?) : Connection refused
 sent 0, rcvd 0
```

<table><thead><tr><th width="155">Flag</th><th>Description</th></tr></thead><tbody><tr><td><code>-nvv</code></td><td>No DNS (<code>-n</code>) and very verbose (<code>-vv</code>)</td></tr><tr><td><code>-w {NUM}</code></td><td>Sets timeout to <code>{NUM}</code> seconds (1 in the example)</td></tr><tr><td><code>-z</code></td><td>Zero-I/O mode, which will send no data and is used for scanning.<br><br>Netcat sends <code>FIN-ACK</code> packet immediately after finishing the handshake. </td></tr></tbody></table>

The port range is given at the end of the command (3388-3390 in the example).

### UDP Scanning

UDP is stateless and thus does not rely on a three-way handshake. As a result the process for scanning is marginally different:

```bash
kali@kali:~$ nc -nv -u -z -w 1 10.11.1.115 160-162
(UNKNOWN) [10.11.1.115] 161 (snmp) open
```

* `-u` is used to indicate UDP for Netcat
* All other flags match the TCP scan

## Nmap

The following will be primarly academic discussion and cursory examples. For details on using Nmap scanning functions see [this section](../networking-tools/nmap/).

### Traffic Accountability

Nmap generates a tremendous amount of traffic. Any system with sophisticated defenses is almost certain to notice the noise generated.

#### Example

In this example the amount of network traffic will be tracked using the `iptables` utility.

```bash
kali@kali:~$ $ip={IP}
kali@kali:~$ sudo iptables -I INPUT 1 -s $ip -j ACCEPT
kali@kali:~$ sudo iptables -I OUTPUT 1 -d $ip -j ACCEPT
kali@kali:~$ sudo iptables -Z
```

<table><thead><tr><th width="213">Component</th><th>Description</th></tr></thead><tbody><tr><td><code>-I {chain} {num}</code></td><td><em>Insert</em> a new rule into a given chain.<br><br>In this case includes both the <code>INPUT</code> (Inbound) and <code>OUTPUT</code> (Outbound) chains followed by the rule number</td></tr><tr><td><code>-j ACCEPT</code></td><td>Used to tell tool to <em>accept</em> the traffic</td></tr><tr><td><code>-Z</code></td><td>Zero the packet and byte counters in all chains</td></tr><tr><td><code>-s {IP}</code></td><td>Specify a <em>source</em> IP address</td></tr><tr><td><code>-d {IP}</code></td><td>Specify a <em>desitnation</em> IP address</td></tr></tbody></table>

An nmap scan is then run against `$ip` (E.g. `nmap $ip`) and the output of `iptables` can be checked:

```bash
kali@kali:~$ sudo iptables -vn -L
Chain INPUT (policy ACCEPT 1528 packets, 226K bytes)
 pkts bytes target     prot opt in     out     source               destination
 1263 51264 ACCEPT     all  --  *      *       10.11.1.220          0.0.0.0/0

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination

Chain OUTPUT (policy ACCEPT 1323 packets, 191K bytes)
 pkts bytes target     prot opt in     out     source               destination
 1314 78300 ACCEPT     all  --  *      *       0.0.0.0/0            10.11.1.220
```

<table><thead><tr><th width="135">Flag</th><th>Component</th></tr></thead><tbody><tr><td><code>-vn</code></td><td>Add <em>verbosity</em> (<code>-v</code>) and enable <em>numeric</em> output (-n)</td></tr><tr><td><code>-L</code></td><td><em>List</em> all rules presnt in all chains</td></tr></tbody></table>

As can be seen even a stock-standard Nmap scan generates quite a bit of traffic.

### Stealth Scanning

Nmap's preferred scanning technique is a SYN, or **stealth** scan. As such, it is the default scan technique used when no scan technique is specified in an `nmap` command and the user has the required raw sockets privileges (E.g. `sudo nmap {IP}`).

TCP port scanning method that involves sending SYN packets to various ports on a target machine without completing a TCP handshake. If a TCP port is open, a SYN-ACK should be sent back from the target machine, informing us that the port is open. At this point, the port scanner does not bother to send the final ACK to complete the three-way handshake.

## Common Pitfalls

Below are some common pitfalls found (and mistakes made) during port scanning.

### UDP Scanning

UDP scanning can often be unreliable as firewalls and routers may drop ICMP packets. This can lead to false positives and ports showing as open when they are, in fact, closed.

### Not Scanning All Ports

Many port scanning tools do not scan all ports, opting instead to stick to a smaller list of "interesting" ports. This can lead to missing vulnerable items running on potentially odd ports.

Additionally, this often leads to UDP ports being overlooked as they are ignored for TCP ports.

Particularly **on the OSCP exam all scans should be run on all ports**.
