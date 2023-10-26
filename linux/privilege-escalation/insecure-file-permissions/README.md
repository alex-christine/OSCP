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

# Insecure File Permissions

This section is going to explore how misconfigured file permissions might lead to different paths for privilege escalation.

The high level topics for this section are:

* Abuse insecure cron jobs to escalate privileges
* Abuse insecure file permissions to escalate privileges

As seen in previous sections, all examples herein will continue to leverage the compromised `joe:offsec` account via SSH on the example target machine.
