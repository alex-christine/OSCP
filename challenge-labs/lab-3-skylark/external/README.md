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

# External

Machines that are externally-accessible, residing in the `192.168.xx.0/24`  range.

## Network Enumeration

I compile a list of IPs in `hosts.txt` and run a Nmap scan:

{% code title="hosts.txt" %}
```
192.168.249.220
192.168.249.221
192.168.249.222
192.168.249.223
192.168.249.224
192.168.249.225
192.168.249.226
192.168.249.227
```
{% endcode %}

```bash
sudo nmap -sS -Pn -A -p- -o ./external_tcp_all.nmap -iL hosts.txt
```

The results for each machine will be included in their respective pages.
