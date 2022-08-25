# Behavior Flags

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

