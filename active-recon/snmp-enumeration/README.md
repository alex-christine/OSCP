---
description: Techniques for enumerating via SNMP
---

# SNMP Enumeration

It has often been demonstrated that the Simple Network Management Protocol (SNMP) is not well-understood by many network administrators. This often results in SNMP misconfigurations, which can result in significant information leakage.

SNMP is based on UDP, a simple, stateless protocol, and is therefore susceptible to IP spoofing and replay attacks. In addition, the commonly used SNMP protocols 1, 2, and 2c offer no traffic encryption, meaning that SNMP information and credentials can be easily intercepted over a local network. Traditional SNMP protocols also have weak authentication schemes and are commonly left configured with default public and private community strings.

#### Example

The following MIB variables correspond to specific Windows SMTP information:

<table><thead><tr><th width="201">MIB Variable</th><th>Variable Name</th></tr></thead><tbody><tr><td>1.3.6.1.2.1.25.1.6.0</td><td>System Processes</td></tr><tr><td>1.3.6.1.2.1.25.4.2.1.2</td><td>Running Programs</td></tr><tr><td>1.3.6.1.2.1.25.4.2.1.4</td><td>Processes Path</td></tr><tr><td>1.3.6.1.2.1.25.2.3.1.4</td><td>Storage Units</td></tr><tr><td>1.3.6.1.2.1.25.6.3.1.2</td><td>Software Name</td></tr><tr><td>1.3.6.1.4.1.77.1.2.25</td><td>User Accounts</td></tr><tr><td>1.3.6.1.2.1.6.13.1.3</td><td>TCP Local Ports</td></tr></tbody></table>

These variables can be queried via SNMP to gain information about the network.
