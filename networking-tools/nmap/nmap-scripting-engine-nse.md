---
description: Scripting engine notes and important scripts
---

# Nmap Scripting Engine (NSE)

Nmap provides support for scripts using the Lua language

Nmap Scripting Engine (NSE) is a Lua interpreter that allows Nmap to execute Nmap scripts written in Lua language

Default scripts stored at `/usr/share/nmap/scripts`

The `--script-help` option can be used in conjunction with a script name for more information about that script:

```
kali@kali:~$ nmap --script-help http-headers
Starting Nmap 7.94 ( https://nmap.org ) at 2023-08-02 20:01 MDT

http-headers
Categories: discovery safe
https://nmap.org/nsedoc/scripts/http-headers.html
  Performs a HEAD request for the root folder ("/") of a web server and displays the HTTP headers returned.
```

## Script Categories

<table><thead><tr><th width="127">Category</th><th>Description</th></tr></thead><tbody><tr><td>auth</td><td>Authentication related scripts</td></tr><tr><td>broadcast</td><td>Discover hosts by sending broadcast messages</td></tr><tr><td>brute</td><td>Performs brute-force password auditing against logins</td></tr><tr><td>default</td><td>Default scripts, same as <code>-sC</code></td></tr><tr><td>discovery</td><td>Retrieve accessible information, such as database tables and DNS names</td></tr><tr><td>dos</td><td>Detects servers vulnerable to Denial of Service (DoS)</td></tr><tr><td>exploit</td><td>Attempts to exploit various vulnerable services</td></tr><tr><td>external</td><td>Checks using a third-party service, such as Geoplugin and Virustotal</td></tr><tr><td>fuzzer</td><td>Launch fuzzing attacks</td></tr><tr><td>intrusive</td><td>Intrusive scripts such as brute-force attacks and exploitation</td></tr><tr><td>malware</td><td>Scans for backdoors</td></tr><tr><td>safe</td><td>Safe scripts that won’t crash the target</td></tr><tr><td>version</td><td>Retrieve service versions</td></tr><tr><td>vuln</td><td>Checks for vulnerabilities or exploit vulnerable services</td></tr></tbody></table>

Used via `--script=<category or script-name>`

* Can be a comma-separated list of script-names

## Unsafe

Safe scripts are designed not to crash services and machines. However, if one sets the script parameter `unsafe=1`, the scripts that will run are almost (or in some cases, totally) guaranteed to crash a vulnerable system. Exercise extreme caution when enabling this argument, especially when scanning production systems.

#### Example

In this example a server is being scanned for vulnerability to [MS08-067](https://docs.microsoft.com/en-us/security-updates/SecurityBulletins/2008/ms08-067) and the  `unsafe=1` has been passed via `--script-args=`. This is almost guaranteed to crash the machine if it is vulnerable.

```bash
kali@kali:~$ nmap -v -p 139,445 --script=smb-vuln-ms08-067 --script-args=unsafe=1 10.11.1.5
Starting Nmap 7.70 ( https://nmap.org ) at 2019-03-04 11:27 EST
NSE: Loaded 1 scripts for scanning.
NSE: Script Pre-scanning.
...
Scanning 10.11.1.5 [2 ports]
...
Completed NSE at 00:04, 17.39s elapsed
Nmap scan report for 10.11.1.5
Host is up (0.17s latency).
PORT    STATE SERVICE
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds
MAC Address: 00:50:56:AF:02:91 (VMware)

Host script results:
| smb-vuln-ms08-067:
|   VULNERABLE:
|   Microsoft Windows system vulnerable to remote code execution (MS08-067)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2008-4250
|           The Server service in Microsoft Windows 2000 SP4, XP SP2 and SP3, Server 2
|           Vista Gold and SP1, Server 2008, and 7 Pre-Beta allows remote attackers to
|           code via a crafted RPC request that triggers the overflow during path cano
|
|     Disclosure date: 2008-10-23
|     References:
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2008-4250
|_      https://technet.microsoft.com/en-us/library/security/ms08-067.aspx
...
```

## Important Scripts

Full list compiled [here](https://nmap.org/nsedoc/scripts/)

### SMB

<table><thead><tr><th width="224">Script Name</th><th>Description</th></tr></thead><tbody><tr><td><code>smb-os-discovery</code></td><td>Attempts to discover host OS, domain name, and other information via open SMB ports</td></tr><tr><td><code>smb-security-mode</code></td><td>Returns information about the SMB security level determined by SMB</td></tr><tr><td><code>smb2-vuln-uptime</code></td><td>Attempts to detect missing patches in Windows systems by checking the uptime returned during the SMB2 protocol negotiation</td></tr><tr><td><code>smb-enum-domains</code></td><td>Attempts to enumerate domains on a system, along with their policies. This generally requires credentials, except against Windows 2000</td></tr><tr><td><code>smb-enum-groups</code></td><td>Obtains a list of groups from the remote Windows system, as well as a list of the group's users</td></tr><tr><td><code>smb-enum-users</code></td><td>Attempts to enumerate the users on a remote Windows system, with as much information as possible</td></tr><tr><td><code>smb-enum-shares</code></td><td>Attempts to list shares using the <code>srvsvc.NetShareEnumAll MSRPC</code> function and retrieve more information about them using <code>srvsvc.NetShareGetInfo</code></td></tr></tbody></table>

### HTTP

<table><thead><tr><th width="181">Script Name</th><th>Description</th></tr></thead><tbody><tr><td><code>http-headers</code></td><td>Attempts to connect to the HTTP service on a target system and determine the supported headers</td></tr><tr><td><code>http-enum</code></td><td>Enumerates directories used by popular web applications and servers</td></tr></tbody></table>

### Vulnerabilities

<table><thead><tr><th width="183">Script Name</th><th>Description</th></tr></thead><tbody><tr><td><code>vulners</code></td><td>For each available CPE the script prints out known vulns (links to the correspondent info) and correspondent CVSS scores.</td></tr></tbody></table>
