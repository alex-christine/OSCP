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

# Lateral Movement

Lateral Movement is a tactic consisting of various techniques aimed at gaining further access within the target network. As described in the [MITRE Framework](https://attack.mitre.org/tactics/TA0008/), these techniques may use the current valid account or reuse authentication material such as password hashes, Kerberos tickets, and application access tokens obtained from the previous attack stages.

[Previous](../enumeration/) [sections](../authentication/) covered things like how to locate high-value targets that could lead to an Active Directory compromise and the workstations or servers they are logged in to. They also demonstrated how to recover password hashes and leverage existing tickets for Kerberos authentication. While these techniques are important and valuable a lot of them rely on being able to crack a hash. While that is certainly possible it is time consuming and may fail. In addition, Kerberos and NTLM do not use the clear text password directly, and native tools from Microsoft do not support authentication using the password hash. This section will cover other lateral movement techniques to compromise high-value targets in an AD environment using extracted hashes (without cracking) and tickets.

The section will cover:

* WMI, WinRS, and WinRM lateral movement techniques
* Abusing `PsExec` for lateral movement
* Pass-the-hash and overpass-the-hash techniques
* Misuse DCOM for lateral movement
* Kerberos Golden Tickets
* Shadow Copies
