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
