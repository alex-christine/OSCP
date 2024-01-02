---
description: Techniques for enumerating via SMTP
---

# SMTP Enumeration

It is possible to gather information about a host or network from vulnerable mail servers

The Simple Mail Transport Protocol (SMTP) supports several interesting commands, such as:

* `VRFY` request asks the server to verify an email address
* `EXPN` asks the server for the membership of a mailing list

These can often be abused to verify existing users on a mail server, which is often useful information.

#### Example

Using Nectat to manually enumerate valid email users from a server:

```
kali@kali:~$ nc -nv 10.11.1.217 25
(UNKNOWN) [10.11.1.217] 25 (smtp) open
220 hotline.localdomain ESMTP Postfix
VRFY root
252 2.0.0 root
VRFY idontexist
550 5.1.1 <idontexist>: Recipient address rejected: User unknown in local recipient table
^C
```

This procedure can be used to help guess valid usernames in an automated fashion.

It can also be automated, in this example a Python script is used to open a TCP socket, connect to the SMTP server, and issues a `VRFY` command for a given username:

```python
#!/usr/bin/python

import socket
import sys

if len(sys.argv) != 2:
        print "Usage: vrfy.py <username>"
        sys.exit(0)

# Create a Socket
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# Connect to the Server
connect = s.connect(('10.11.1.217',25))

# Receive the banner
banner = s.recv(1024)

print banner

# VRFY a user
s.send('VRFY ' + sys.argv[1] + '\r\n')
result = s.recv(1024)

print result

# Close the socket
s.close()
```

* This could be easily extended to go through a wordlist or tied to a random username generator
