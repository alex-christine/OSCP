---
description: Techniques for and examples of passing and cracking different versions of NTLM
---

# Working with NTLM Hashes

Windows stores hashed user passwords in the **Security Account Manager** (**SAM**) database file, which is used to authenticate local or remote users.

To deter offline SAM database password attacks, Microsoft introduced the _SYSKEY_ feature in Windows NT 4.0 SP3, which partially encrypts the SAM file. The passwords can be stored in two different hash formats: _LAN Manager_ (LM) and NTLM. LM is based on _DES_, and is known to be very weak. For example, passwords are case insensitive and cannot exceed fourteen characters. If a password exceeds seven characters, it is split into two strings, each hashed separately. LM is disabled by default beginning with Windows Vista and Windows Server 2008.

On modern systems, the hashes in the SAM are stored as NTLM hashes. This hash implementation addresses many weaknesses of LM. For example, passwords are case-sensitive and are no longer split into smaller, weaker parts. However, NTLM **hashes stored in the SAM database are not salted.**

One cannot simply copy, rename, or move the SAM database from `C:\Windows\system32\config\sam` while the Windows operating system is running because the kernel keeps an exclusive file system lock on the file. Fortunately, in many instances, the `mimikatz` tool can bypass this restriction.

### Mimikatz

[Mimikatz](https://github.com/gentilkiwi/mimikatz) provides the functionality to extract plain-text passwords and password hashes from various sources in Windows and leverage them in further attacks like pass-the-hash.&#x20;

Mimikatz also includes the `sekurlsa` module, which extracts password hashes from the LSASS process memory.&#x20;

#### LSASS

**Local Security Authority Subsystem** ([LSASS](https://en.wikipedia.org/wiki/Local\_Security\_Authority\_Subsystem\_Service)) is a process in Windows is responsible for enforcing the security policy on the system. It handles user authentication, password changes, and the creation of [access tokens](https://learn.microsoft.com/en-us/windows/win32/secauthz/access-tokens). It also writes to the _Windows Security Log._

Forcible termination of `lsass.exe` will result in the system losing access to any account, including NT AUTHORITY, prompting a restart of the machine.

LSASS is important because it caches NTLM hashes and other credentials, which can be extracted using the `sekurlsa` Mimikatz module.

LSASS runs under the `SYSTEM` user and is therefore even more privileged than a process started as Administrator.

Due to this, passwords can only be extracted if Mimikatz is being run as `Administrator` (or higher) and has the [SeDebugPrivilege](https://devblogs.microsoft.com/oldnewthing/20080314-00/?p=23113) access right enabled. This access right grants attackers the ability to debug not only processes they own, but also all other users' processes.

#### Privilege Escalation Techniques

More techniques are covered in [Privilege Escalation section](../../../windows/privilege-escalation/), but an attacker may elevate their privileges to the `SYSTEM` account with tools like [PsExec](https://learn.microsoft.com/en-us/sysinternals/downloads/psexec) or the built-in Mimikatz _token elevation function_ to obtain the required privileges. The token elevation function requires the [SeImpersonatePrivilege](https://learn.microsoft.com/en-us/troubleshoot/windows-server/windows-security/seimpersonateprivilege-secreateglobalprivilege) access right to work, but all local administrators have it by default.
