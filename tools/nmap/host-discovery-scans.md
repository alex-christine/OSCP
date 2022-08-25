---
description: Techniques for host discovery with nmap
---

# Host Discovery Scans

| Flag  | Description                                                          |
| ----- | -------------------------------------------------------------------- |
| `-PA` | [TCP ACK Scan](host-discovery-scans.md#tcp-ack-scan)                 |
| `-PE` | ICMP Echo (ping) scan                                                |
| `-PM` | ICMP Address Mask queries (ICMP Type 17 request/ ICMP Type 18 reply) |
| `-PP` | ICMP Timestamp requests                                              |
| `-PR` | ARP scan                                                             |
| `-PS` | [TCP SYN Scan](host-discovery-scans.md#tcp-syn-scan)                 |
| `-PU` | UDP Ping Scan                                                        |
| `-Pn` | [Skip Host Discovery](host-discovery-scans.md#skip-host-discovery)   |



## TCP ACK Scan

Sends TCP ACK packet to target.

* Open port should reply with a RST (Reset)&#x20;
* Closed does not respond (each request is sent twice for assurance)

Runs on port 80 by default, but can be configured to run specific ports e.g. `-PA21-25` will scan ports 21-25

## TCP SYN Scan

Sends TCP SYN packet to target.

* Open port should reply with a SYN/ACK (Acknowledge)
* Closed port would result in an RST (Reset)

Can be configured to run specific ports e.g. `-PS21-25` will scan ports 21-25

## Skip Host Discovery

This is particularly useful in Windows environments where machines are configured to ignore `ping` (ICMP) by default. This could cause nmap to skip port/service discovery on a host because it appeared down. The `-Pn` flag tells nmap to continue with these processes even if a host did not reply to the discovery portion

* Forces nmap to treat all hosts as online
  * Even if they are non-discoverable or even non-existent
