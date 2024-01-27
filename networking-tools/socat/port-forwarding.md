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

# Port Forwarding

Socat can be used as a port forwarding tool. The [official documentation](http://www.dest-unreach.org/socat/doc/socat.html) contains some examples of port forwarding but they are not the most clear. [This article](https://www.redhat.com/sysadmin/getting-started-socat) contains some better examples for using socat as a port forwarding tool.&#x20;

## Single Connection

To create a port forward that will only accept a single connection use the command structure:

```bash
socat TCP4-LISTEN:81 TCP4:192.168.1.10:80
```

* `TCP4-LISTEN:$PORT` sets the WAN interface listening port (`81` in this example)
* `TCP4:$IP:$PORT` sets the forward connection (to port `80` on `192.168.1.10` in this example)

## Multiple Connections

To create a port forward that will accept multiple incoming connections use the command structure:

```bash
socat TCP4-LISTEN:81,fork,reuseaddr TCP4:192.168.1.10:80
```

* `TCP4-LISTEN:$PORT` sets the WAN interface listening port (`81` in this example)
* The command [`fork`](http://www.dest-unreach.org/socat/doc/socat.html#OPTION\_FORK) launches each connection in a child process
* The command [`reusaddr`](http://www.dest-unreach.org/socat/doc/socat.html#OPTION\_SO\_REUSEADDR) allows other sockets to bind to an address even if parts of it (e.g. the local port) are already in use by socat
* `TCP4:$IP:$PORT` sets the forward connection (to port `80` on `192.168.1.10` in this example)
