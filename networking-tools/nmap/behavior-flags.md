---
description: Flags for controlling Nmap operations
---

# Behavior Flags

## General

| Flag         | Description                                                    |
| ------------ | -------------------------------------------------------------- |
| `-d` & `-dd` | Include debug info (`-dd` is the more in-depth option)         |
| `-e`         | Specify network interface (e.g. `-e tun0`)                     |
| `-f`         | Fragment packets                                               |
| `-F`         | Fast port scan (usually decreases scan from 1000 to 100 ports) |
| `-n`         | Disable reverse DNS lookup                                     |
| `-p`         | Specify port(s)                                                |
| `-sn`        | Host discovery only                                            |
| `-R`         | Query DNS database for all hosts (even offline ones)           |
| `-T`         | Scan Timing                                                    |
| `-v` & `-vv` | Verbose mode & very verbose mode (respectively)                |

## Machine Scan

Flags controlling scans run on hosts (after port detection)

| Flag | Description                                                             |
| ---- | ----------------------------------------------------------------------- |
| `-A` | Enable OS detection, version detection, script scanning, and traceroute |
| `-O` | OS detection                                                            |

## Fragment Packets

IP data will be divided into 8 bytes or less. This can be useful in circumventing firewalls that are trying to prevent scanning

* Adding another `-f` (`-f -f` or `-ff`) will split the data into 16 byte-fragments instead of 8
* Can change the default value by using the `--mtu` flag
  * Value provided should always be a multiple of 8

## Specify Ports

Can be specified as a single port, range of ports, or all ports

| Port Flag |                              |
| --------- | ---------------------------- |
| `-p-`     | Scan all ports               |
| `-p21`    | Scan port 21 only            |
| `-p21-25` | Scan ports 21-25 (inclusive) |
| `-p21,25` | Scan only ports 21 and 25    |

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

| Flag                      | Description                                                                                      |
| ------------------------- | ------------------------------------------------------------------------------------------------ |
| `--data-length {NUM}`     | Append random data to reach given length (in bytes)                                              |
| `--disable-arp-ping`      | Disable ARP pings                                                                                |
| `--dns-servers {SERVER}`  | ​Set DNS server for reverse DNS                                                                  |
| `-iL {FILENAME}`          | Accepts an input file containing hosts to be scanned (one per line)                              |
| `--max-parallelism {NUM}` | Control parallelization an ensure nmap is using at most `NUM` probe(s) running in parallel       |
| `--min-parallelism {NUM}` | Control parallelization an ensure nmap is using at least `NUM` probe(s) running in parallel      |
| `--max-rate {NUM}`        | Ensures nmap is sending at most `NUM` request(s) per second                                      |
| `--min-rate {NUM}`        | Ensures nmap is sending at least NUM request(s) per second                                       |
| `--reason`                | Nmap will include the reasoning behind each of its conclusions on OS, port status, service, etc. |
| `--source-port {PORT}`    | Specify source port for requests                                                                 |
| `--traceroute`            | Add information on the route between scanner and target                                          |

## Output Formats <a href="#output-formats" id="output-formats"></a>

Used to send output to a file (as well as on screen)​.

| Flag             | Description                                                                 |
| ---------------- | --------------------------------------------------------------------------- |
| `-oN {filepath}` | Normal (file will appear just like on-screen results)                       |
| `-oG {filepath}` | Grepable, condenses output into fewer lines (more useful with `grep` later) |
| `-oX {filepath}` | XML                                                                         |
| `-oN {filepath}` | Generates all 3 output file types (same name, different extensions)         |
