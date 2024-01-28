---
description: Some other
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

# Other Uses

## Host Detection

Although it is sub-optimal, Netcat can be used to scan a network for listening hosts. In order to loop through a network and check for hosts listening on a specific port, a for loop could be used in Bash:

```bash
port=443; for i in $(seq 1 254); do nc -zv -w 1 172.16.50.$i $port; done
```

* This would iterate through hosts on the `172.16.50.0/24` subnet and check for machines listening on port `443`
* Several Netcat flags are used:
  * `-z` checks for a listening port without sending data
  * `-v` is for verbosity
  * `-w 1` sets a low timeout threshold

When used in an example the output appears as follows:

```bash
kali@kali:~$ port=445; for i in $(seq 1 254); do nc -zv -w 1 172.16.205.$i $port; done
nc: connect to 172.16.205.1 port 445 (tcp) timed out: Operation now in progress
nc: connect to 172.16.205.2 port 445 (tcp) timed out: Operation now in progress
nc: connect to 172.16.205.3 port 445 (tcp) timed out: Operation now in progress
...
Connection to 172.16.205.217 445 port [tcp/microsoft-ds] succeeded!
...
nc: connect to 172.16.205.254 port 445 (tcp) failed: Connection refused
```

* This example is searching `172.16.205.0/24` for hosts on port `445`
* One host was found at `172.16.205.217`
