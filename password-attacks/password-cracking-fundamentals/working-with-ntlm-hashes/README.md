---
description: Techniques for and examples of passing and cracking different versions of NTLM
---

# Working with NTLM Hashes

Windows stores hashed user passwords in the **Security Account Manager** (**SAM**) database file, which is used to authenticate local or remote users.

To deter offline SAM database password attacks, Microsoft introduced the _SYSKEY_ feature in Windows NT 4.0 SP3, which partially encrypts the SAM file. The passwords can be stored in two different hash formats: _LAN Manager_ (LM) and NTLM. LM is based on _DES_, and is known to be very weak. For example, passwords are case insensitive and cannot exceed fourteen characters. If a password exceeds seven characters, it is split into two strings, each hashed separately. LM is disabled by default beginning with Windows Vista and Windows Server 2008.

On modern systems, the hashes in the SAM are stored as NTLM hashes. This hash implementation addresses many weaknesses of LM. For example, passwords are case-sensitive and are no longer split into smaller, weaker parts. However, NTLM **hashes stored in the SAM database are not salted.**

One cannot simply copy, rename, or move the SAM database from `C:\Windows\system32\config\sam` while the Windows operating system is running because the kernel keeps an exclusive file system lock on the file. Fortunately, in many instances, the [Mimikatz](../../../software/mimikatz.md) tool can bypass this restriction.
