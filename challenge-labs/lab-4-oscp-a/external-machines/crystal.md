---
description: Standalone machine
---

# CRYSTAL

## Enumeration

```
Nmap scan report for 192.168.152.144
Host is up (0.052s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.5
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 fb:ea:e1:18:2f:1d:7b:5e:75:96:5a:98:df:3d:17:e4 (ECDSA)
|_  256 66:f4:54:42:1f:25:16:d7:f3:eb:f7:44:9f:5a:1a:0b (ED25519)
80/tcp open  http    Apache httpd 2.4.52 ((Ubuntu))
|_http-generator: Nicepage 4.21.12, nicepage.com
|_http-title: Home
| http-git: 
|   192.168.152.144:80/.git/
|     Git repository found!
|     Repository description: Unnamed repository; edit this file 'description' to name the...
|     Last commit message: Security Update 
|     Remotes:
|_      https://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@github.com/PWK-Challenge-Lab/dev.git
|_http-server-header: Apache/2.4.52 (Ubuntu)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 4.15 - 5.8 (87%), Linux 5.0 - 5.4 (87%), Linux 2.6.32 (87%), Linux 2.6.32 or 3.10 (87%), WatchGuard Fireware 11.8 (87%), Linux 4.8 (86%), Synology DiskStation Manager 5.1 (86%), Linux 2.6.18 (86%), Linux 2.6.35 (86%), Linux 4.9 (86%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 4 hops
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 22/tcp)
HOP RTT      ADDRESS
1   50.96 ms 192.168.45.1
2   50.89 ms 192.168.45.254
3   52.10 ms 192.168.251.1
4   52.23 ms 192.168.152.144
```

Seems like there is only one logical place to start.

### Web Server Port 80

I find it is running on the [Joomla CMS](https://github.com/joomla/joomla-cms/tree/4.4-dev) which had some [RCE options](https://vulncheck.com/blog/joomla-for-rce). I spent awhile messing with them to no avail.

### git Enumeration

I start looking at the contents of `.git`. At first I thought the advantage was seeing the source code. Some of it looks vulnerable to SQLi. Only later did I realize it was the `.git` repo itself that was the target.

#### Downloading Repository

Since it is on a website, but not GitHub,  use `wget` to recursively download the entire directory:

```bash
wget --no-parent -r http://crystal.oscp/.git/
```

<figure><img src="../../../.gitbook/assets/OSA-CRYSTAL-RecursiveGitDownload.png" alt=""><figcaption><p>Downloading with wget</p></figcaption></figure>

#### Examining the Repository

Once downloaded I start looking through it with the basic git commands. First I confirm I am looking at the right git repo with:

```bash
git status
```

Then I look through the change history with the log command:

```bash
git log
```

<figure><img src="../../../.gitbook/assets/OSA-CRYSTAL-GitLog.png" alt=""><figcaption><p>Change history</p></figcaption></figure>

The last change involved a "Security Update" - I wonder what that was. I check with git show and the ID of the commit:

```bash
git show 44a055daf7a0cd777f28f444c0d29ddf3ff08c54
```

<figure><img src="../../../.gitbook/assets/OSA-CRYSTAL-GitShow.png" alt=""><figcaption><p>Credentials in diff</p></figcaption></figure>

I test these credentials, `stuart:BreakingBad92`, via SSH and I am in:

<figure><img src="../../../.gitbook/assets/OSA-CRYSTAL-StuartSsh.png" alt=""><figcaption><p>Stuart SSH</p></figcaption></figure>

## Privilege Escalation

I ran PEAS and `pspy`, but nothing too interesting turned up.

### Manual Enumeration

I check for several potentially interesting files and file types:

1. SSH key file with default `id_rsa` name
   * If stuck check for [non-default names](https://askubuntu.com/a/30792)
2. `.git` directories
3. `.zip` files

{% code overflow="wrap" %}
```bash
find / -type f -name "id_rsa" 2>/dev/null
```
{% endcode %}

{% code overflow="wrap" %}
```bash
find / -type d -name ".git" 2>/dev/null
```
{% endcode %}

{% code overflow="wrap" %}
```bash
find / -type f -name "*.zip" 2>/dev/null
```
{% endcode %}

Sometimes the output of these can be overwhelming but it is good practice to look through them

### Site Backups

With the search for `.zip` files I find 3 files called `sitebackup1`, `sitebackup2`, and `sitebackup3.zip`. I took these back to my machine and checked them out. The first 2 could not be decompressed due to errors:

<figure><img src="../../../.gitbook/assets/OSA-CRYSTAL-UnzipAttempts.png" alt=""><figcaption><p>Errors in zips</p></figcaption></figure>

The third one was a valid `.zip` but it required a password:

<figure><img src="../../../.gitbook/assets/OSA-CRYSTAL-Backup3LockedItems.png" alt=""><figcaption><p>Locked files in zip</p></figcaption></figure>

#### Cracking a ZIP

I used john to crack the sitebackup3.zip password:

```bash
zip2john sitebackup3.zip > sb3.hash
```

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt sb3.hash
```

All files used the same password:

<figure><img src="../../../.gitbook/assets/OSA-CRYSTAL-Siteback3_Cracked.png" alt=""><figcaption><p>Cracked with john</p></figcaption></figure>

I now can then unzip with:

```bash
unzip -P 'codeblue' sitebackup3.zip
```

<figure><img src="../../../.gitbook/assets/OSA-CRYSTAL-Siteback3_Unzipped.png" alt=""><figcaption><p>Zip file full decompressed</p></figcaption></figure>

I start making my way through them and find what seem to be credentials in the `configuration.php` file:

<figure><img src="../../../.gitbook/assets/OSA-CRYSTAL-Sitebackup3_ConfigPhpCreds.png" alt=""><figcaption><p>Potential creds in unzipped file</p></figcaption></figure>

I mistakenly tried the `joomla` user pair first (red highlight) but it was unsuccessful. Noticing that the `$sescret` did not appear to be hashed I decided to try that with the `chloe` username and it worked:

<figure><img src="../../../.gitbook/assets/OSA-CRYSTAL-ChloeSu.png" alt=""><figcaption><p>Testing Chloe's credentials</p></figcaption></figure>

`chloe` had `sudo` permissions for all binaries. Elevated access achieved:

<figure><img src="../../../.gitbook/assets/OSA-CRYSTAL-ChloeSudo.png" alt=""><figcaption><p>Run any command with sudo</p></figcaption></figure>
