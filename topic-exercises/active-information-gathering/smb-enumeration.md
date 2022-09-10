---
description: Answers to SMB enumeration Topic Exercises
---

# SMB Enumeration

## VM1

Server message block (SMB) is an extremely important service that can be used to determine a wealth of information about a server including its users. Use **nmap** to identify the lab machines listening on the smb port and then use **enum4linux** to enumerate those machines. In doing so, you will find a machine with the **local user `alfred`**. The **flag is located in the comments** of one of the SMB shares of the host that has the `alfred` user.

### Response

All IPs in the `192.168.229.0/24` range.

#### Enumeration

```
$ nmap -sV -p 139,445 192.168.229.1-255

Nmap scan report for 192.168.229.9
Host is up (0.064s latency).

PORT    STATE SERVICE       VERSION
139/tcp open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds?
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Nmap scan report for 192.168.229.11
Host is up (0.064s latency).

PORT    STATE SERVICE       VERSION
139/tcp open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds?
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Nmap scan report for 192.168.229.12
Host is up (0.061s latency).

PORT    STATE SERVICE       VERSION
139/tcp open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds?
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Nmap scan report for 192.168.229.13
Host is up (0.064s latency).

PORT    STATE SERVICE     VERSION
139/tcp open  netbios-ssn Samba smbd 4.6.2
445/tcp open  netbios-ssn Samba smbd 4.6.2

Nmap scan report for 192.168.229.14
Host is up (0.065s latency).

PORT    STATE SERVICE       VERSION
139/tcp open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds?
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Nmap scan report for 192.168.229.15
Host is up (0.064s latency).

PORT    STATE SERVICE       VERSION
139/tcp open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds?
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Nmap scan report for 192.168.229.20
Host is up (0.064s latency).

PORT    STATE SERVICE     VERSION
139/tcp open  netbios-ssn Samba smbd 4.6.2
445/tcp open  netbios-ssn Samba smbd 4.6.2

Nmap scan report for 192.168.229.149
Host is up (0.063s latency).

PORT    STATE SERVICE       VERSION
139/tcp open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds?
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

```

* I only kept the output for machines I found running SMB or Samba
* Found **8 IPs** to investigate further
  1. `192.168.229.9`
  2. `192.168.229.11`
  3. `192.168.229.12`
  4. `192.168.229.13`
  5. `192.168.229.14`
  6. `192.168.229.15`
  7. `192.168.229.20`
  8. `192.168.229.149`

Enum4linux errored on all of the IPs except `192.168.229.13` & `192.168.229.20` (technically it errored on these as well but in a different way):

```
kali@kali:~$ enum4linux -U 192.168.229.9
Starting enum4linux v0.9.1 ( http://labs.portcullis.co.uk/application/enum4linux/ ) on Fri Sep  9 19:04:48 2022

 =========================================( Target Information )=========================================
                                                                                                                   
Target ........... 192.168.229.9                                                                                   
RID Range ........ 500-550,1000-1050
Username ......... ''
Password ......... ''
Known Usernames .. administrator, guest, krbtgt, domain admins, root, bin, none


 ===========================( Enumerating Workgroup/Domain on 192.168.229.9 )===========================
                                                                                                                   
                                                                                                                   
[E] Can't find workgroup/domain                                                                                    
                                                                                                                   
                                                                                                                   

 ===================================( Session Check on 192.168.229.9 )===================================
                                                                                                                   
                                                                                                                   
[E] Server doesn't allow session using username '', password ''.  Aborting remainder of tests.
```

This was the same for all IPs except `192.168.229.13` & `192.168.229.20`:

```
kali@kali:~$ enum4linux -U 192.168.229.20
Starting enum4linux v0.9.1 ( http://labs.portcullis.co.uk/application/enum4linux/ ) on Fri Sep  9 19:09:18 2022

 =========================================( Target Information )=========================================
                                                                                                                   
Target ........... 192.168.229.20                                                                                  
RID Range ........ 500-550,1000-1050
Username ......... ''
Password ......... ''
Known Usernames .. administrator, guest, krbtgt, domain admins, root, bin, none


 ===========================( Enumerating Workgroup/Domain on 192.168.229.20 )===========================
                                                                                                                   
                                                                                                                   
[+] Got domain/workgroup name: WORKGROUP                                                                           
                                                                                                                   
                                                                                                                   
 ==================================( Session Check on 192.168.229.20 )==================================
                                                                                                                   
                                                                                                                   
[+] Server 192.168.229.20 allows sessions using username '', password ''                                           
                                                                                                                   
                                                                                                                   
 ===============================( Getting domain SID for 192.168.229.20 )===============================
                                                                                                                   
Domain Name: WORKGROUP                                                                                             
Domain Sid: (NULL SID)

[+] Can't determine if host is part of domain or part of a workgroup                                               
                                                                                                                   
                                                                                                                   
 ======================================( Users on 192.168.229.20 )======================================
                                                                                                                   
Use of uninitialized value $users in print at ./enum4linux.pl line 972.                                            
Use of uninitialized value $users in pattern match (m//) at ./enum4linux.pl line 975.

Use of uninitialized value $users in print at ./enum4linux.pl line 986.
Use of uninitialized value $users in pattern match (m//) at ./enum4linux.pl line 988.
enum4linux complete on Fri Sep  9 19:09:22 2022
```

This is progress. At least I know the tool can interact with these 2 machines. Now the question is can I find what I need through some other scan or do I have to fork & fix?

* In theory if I can still examine the comments of each machine it should be solvable...there are only 2
  * I know the shares are public because it seems enum4linux attempts to sign in with blank user and password - it did this successfully

No other enum4linux commands worked properly so I began digging into some other tools. Eventually I found `rpcclient` which allowed me to create a user session. Once signed in to the share I enumerated the share and the flag was presented:

```
kali@kali:~$ rpcclient -U "" -N 192.168.229.13
rpcclient $> netshareenum
netname: files
        remark: Flag: OS{c1d641e5642b32f96ad30c3c6aa6b8a7}
        path:   C:\tmp
        password:
rpcclient $> exit
```

* I think I got lucky the flag presented so quickly but exploring this path was helpful nonetheless
