---
description: Collection of commands to aid in local privilege escalation
---

# Local Privilege Escalation

## Machine Information

### Username

```
whoami
```

If `whoami` is available just run the command below and skip [past Privileges](./#network-information):

```
whoami /all
```

If `whoami` is not available use:

```
echo %USERNAME%
```

### Group Membership

```
net user "$user"
```

```
whoami /groups
```

### Privileges

```
whoami /priv
```

### Network Information





