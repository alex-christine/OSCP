---
description: How to interact with a DNS server
---

# Interacting with DNS Server

## `host`

The simplest way to interact with DNS on a Linux machine is the `host` command. It can be used to lookup various record types of a domain.

```bash
kali@kali:~$ host www.megacorpone.com
www.megacorpone.com has address 38.100.193.76

kali@kali:~$ host -t mx megacorpone.com
megacorpone.com mail is handled by 10 fb.mail.gandi.net.
megacorpone.com mail is handled by 50 mail.megacorpone.com.
megacorpone.com mail is handled by 60 mail2.megacorpone.com.
megacorpone.com mail is handled by 20 spool.mail.gandi.net.

kali@kali:~$ host -t txt megacorpone.com
megacorpone.com descriptive text "Try Harder"
```

* Use the `-t {TYPE}` flag to search specific DNS record types
  * Default is to look for an `A` record
