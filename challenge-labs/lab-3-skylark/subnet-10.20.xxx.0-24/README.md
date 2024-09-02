# Subnet 10.20.XXX.0/24

Holds writeups of the machines in `10.10.xxx.0/24` address range.

There are 4 machines listed in the network (names are just taken from machine launch screen in Challenge Labs):

{% code title="machines.txt" %}
```
10.20.XXX.14    vm5.skylark
10.20.XXX.15    vm6.skylark
10.20.XXX.110   vm7.skylark
10.20.XXX.111   vm8.skylark
```
{% endcode %}

* For easier scanning I also split by column into 2 files called `hosts.txt` and `hostnames.txt` containing the IP address and hostname columns respectively
* Machine names will be updated as new information is discovered

## Network Access

Access to this subnet is provided via a double-hop with `ligolo-ng` through agents on `AUSTIN02` and `MAIL`. The double-hop setup was described in the [Post-Exploit section](../subnet-10.10.xxx.0-24/mail.md#double-pivot-with-ligolo-ng) of the `MAIL` writeup.

## Network Enumeration

### Nmap

Once the proxy is set up I simply use a standard `nmap` command to enumerate the internal network:

{% code overflow="wrap" %}
```bash
sudo nmap -sT -Pn -p- -A -o ./10_20-tcp_connect-all.nmap -iL hostnames.txt
```
{% endcode %}

* Output from this command will be provided in each machine's writeup



