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

# Leveraging Services

[Windows Services](../../../core-concepts/windows-services.md) are one of the main areas to analyze when searching for privilege escalation vectors. This section will cover three abuse techniques:

1. Service Binary Hijacking
2. Service DLL Hijacking
3. Unquoted Service Paths

## Scanning for Escalation Paths

While the following sections cover the techniques for exploiting services there are some useful tools for examining what escalation paths exist.

### PowerUp

[PowerUp.ps1](https://github.com/PowerShellMafia/PowerSploit/blob/master/Privesc/PowerUp.ps1) is a script that automatically scans for privilege escalation vectors. It can be loaded onto a target machine and run to see what is found in an automated pass.

This example will utilize a machine `CLIENTWK220` on which the attacker is assumed to have access to a compromised user account `dave` via RDP.

Once signed in to the machine the attacker can move the script file to the machine however is easiest (HTTP/S, RDP Client, direct download from source). Once the file is downloaded and stored at `C:\Users\dave\PowerUp.ps1` the attacker can attempt to run it
