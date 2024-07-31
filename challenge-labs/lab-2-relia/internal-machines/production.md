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

# PRODUCTION

## Enumeration

```
Nmap scan report for 172.16.159.20
Host is up (0.035s latency).
Not shown: 999 filtered tcp ports (no-response)
PORT   STATE SERVICE    VERSION
22/tcp open  tcpwrapped
|_ssh-hostkey: ERROR: Script execution failed (use -d to debug)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
No OS matches for host

TRACEROUTE
HOP RTT      ADDRESS
1   35.07 ms 172.16.159.20
```

## Foothold

Initial access is provided with `andrew`'s credentials over SSH. These were found in the Borg backup [on `BACKUP`](backup.md#borg).

```bash
ssh andrew@172.16.158.20
```

```
Rb9kNokjDsjYyH
```

<figure><img src="../../../.gitbook/assets/Relia-PRODUCTION-AndrewSsh.png" alt=""><figcaption><p>Initial access</p></figcaption></figure>

## Privilege Escalation



