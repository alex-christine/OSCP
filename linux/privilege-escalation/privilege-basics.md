---
description: Basic file and user privileges explained for Linux
---

# Privilege Basics

One of the defining features of Linux and other UNIX derivatives is that most resources, including files, directories, devices, and even network communications are represented in the filesystem. Put colloquially, "everything is a file".

## File Permissions

Every file (and by extension every element of a Linux system) abides by user and group permissions based on three primary properties:&#x20;

1. **Read** (symbolized by `r`): allows reading the file content
2. **Write** (symbolized by `w`): allows changing file content
3. **Execute** (symbolized by `x`): allows running of file

Each file or directory has specific permissions for three categories of users:

1. The owner
2. Owner group
3. Others group

#### Example

```shell-session
kali@kali:~$ ls -l /etc/shadow
-rw-r----- 1 root shadow 1751 May  2 09:31 /etc/shadow
```

The first dash "`-`" is the file type. The first grouping of 3 (`rw-`) indicates the owner (`root`) has read/write permissions for the file. The second grouping (`r--`) indicates that the owner group has read permissions. The final grouping (`---`) indicates that others may not even read the file.&#x20;

### Directory Permissions

A directory is handled differently from a file.&#x20;

* Read access gives the right to consult the list of its contents (files and directories)
* Write access allows creating or deleting files
* Execute access allows crossing through the directory to access its contents (using the `cd` command, for example)
