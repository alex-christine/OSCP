---
description: Page for domain-wide information
---

# Domain

## Starting Point

I started with SharpHound run from as `jim` from `WK01` as I followed the steps [described here](wk01.md#domain-enumeration).

I discovered domain accounts have **no lockout threshold** and I was able to AS-REP roast michelle which gave me RDP access to [`INTRANET`](intranet.md). Once there I also ran SharpHound as michelle from INTRANET. Once I combined the results, BloodHound became a lot more useful. Accounts had names and understandable relationships.

I was able to Kerberoast the `iis_service` user but I could not crack the hash with `rockyou.txt` and `best64` rule list.

As of writing this I have user access to WK01 and INTRANET. It seems my path forward may be escalating privileges on one of them and finding a different user's credentials in memory.



