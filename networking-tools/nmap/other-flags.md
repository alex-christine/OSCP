---
description: Miscellaneous flags for nmap
---

# Other Flags

## General

| Flag                      | Command                                                                                          |
| ------------------------- | ------------------------------------------------------------------------------------------------ |
| `--data-length <NUM>`     | Append random data to reach given length (in bytes)                                              |
| `--disable-arp-ping`      | Disable ARP pings                                                                                |
| `--dns-servers <SERVER>`  | Set DNS server for reverse DNS                                                                   |
| `-iL <FILENAME>`          | Accepts an input file containing hosts to be scanned (one per line)                              |
| `--max-parallelism <MAX>` | Control parallelization an ensure nmap is using at most NUM probe(s) running in parallel         |
| `--min-parallelism <MIN>` | Control parallelization an ensure nmap is using at least NUM probe(s) running in parallel        |
| `--max-rate <MAX>`        | Ensures nmap is sending at most NUM request(s) per second                                        |
| `--min-rate <MIN>`        | Ensures nmap is sending at least NUM request(s) per second                                       |
| `--reason`                | Nmap will include the reasoning behind each of its conclusions on OS, port status, service, etc. |
| `--source-port <PORT>`    | Specify source port for requests                                                                 |
| `--traceroute`            | Add information on the route between scanner and target                                          |

## Output Formats

Used to send output to a file (as well as on screen)

| Flag             | Description                                                                 |
| ---------------- | --------------------------------------------------------------------------- |
| `-oN <filename>` | Normal (file will appear just like on-screen results)                       |
| `-oG <filename>` | Grepable, condenses output into fewer lines (more useful with `grep` later) |
| `-oX <filename>` | XML                                                                         |
| `-oA <filename>` | Generates all 3 output file types (same name, different extensions)         |
