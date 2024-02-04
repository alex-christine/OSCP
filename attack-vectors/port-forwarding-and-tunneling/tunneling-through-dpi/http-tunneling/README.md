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

# HTTP Tunneling

HTTP Tunneling is exactly what it sounds like. Instead of placing data in an "SSH tunnel" as seen in an [earlier section](../../ssh-tunneling/), data is encapsulated in an HTTP stream. The data itself is packed into the HTTP message body (or headers) and then the traffic is sent over what appears to be a normal HTTP connection but is actually a tunnel.

Chisel is one of the best tools for this available. Older tools such as [HTTPTunnel](https://http-tunnel.sourceforge.net/) provide viable alternatives but lack the cross-platform compatibility of Chisel.
