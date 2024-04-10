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

# VShadow

A [**Shadow Copy**](https://en.wikipedia.org/wiki/Shadow\_Copy), also known as **Volume Shadow Service** (**VSS**) is a Microsoft backup technology that allows the creation of snapshots of files or entire volumes. One of the benefits is that this can be done even when they are in use.

[`vshadow.exe`](https://learn.microsoft.com/en-us/windows/win32/vss/vshadow-tool-and-sample) is a command-line tool that allows users to create and manage volume shadow copies.

The basic command structure for copying a volume is:

```sh
vshadow [OptionalFlags] VolumeList
```

Some commonly used `[OptionalFlags]` are:

* `-p` specifies [**persistent shadow copies**](https://learn.microsoft.com/en-us/windows/win32/vss/vssgloss-p).
  * Flag is supported only on Windows server operating systems.
* `-nw` specifies shadow copies without involving [writers](https://learn.microsoft.com/en-us/windows/win32/vss/shadow-copy-creation-details)
  * Writers add some extra steps prior to the creation of the copy to create a more stable picture. This is needed for actual backups but when trying to take a snapshot as an attacker this is not needed
  * Mutually exclusive to the `-wi=<writer>` (include specific writer) and `-ww=<writer>` (exclude specific writer) flags

In attack scenarios this will most commonly be used to take a snapshot of a domain controller using the command:

```sh
vshadow.exe -nw -p  C:
```

An example of this can be seen [here](../active-directory/persistence/shadow-copies.md).
