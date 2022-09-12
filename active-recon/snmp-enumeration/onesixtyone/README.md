# onesixtyone

onesixtyone is an SNMP scanner which utilizes a sweep technique. It can be used to discover devices responding to well-known community names or to mount a dictionary attack against one or more SNMP devices.

onesixtyone sends a request for the `system.sysDescr.0` value, which is present on almost all SNMP enabled devices. This returned value gives users a description of the system software running on the device.

## Usage

First one must build text files containing community strings and the IP addresses to be scanned:

```bash
kali@kali:~$ echo public > community
kali@kali:~$ echo private >> community
kali@kali:~$ echo manager >> community

kali@kali:~$ for ip in $(seq 1 254); do echo 10.11.1.$ip; done > ips
```

At this point the text files can be fed to `onesixtyone`:

```bash
kali@kali:~$ onesixtyone -c community -i ips
Scanning 254 hosts, 3 communities
10.11.1.14 [public] Hardware: x86 Family 6 Model 12 Stepping 2 AT/AT COMPATIBLE - Software: Windows 2000 Version 5.1 (Build 2600 Uniprocessor Free)
10.11.1.13 [public] Hardware: x86 Family 6 Model 12 Stepping 2 AT/AT COMPATIBLE - Software: Windows 2000 Version 5.1 (Build 2600 Uniprocessor Free)
10.11.1.22 [public] Linux barry 2.4.18-3 #1 Thu Apr 18 07:37:53 EDT 2002 i686
...
```

Once SNMP-capable systems have been revealed they can be probed for more useful information.
