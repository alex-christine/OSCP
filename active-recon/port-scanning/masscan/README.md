---
description: Utilizing the Masscan port-scanning utility
---

# Masscan

Masscan is arguably the fastest port scanner; it can transmit up to 10 million packets per second.

It is a TCP port scanner, that spews SYN packets asynchronously, scanning entire Internet in under 5 minutes.

Masscan implements its own custom TCP/IP stack. Therefore it needs access to raw sockets and must be run with root privileges.

* Masscan is primarily used for quickly scanning large network
  * Originally designed to scan the entire Internet and thus can easily handle a class A or B network
* Should be thought of as complementary to nmap (but not a replacement)
  * Runs much faster than nmap
    * It is roughly the equivalent of running `nmap -Pn -sS -p {ports} {IP_range}`
      * With no scripting or service discovery
  * Does not possess the same scripting and fine tooling as nmap
    * Masscan is capable of running a full TCP connect scan and doing some basic header grabbing (though this usually requires some additional configuration)
    * Will not make any attempt to determine what service is behind the port
  * May be helpful to spray across a network and then target nmap at any results that look promising/interesting

#### Syntax

```bash
kali@kali:~$ sudo masscan -p80 10.11.1.0/24 --rate=1000 -e tap0 --router-ip 10.11.0.1

Starting masscan 1.0.3 (http://bit.ly/14GZzcT) at 2019-03-04 17:15:40 GMT
 -- forced options: -sS -Pn -n --randomize-hosts -v --send-eth
Initiating SYN Stealth Scan
Scanning 256 hosts [1 port/host]
Discovered open port 80/tcp on 10.11.1.14                                      
Discovered open port 80/tcp on 10.11.1.39                                      
Discovered open port 80/tcp on 10.11.1.219                                     
Discovered open port 80/tcp on 10.11.1.227                                     
Discovered open port 80/tcp on 10.11.1.10                                      
Discovered open port 80/tcp on 10.11.1.50                                      
Discovered open port 80/tcp on 10.11.1.234
...
```

| Component          | Description                                                                                                                                            |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `-p{PORT}`         | <p>Specifies the <em>port</em> for scanning.<br><br>Can also be given as a range. E.g. <code>-p0-65535</code> would scan all ports.</p>                |
| `10.11.1.0/24`     | <p><em>IP range</em> (CIDR notation) to scan.<br><br>The entire Internet is scanned with <code>0.0.0.0/0</code></p>                                    |
| `--rate={num}`     | <p>Sets the scanning <em>rate</em> to <code>{num}</code> packets per second.<br><br>Maximum supported value is 10 million (<code>10000000</code>).</p> |
| `-e {interface}`   | <p>Sets the outbound <em>network interface</em> to use for the scan.<br><br>E.g. <code>eth0</code>, <code>tun0</code>, etc.</p>                        |
| `--router-ip {IP}` | Specify the IP address for the appropriate gateway                                                                                                     |
