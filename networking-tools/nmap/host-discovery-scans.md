---
description: Techniques for host discovery with nmap
---

# Host Discovery Scans

<table><thead><tr><th width="122">Flag</th><th>Description</th></tr></thead><tbody><tr><td><code>-PA</code></td><td><a href="host-discovery-scans.md#tcp-ack-scan">TCP ACK Scan</a></td></tr><tr><td><code>-PE</code></td><td>ICMP Echo (ping) scan</td></tr><tr><td><code>-PM</code></td><td>ICMP Address Mask queries (ICMP Type 17 request/ ICMP Type 18 reply)</td></tr><tr><td><code>-PP</code></td><td>ICMP Timestamp requests</td></tr><tr><td><code>-PR</code></td><td>ARP scan</td></tr><tr><td><code>-PS</code></td><td><a href="host-discovery-scans.md#tcp-syn-scan">TCP SYN Scan</a></td></tr><tr><td><code>-PU</code></td><td>UDP Ping Scan</td></tr><tr><td><code>-Pn</code></td><td><a href="host-discovery-scans.md#skip-host-discovery">Skip Host Discovery</a></td></tr></tbody></table>

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
