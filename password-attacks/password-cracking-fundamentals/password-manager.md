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

# Password Manager

This example will demonstrate a very common penetration test scenario. Assume the attacker has gained **access to a client workstation running a password manager**.

The example will show how to extract the password manager's database, transform the file into a format usable by Hashcat, and crack the master database password.

The target machine for this example is a Windows workstation called `SALESWK01` and located at `192.168.240.203`.  The attacker has obtained the credentials `jason:lab` for the machine and it accessible via RDP.

Given that accessing via RDP will give the attacker a GUI, they can just utilize the Windows "Apps & features" function&#x20;
