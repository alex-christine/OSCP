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

# Domain

Writeups for domain-connected exam machines. The pivot point is `MS01` which is accessible from the external network at `192.168.xxx.153`.

{% code title="machines.txt" %}
```
192.168.xxx.153 ms01.oscp.exam
10.10.yyy.152   dc01.oscp.exam
10.10.yyy.154   ms02.oscp.exam
```
{% endcode %}

* For easier scanning I also split this file by column into 2 files called `hosts.txt` and `hostnames.txt` containing the IP address and hostname columns respectively
