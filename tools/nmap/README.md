---
description: Usage of nmap tool
---

# Nmap

Tool used to answer the questions

* Which systems are up?
* What services are running on these systems?

By default, Nmap will attempt to connect to the 1000 most common ports (though this behavior can be modified)

## Order of Operations

Some of these steps are optional but the order does not change even if certain steps are skipped.

1. Enumerate targets
2. Discover live hosts
3. Reverse DNS lookup
4. Scan ports
5. Detect versions
6. Detect OS
7. Traceroute
8. Scripts
9. Write Output
