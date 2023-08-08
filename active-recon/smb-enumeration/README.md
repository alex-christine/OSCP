---
description: >-
  Methodologies to determine what SMB-enabled machines are available on a
  network
---

# SMB Enumeration

## Scanning for NetBIOS Service

The NetBIOS service listens on TCP port 139 as well as several UDP ports.

It should be noted that SMB (TCP port 445) and NetBIOS are two separate protocols. NetBIOS is an independent session layer protocol and service that allows computers on a local network to communicate with each other.

While modern implementations of SMB can work without NetBIOS, **NetBIOS over TCP** (NBT) is required for backward compatibility and is often enabled together. For this reason, the enumeration of these two services often goes hand-in-hand.

### Nmap

Nmap can be used to do basic SMB scanning:

```
kali@kali:~$ nmap -p 139,445 {IP_range}
```

This will enumerate SMB (on the default ports) for a range of IP addresses.

It should also be noted that Nmap has a fairly extensive list of SMB scripts:

```bash
kali@kali:~$ ls /usr/share/nmap/scripts | grep smb
smb2-capabilities.nse
smb2-security-mode.nse
smb2-time.nse
smb2-vuln-uptime.nse
smb-brute.nse
smb-double-pulsar-backdoor.nse
smb-enum-domains.nse
smb-enum-groups.nse
smb-enum-processes.nse
smb-enum-services.nse
smb-enum-sessions.nse
smb-enum-shares.nse
smb-enum-users.nse
smb-flood.nse
smb-ls.nse
smb-mbenum.nse
smb-os-discovery.nse
smb-print-text.nse
smb-protocols.nse
smb-psexec.nse
smb-security-mode.nse
smb-server-stats.nse
smb-system-info.nse
smb-vuln-conficker.nse
smb-vuln-cve2009-3103.nse
smb-vuln-cve-2017-7494.nse
smb-vuln-ms06-025.nse
smb-vuln-ms07-029.nse
smb-vuln-ms08-067.nse
smb-vuln-ms10-054.nse
smb-vuln-ms10-061.nse
smb-vuln-ms17-010.nse
smb-vuln-regsvc-dos.nse
smb-vuln-webexec.nse
smb-webexec-exploit.nse
```

### Nbtscan

More specialized tool for scanning a network for NetBIOS machines. Unfortunately per its [official page](https://github.com/resurrecting-open-source-projects/nbtscan), it is no longer being actively maintained (they are searching for contributors).

```
kali@kali:~$ sudo nbtscan -r 10.11.1.0/24
Doing NBT name scan for addresses from 10.11.1.0/24

IP address       NetBIOS Name     Server    User             MAC address      
------------------------------------------------------------------------------
10.11.1.5        ALICE            <server>  ALICE            00:50:56:89:35:af
10.11.1.31       RALPH            <server>  HACKER           00:50:56:89:08:19
10.11.1.24       PAYDAY           <server>  PAYDAY           00:00:00:00:00:00
```

* `-r` option is used to specify the originating UDP port as 137 (used to query the NetBIOS name service for valid NetBIOS names)

## Tools for Examining SMB Machines

Most of these tools will work for SMB and Samba.

### `nmblookup`

Designed to make use of queries for the NetBIOS names and then map them to their subsequent IP addresses in a network. The options allow the name queries to be directed at a particular IP broadcast area or to a particular machine. All queries are done over UDP.

### `nbtscan`

NBTscan is a program for scanning IP networks for NetBIOS name information. It sends NetBIOS status query to each address in supplied range and lists received information in human-readable form. For each responded host it lists IP address, NetBIOS computer name, logged-in user name and MAC address (such as Ethernet).

### `nbtstat`

This **Windows command** displays the NetBIOS over TCP/IP (NetBT) protocol statistics. It can read the NetBIOS name tables for both the local computer and remote computers. It can also read the NetBIOS name cache. This command allows a refresh of the NetBIOS name cache and the names registered with Windows Internet Name Service (WINS). When used without any parameters, this command displays Help Information. This command is available only if the Internet Protocol (TCP/IP) protocol is installed as a component in the properties of a network adapter in Network Connections.

### SMBMap

**Requires access to the share** (can be in the form of credentials or a public share).

&#x20;SMBMap allows users to enumerate samba share drives across an entire domain. List share drives, drive permissions, share contents, upload/download functionality, file name auto-download pattern matching, and even execute remote commands. This tool was designed with pen testing in mind and is intended to simplify searching for potentially sensitive data across large networks.

### `smbclient`

**Requires access to the share** (can be in the form of credentials or a public share).

smbclient is samba client with an “ftp like” interface. It is a useful tool to test connectivity to a Windows share. It can be used to transfer files, or to look at share names. In addition, it has a nifty ability to ‘tar’ (backup) and restore files from a server to a client and vice versa.

### Net view

**Requires access to the share** (can be in the form of credentials or a public share).

Displays a list of domains, computers, or resources that are being shared by the specified computer. Used without parameters, `net view` displays a list of computers in your current domain.

### `rpcclient`

rpcclient is a utility initially developed to test MS-RPC functionality in Samba itself. It has undergone several stages of development and stability. Many system administrators have now written scripts around it to manage Windows NT clients from their UNIX workstation.

## Windows Tools

### net view

A helpful tool for enumerating SMB shares from a Windows client is `net view`. It lists domains, resources, and computers belonging to a given host.

```
C:\Users\student>net view \\dc01 /all
Shared resources at \\dc01

Share name  Type  Used as  Comment

-------------------------------------------------------------------------------
ADMIN$      Disk           Remote Admin
C$          Disk           Default share
IPC$        IPC            Remote IPC
NETLOGON    Disk           Logon server share
SYSVOL      Disk           Logon server share
The command completed successfully.
```

The `/all` keyword lists the administrative shares ending with the dollar sign.
