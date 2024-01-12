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

# Client-Side Attacks

According to Verizon's Data Breach Investigation Report, overcoming the perimeter by exploiting technical vulnerabilities has become increasingly rare and difficult. As of 2022, the report states that **phishing** is the second largest attack vector used for breaching a perimeter, surpassed only by credential attacks.

Phishing often leverages client-side attacks. This type of attack works by delivering malicious files directly to users. Once they execute these files on their machine, attackers can get a foothold in the internal network. Client-side attacks often exploit weaknesses or functions in local software and applications such as browsers, operating system components, or office programs. To execute malicious code on the client's system, one must often persuade, trick, or deceive the target user.

Client-side attacks often use specific delivery mechanisms and payload combinations, including email attachments or links to malicious websites or files. Phishing is the obvious delivery method, but one could leverage even more advanced delivery mechanisms such as **USB Dropping** or **watering hole attacks**.

Regardless of which delivery mechanism is chosen, the payload must often be delivered to a target on a non-routable internal network, since client systems are rarely exposed externally.

When choosing an attack vector and payload, an attacker must first perform reconnaissance to determine the operating system of the target as well as any installed applications. This is a critical first step, as the payload must match the capability of the target.

For example, if the target is running the Windows operating system, one could use a variety of client-side attacks like malicious JScript code executed through the Windows Script Host ([wscript](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/wscript)) or `.lnk` shortcut files pointing to malicious resources. If the target has installed Microsoft Office, one could leverage documents with embedded malicious macros.

This module will cover:

* Target Reconnaissance
* Exploiting Microsoft Office
* Abusing Windows Library Files
