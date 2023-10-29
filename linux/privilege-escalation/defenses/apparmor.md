---
layout:
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# AppArmor

[AppArmor](https://apparmor.net/) is a kernel module that provides mandatory access control (MAC) on Linux systems by running various application-specific profiles, and it's enabled by default on Debian 10.

To hear them tell it per their page: "AppArmor is an effective and easy-to-use Linux application security system. AppArmor proactively protects the operating system and applications from external or internal threats, even zero-day attacks, by enforcing good behavior and preventing both known and unknown application flaws from being exploited....AppArmor supplements the traditional Unix discretionary access control (DAC) model by providing mandatory access control (MAC)."

Basically, AppArmor allows administrators to set a profile for each binaries defining what they are and are not allowed to do. This can limit the success of attacks.

## Discovery

Unfortunately there is no way to check what profiles are active from a non-privileged account. If a user has root access to a machine the `aa-status` command will show the active profiles (this list is from the example machine after successful elevation):

```shell-session
root@debian-privesc:~# aa-status
apparmor module is loaded.
20 profiles are loaded.
18 profiles are in enforce mode.
   /usr/bin/evince
   /usr/bin/evince-previewer
   /usr/bin/evince-previewer//sanitized_helper
   /usr/bin/evince-thumbnailer
   /usr/bin/evince//sanitized_helper
   /usr/bin/man
   /usr/lib/cups/backend/cups-pdf
   /usr/sbin/cups-browsed
   /usr/sbin/cupsd
   /usr/sbin/cupsd//third_party
   /usr/sbin/tcpdump
...
2 profiles are in complain mode.
   libreoffice-oopslash
   libreoffice-soffice
3 processes have profiles defined.
3 processes are in enforce mode.
   /usr/sbin/cups-browsed (502)
   /usr/sbin/cupsd (654)
   /usr/lib/cups/notifier/dbus (658) /usr/sbin/cupsd
0 processes are in complain mode.
0 processes are unconfined but have a profile defined.
```

### Unprivileged Users

An unprivileged user is allowed to check the existence of AppArmor via the `aa-status` command:

```shell-session
joe@debian-privesc:~$ aa-status
apparmor module is loaded.
You do not have enough privilege to read the profile set.
```

Unfortunately, as noted in the output, the unprivileged user is not allowed to see how AppArmor is protecting this particular machine. This means the main way for an unprivileged user to discover AppArmor's existence is through accidentally running into it.

Consider the scenario where a compromised (unprivileged) account has the following `sudo` capabilities:

```shell-session
joe@debian-privesc:~$ sudo -l
[sudo] password for joe: 
Matching Defaults entries for joe on debian-privesc:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User joe may run the following commands on debian-privesc:
    (ALL) /usr/bin/crontab -l, /usr/sbin/tcpdump, /usr/bin/apt-get
```

Obviously `crontab -l` can be ruled out for exploitation, but tcpdump seems promising. It even has it's own [sudo entry](https://gtfobins.github.io/gtfobins/tcpdump/#sudo) on GTFOBins. However attempting to run the command found there is unsuccessful:

```shell-session
joe@debian-privesc:~$ sudo tcpdump -ln -i lo -w /dev/null -W 1 -G 1 -z $TF -Z root
[sudo] password for joe: 
dropped privs to root
tcpdump: listening on lo, link-type EN10MB (Ethernet), capture size 262144 bytes
Maximum file limit reached: 1
1 packet captured
38 packets received by filter
0 packets dropped by kernel
compress_savefile: execlp(/tmp/tmp.v6dh1rsrnT, /dev/null) failed: Permission denied
```

Note the last line of the output that states "`Permission denied`" for part of the exploit chain.

Investigation into what caused this failure could start at the system log, specifically any entries pertaining to `tcpdump`:

{% code overflow="wrap" %}
```shell-session
joe@debian-privesc:~$ cat /var/log/syslog | grep tcpdump
Oct 29 14:24:15 debian-privesc kernel: [ 3205.049751] audit: type=1400 audit(1698607455.628:24): apparmor="DENIED" operation="exec" profile="/usr/sbin/tcpdump" name="/tmp/tmp.v6dh1rsrnT" pid=7100 comm="tcpdump" requested_mask="x" denied_mask="x" fsuid=0 ouid=1000
```
{% endcode %}

`apparmor="DENIED"` indicates that this command was interrupted because it was denied by AppArmor. Unfortunately, unless there is some exploit that also gets around AppArmor, it is likely worth abandoning this path and seeking a different one.
