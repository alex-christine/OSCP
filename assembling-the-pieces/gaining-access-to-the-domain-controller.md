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

# Gaining Access to the Domain Controller

Now that the attacker has gained local Administrator access to the `MAILSRV1` machine there seems to be a clear path forward. Recall that the Domain Admin user `beccy` had an active session on `MAILSRV1`. With this elevated access perhaps the hash of her credentials can be accessed and then levered into domain controller access.

