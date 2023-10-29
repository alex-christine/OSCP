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

# Insecure System Components

This section will describe how misconfigured system applications and permissions can also lead to elevation of rights.

The core principles are learning how to:

* Abuse SUID programs and capabilities for privilege escalation
* Circumvent special sudo permissions to escalate privileges
* Enumerate the system's kernel for known vulnerabilities, then abuse them for privilege escalation

As seen in previous sections, all examples herein will continue to leverage the compromised `joe:offsec` account via SSH on the example target machine.
