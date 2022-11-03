---
description: Scripting engine notes and important scripts
---

# Nmap Scripting Engine (NSE)

Nmap provides support for scripts using the Lua language

Nmap Scripting Engine (NSE) is a Lua interpreter that allows Nmap to execute Nmap scripts written in Lua language

Default scripts stored at `/usr/share/nmap/scripts`

## Script Categories

| Category  | Description                                                            |
| --------- | ---------------------------------------------------------------------- |
| auth      | Authentication related scripts                                         |
| broadcast | Discover hosts by sending broadcast messages                           |
| brute     | Performs brute-force password auditing against logins                  |
| default   | Default scripts, same as `-sC`                                         |
| discovery | Retrieve accessible information, such as database tables and DNS names |
| dos       | Detects servers vulnerable to Denial of Service (DoS)                  |
| exploit   | Attempts to exploit various vulnerable services                        |
| external  | Checks using a third-party service, such as Geoplugin and Virustotal   |
| fuzzer    | Launch fuzzing attacks                                                 |
| intrusive | Intrusive scripts such as brute-force attacks and exploitation         |
| malware   | Scans for backdoors                                                    |
| safe      | Safe scripts that won’t crash the target                               |
| version   | Retrieve service versions                                              |
| vuln      | Checks for vulnerabilities or exploit vulnerable services              |

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

| Script Name         | Description                                                                                                                                       |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| `smb-os-discovery`  | Attempts to discover host OS, domain name, and other information via open SMB ports                                                               |
| `smb-security-mode` | Returns information about the SMB security level determined by SMB                                                                                |
| `smb2-vuln-uptime`  | Attempts to detect missing patches in Windows systems by checking the uptime returned during the SMB2 protocol negotiation                        |
| `smb-enum-domains`  | Attempts to enumerate domains on a system, along with their policies. This generally requires credentials, except against Windows 2000            |
| `smb-enum-groups`   | Obtains a list of groups from the remote Windows system, as well as a list of the group's users                                                   |
| `smb-enum-users`    | Attempts to enumerate the users on a remote Windows system, with as much information as possible                                                  |
| `smb-enum-shares`   | Attempts to list shares using the `srvsvc.NetShareEnumAll MSRPC` function and retrieve more information about them using `srvsvc.NetShareGetInfo` |
