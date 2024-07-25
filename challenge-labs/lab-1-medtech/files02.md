---
description: Internal Network Machine
---

# FILES02

This machine is at the internal IP address of `172.16.X.11`. Initially I can access it via WinRM as `joe`:

{% code overflow="wrap" %}
```bash
proxychains evil-winrm -i 172.16.215.11 -u 'joe' -p 'Flowers1'
```
{% endcode %}

<figure><img src="../../.gitbook/assets/Medtech-FILES02-EvilWinrmJoe.png" alt=""><figcaption><p>evil-winrm session to FILES02</p></figcaption></figure>



