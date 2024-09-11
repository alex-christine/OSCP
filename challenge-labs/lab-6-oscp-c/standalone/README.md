---
layout:
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# Standalone

Writeups for standalone machines on the `192.168.xxx.0/24` subnet.

{% code title="machines.txt" %}
```
192.168.xxx.155 pascha.oscp.exam
192.168.xxx.156 frankfurt.oscp.exam
192.168.xxx.157 charlie.oscp.exam
```
{% endcode %}

* For easier scanning I also split this file by column into 2 files called `hosts.txt` and `hostnames.txt` containing the IP address and hostname columns respectively

## Enumeration

### Nmap

I start with an Nmap TCP stealth scan of all ports and a UDP scan of the top 250 for each:

```bash
sudo nmap -sS -Pn -A -p- -o standalone-tcp_stealth-all.nmap -iL hostnames.txt
```

```bash
sudo nmap -sU -Pn -A --top-ports 250 -o standalone-udp-top250.nmap -iL hostnames.txt
```

The output of these commands will be included in each machine's individual writeup.
