---
description: Using snmpcheck to request specific SNMP variables
---

# snmpcheck

Like to snmpwalk, snmpcheck (called with `snmp-check`) allows you to enumerate the SNMP devices and places the output in a very human readable friendly format.

The basic syntax is:

```
kali@kali:~$ snmp-check -c {string} {IP}
```

My basic read is this tool is better to use as a human user (output is prettier and better organized than snmwalk), but for automation purposes snmp-walk (or a custom tool) would be better due to its simplicity and its ability to look at specific SNMP OIDs.
