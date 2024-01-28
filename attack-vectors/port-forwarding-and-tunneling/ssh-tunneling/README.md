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

# SSH Tunneling

As described [earlier](../#tunneling), **tunneling** describes the act of encapsulating one kind of data stream within another as it travels across a network. Certain protocols called [tunneling protocols](https://en.wikipedia.org/wiki/Tunneling\_protocol) are designed specifically to do this. [Secure Shell](https://www.ssh.com/academy/ssh/protocol) (SSH) is an example of one of these protocols.

SSH was initially developed to give administrators the ability to log in to their servers remotely through an encrypted connection. Before SSH, tools such as `rsh`, [`rlogin`](https://www.ssh.com/academy/ssh/rlogin), and [Telnet](https://en.wikipedia.org/wiki/Telnet) provided similar remote administration capabilities, but over an _unencrypted_ connection.

In the background of each SSH connection, all shell commands, passwords, and data are transported through an encrypted tunnel built using the SSH protocol. The SSH protocol is primarily a tunneling protocol, so it's possible to pass almost any kind of data through an SSH connection. For that reason, tunneling capabilities are built into most SSH tools.

Another great benefit of SSH tunneling is how its use can easily blend into the background traffic of network environments. SSH is used often by network administrators for legitimate remote administration purposes, and flexible port forwarding setups in restrictive network situations. It's therefore common to find SSH client software already installed on Linux hosts, or even SSH servers running there. It's also increasingly common to find [OpenSSH](https://www.openssh.com/) client software installed on Windows hosts. In network environments that are not heavily monitored, SSH traffic will not seem anomalous, and SSH traffic will look much like regular administrative traffic. Its contents also cannot be easily monitored.

In most official documentation, tunneling data through an SSH connection is referred to as [SSH port forwarding](https://www.ssh.com/academy/ssh/tunneling-example#what-is-ssh-port-forwarding,-aka-ssh-tunneling?). Different SSH software will provide slightly different port forwarding capabilities.



