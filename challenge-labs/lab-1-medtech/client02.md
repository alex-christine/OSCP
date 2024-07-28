---
description: Internal machine
---

# CLIENT02

My initial access is provided via the `wario` user over WinRM:

{% code overflow="wrap" %}
```bash
proxychains evil-winrm -i 172.16.215.83 -u 'wario' -p 'Mushroom!'
```
{% endcode %}

<figure><img src="../../.gitbook/assets/Medtech-CLIENT02-MarioWinrm.png" alt=""><figcaption><p>Initial access</p></figcaption></figure>

`wario` happened to be an admin so getting the root proof file was simple.
