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

# External Enumeration

## External Nmap

I grab the IPs of the externally-facing hosts and place them in a file at external/hosts.txt. I then scan them with Nmap:

{% code overflow="wrap" %}
```bash
sudo nmap -sS -Pn -A -p- -o ./external/enumeration/tcp_syn_all.nmap -iL ./external/hosts.txt
```
{% endcode %}



