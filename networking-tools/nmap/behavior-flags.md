---
description: Flags for controlling Nmap operations
---

# Behavior Flags

## General

<table><thead><tr><th width="132">Flag</th><th>Description</th></tr></thead><tbody><tr><td><code>-d</code> &#x26; <code>-dd</code></td><td>Include debug info (<code>-dd</code> is the more in-depth option)</td></tr><tr><td><code>-e</code></td><td>Specify network interface (e.g. <code>-e tun0</code>)</td></tr><tr><td><code>-f</code></td><td>Fragment packets</td></tr><tr><td><code>-F</code></td><td>Fast port scan (usually decreases scan from 1000 to 100 ports)</td></tr><tr><td><code>-n</code></td><td>Disable reverse DNS lookup</td></tr><tr><td><code>-p</code></td><td>Specify port(s)</td></tr><tr><td><code>-sn</code></td><td>Host discovery only</td></tr><tr><td><code>-R</code></td><td>Query DNS database for all hosts (even offline ones)</td></tr><tr><td><code>-T</code></td><td>Scan Timing</td></tr><tr><td><code>-v</code> &#x26; <code>-vv</code></td><td>Verbose mode &#x26; very verbose mode (respectively)</td></tr></tbody></table>

## Machine Scan

Flags controlling scans run on hosts (after port detection)

<table><thead><tr><th width="192">Flag</th><th>Description</th></tr></thead><tbody><tr><td><code>-A</code></td><td>Enable OS detection, version detection, script scanning, and traceroute</td></tr><tr><td><code>-O</code></td><td>OS detection</td></tr><tr><td><code>--osscan-guess</code></td><td>Force Nmap print the guessed result even if is not fully accurate (used in conjunction with <code>-O</code> or <code>-A</code> flags)</td></tr></tbody></table>

## Fragment Packets

IP data will be divided into 8 bytes or less. This can be useful in circumventing firewalls that are trying to prevent scanning

* Adding another `-f` (`-f -f` or `-ff`) will split the data into 16 byte-fragments instead of 8
* Can change the default value by using the `--mtu` flag
  * Value provided should always be a multiple of 8

## Specify Ports

Can be specified as a single port, range of ports, or all ports

<table><thead><tr><th width="192">Port Flag</th><th></th></tr></thead><tbody><tr><td><code>-p-</code></td><td>Scan all ports</td></tr><tr><td><code>-p21</code></td><td>Scan port 21 only</td></tr><tr><td><code>-p21-25</code></td><td>Scan ports 21-25 (inclusive)</td></tr><tr><td><code>-p21,25</code></td><td>Scan only ports 21 and 25</td></tr><tr><td><code>--top-ports=15</code></td><td>Scans the top 15 (any number can be used) ports as ranked by popularity in the <code>/usr/share/nmap/nmap-services</code> file</td></tr><tr><td><code>--open</code></td><td>Limits the output displayed to only open ports</td></tr></tbody></table>

## Scan Timing

Can be `-T<0-5>` where 0 is slowest (most delay between requests) and 5 is fastest

* Paranoid (0)
* Sneaky (1)
* Polite (2)
* Normal (3) - DEFAULT
* Aggressive (4)
* Insane (5)

Throttling the scan can be useful for evading IDS/IPS and EDR defenses.

## Advanced Controls <a href="#general" id="general"></a>

<table><thead><tr><th width="274">Flag</th><th>Description</th></tr></thead><tbody><tr><td><code>--data-length {NUM}</code></td><td>Append random data to reach given length (in bytes)</td></tr><tr><td><code>--disable-arp-ping</code></td><td>Disable ARP pings</td></tr><tr><td><code>--dns-servers {SERVER}</code></td><td>​Set DNS server for reverse DNS</td></tr><tr><td><code>-iL {FILENAME}</code></td><td>Accepts an input file containing hosts to be scanned (one per line)</td></tr><tr><td><code>--max-parallelism {NUM}</code></td><td>Control parallelization an ensure nmap is using at most <code>NUM</code> probe(s) running in parallel</td></tr><tr><td><code>--min-parallelism {NUM}</code></td><td>Control parallelization an ensure nmap is using at least <code>NUM</code> probe(s) running in parallel</td></tr><tr><td><code>--max-rate {NUM}</code></td><td>Ensures nmap is sending at most <code>NUM</code> request(s) per second</td></tr><tr><td><code>--min-rate {NUM}</code></td><td>Ensures nmap is sending at least NUM request(s) per second</td></tr><tr><td><code>--reason</code></td><td>Nmap will include the reasoning behind each of its conclusions on OS, port status, service, etc.</td></tr><tr><td><code>--source-port {PORT}</code></td><td>Specify source port for requests</td></tr><tr><td><code>--traceroute</code></td><td>Add information on the route between scanner and target</td></tr></tbody></table>

## Output Formats <a href="#output-formats" id="output-formats"></a>

Used to send output to a file (as well as on screen)​.

<table><thead><tr><th width="240">Flag</th><th>Description</th></tr></thead><tbody><tr><td><code>-oN {filepath}</code></td><td>Normal (file will appear just like on-screen results)</td></tr><tr><td><code>-oG {filepath}</code></td><td>Grepable, condenses output into fewer lines (more useful with <code>grep</code> later)</td></tr><tr><td><code>-oX {filepath}</code></td><td>XML</td></tr><tr><td><code>-oN {filepath}</code></td><td>Generates all 3 output file types (same name, different extensions)</td></tr></tbody></table>
