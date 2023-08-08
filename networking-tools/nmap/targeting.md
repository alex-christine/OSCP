---
description: Controlling targets of an nmap scan
---

# Targeting

The basic command structure for nmap is:

```
kali@kali:~$ nmap <BEHAVIOR-FLAGS> <TARGET> <OUTPUT (optional)>
```

This section will focus on the `<TARGET>` section of the command, and describe how to direct nmap against an individual target or a chose group of targets.

## Single Target

Using nmap against a single target is the simplest command format. In order to run a scan against `192.168.50.1` the command would look like:

<pre><code><strong>kali@kali:~$ nmap 192.168.50.1
</strong></code></pre>

No matter how many behavior flags exist this structure does not change:

```
kali@kali:~$ nmap -Pn -sS -O -osscan-guess --top-ports=20 --script http-headers 192.168.50.1 -oG nmap-grep.txt
```

## Multiple Targets

### List of Targets

To specify multiple targets, simply specify the desired targets with a space. In order to run a scan against `192.168.50.1`, `.20`, and `.53` the command would be:

```
kali@kali:~$ nmap 192.168.50.1 192.168.50.20 192.168.50.53
```

### Consecutive Range

To specify multiple targets in a consecutive range the dash (`-`) can be used to denote a range:

```
kali@kali:~$ nmap 192.168.50.20-50
```

The above command would run a default scan against all IPs from `192.168.50.20` to `192.168.50.50` (inclusive).

### Wildcard

A wildcard (`*`) character can be used to search an entire range:

```
kali@kali:~$ nmap 192.168.50.*
```

The wildcard character can appear in any place in the address:

```
kali@kali:~$ nmap 192.168.*.1
```

### Subnet Notation

An entire IP range can be searched using subnet notation:

```
kali@kali:~$ nmap 192.168.50.0/24
```

The above command will search the entire `/24` subnet of the `192.168.50.0` range. This notation can be used for any network range.
