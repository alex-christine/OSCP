---
description: Steps to spoof IP and MAC address while using Nmap
---

# Spoofing IP and MAC

## Spoofing IP

`nmap -S <SPOOFED_IP> <MACHINE_IP>`

Nmap will craft all the packets using the provided source IP address `SPOOFED_IP`.&#x20;

The target machine will respond to the incoming packets sending the replies to the destination IP address `SPOOFED_IP`. For this scan to work and give accurate results, the attacker needs to monitor the network traffic to analyze the replies.

1. Attacker sends a packet with a spoofed source IP address to the target machine
2. Target machine replies to the spoofed IP address as the destination
3. Attacker captures the replies to figure out open ports

In general expect to specify the network interface using `-e` and to explicitly disable ping scan `-Pn`

## Spoofing MAC

When on the same subnet as the target it is possible to spoof the MAC as well as IP

Specify the source MAC address using `--spoof-mac SPOOFED_MAC`

Only possible if the attacker and the target machine are on the same Ethernet (802.3) network or same WiFi (802.11). Spoofing only works in a minimal number of cases where certain conditions are met

* Attacker might resort to using decoys to make it more challenging to be pinpointed
  * Concept is simple, make the scan appears to be coming from many IP addresses so that the attacker’s IP address would be lost among them
  * Launch a decoy scan by specifying a specific or random IP address after `-D`
    * E.g. `nmap -D 10.10.0.1,10.10.0.2,ME 10.10.169.175`
      * Will make the scan of 10.10.169.175 appear as coming from the IP addresses 10.10.0.1, 10.10.0.2, and then ME to indicate that scanner's IP address should appear in the third order
