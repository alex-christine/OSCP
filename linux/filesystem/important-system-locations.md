---
description: List of some important locations within the Linux filesystem
---

# Important System Locations

## Networking

### `/etc/resolv.conf`

Configuration file used by the Linux operating system to store information about Domain Name System (DNS) servers. This file contains a list of DNS server addresses, as well as other options that control how DNS resolution works on the system.

#### Example

{% code title="/etc/resolv.conf" %}
```
nameserver 8.8.8.8
nameserver 8.8.4.4
search example.com
```
{% endcode %}

* First 2 lines of this example are the Google DNS servers
* Third line in this file is the “search” line, which tells the system which domain should be used when resolving hostnames that are not fully qualified
  * E.g., if you try to ping `host` without specifying a domain, the system will automatically search for `host.example.com`
  * This line is optional and can be removed entirely if not needed

### `/etc/iptables/*`

the iptables-persistent package on Debian Linux saves firewall rules in specific files under `/etc/iptables` by default. These files are used by the system to restore `netfilter` rules at boot time. These files are often left with weak permissions, allowing them to be read by any local user on the target system.

## Security and Credentials

### `/etc/passwd`

Plain text-based database that contains information for all user accounts on the system

Owned by root and has 644 permissions

* Can only be modified by root or users with sudo privileges
* Readable by all system users
  * To modify a user account use the command: `usermod`
  * To add a new user account use the command: `useradd`

One entry per line, representing a user account.

* Typically, the first line describes the root user, followed by the system and normal user accounts
  * New entries are appended at the end of the file

#### Structure

Each line consists of seven colon (`:`) separated fields

![Example /etc/passwd entry](../../.gitbook/assets/Example\_etc-passwd.png)

1. Username
2. Password
   * In older Linux systems, the user’s encrypted password was stored in the `/etc/passwd` file
   * On most modern systems, this field is set to `x`, and the user password is stored in the `/etc/shadow` file
3. UID (User Identifier)
   * Number assigned to each user
     * Used by the operating system to refer to a user
4. GID - Group identifier
   * Number referring to the user’s primary group
   * When a user creates a file , the file’s group is set to this group
   * Typically, the name of the group is the same as the name of the user
   * User’s secondary groups are listed in the `/etc/groups` file
5. GECOS - Full name of the user
   * Contains a list of comma-separated values with the following information:
     1. User’s full name or the application name
     2. Room number
     3. Work phone number
     4. Home phone number
     5. Other contact information
6. Home Directory
   * Absolute path to the user’s home directory
   * Contains the user’s files and configurations
   * By default, the user home directories are named after the name of the user and created under the `/home` directory
7. Login Shell
   * Absolute path to the user’s login shell
   * Shell that is started when the user logs into the system
   * On most Linux distributions, the default login shell is Bash

### `/etc/shadow`

Modern complement to the `/etc/passwd` file.

Most commonly used and standard scheme for user authentication in Linux is to perform authentication against the `/etc/passwd` and `/etc/shadow` files.

Owned by user root and group shadow, and has **640 permissions**.

#### Structure

File contains one entry per line, each representing a user account.

Each line consists of nine colon (`:`) separated fields

![Example /etc/shadow entry](../../.gitbook/assets/Example\_etc-shadow.png)

* Encrypted password
  * The password is stored using the `$type$salt$hashed` format
    * `$type` is the method cryptographic hash algorithm

<table><thead><tr><th width="86">Type</th><th>Hash Algorithm</th></tr></thead><tbody><tr><td><code>$1</code></td><td>MD5</td></tr><tr><td><code>$2a</code></td><td>Blowfish</td></tr><tr><td><code>$2y</code></td><td>Eskblowfish</td></tr><tr><td><code>$5</code></td><td>SHA-256</td></tr><tr><td><code>$6</code></td><td>SHA-512</td></tr></tbody></table>

If the password field contains an asterisk (`*`) or exclamation point (`!`), the user will not be able to login to the system using password authentication.

### `/home/<username>/.ssh`

User's SSH keys are stored here.

Usually this file is only readable by root and the user themself.

## System Info

### `/etc/crontab`

Linux system file that creates a table-like structure where fields are separated by white space.

* Users can populate the table by assigning values to each field (`*`)
* Cronjob each complete row can be thought of as an individual job
* A system process called a Daemon (`crond` in this case) runs in the background of our Linux machine

![Cron job definitions](../../.gitbook/assets/Example\_etc-crontab.png)

### `/etc/fstab`

This file lists all drives that will be mounted at boot time. Infromation contained herein is simliar to what is output by the `mount` command.

### `/etc/issue`

File that contains information about the particular distribution of Linux

From the `man` page: issue is a text file which contains a message or system identification to be printed before the lo‐ gin prompt. It may contain various @char and \char sequences, if supported by the getty‐type program employed on the system.

Below is the issue file from the machine I wrote these notes on:

{% code title="/etc/issue" %}
```
Kali GNU/Linux Rolling \n \l
```
{% endcode %}

### `/etc/*release`

Some Linux distributions will have a file named `*release` (e.g. `os-release`). This is not universal to all Unix/Linux distributions but can be useful on the ones that do have them.

Where no release file is present `uname` will usually suffice as a replacement.

Below is the os-release file from this VM:

{% code title="/etc/os-release" %}
```
PRETTY_NAME="Kali GNU/Linux Rolling"
NAME="Kali GNU/Linux"
VERSION_ID="2023.3"
VERSION="2023.3"
VERSION_CODENAME=kali-rolling
ID=kali
ID_LIKE=debian
HOME_URL="https://www.kali.org/"
SUPPORT_URL="https://forums.kali.org/"
BUG_REPORT_URL="https://bugs.kali.org/"
ANSI_COLOR="1;31"
```
{% endcode %}
