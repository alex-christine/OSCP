---
description: Notes on using the server functionality of Netcat
---

# Server

## Reverse Shells

Netcat can be used to catch reverse shells by listening on a specific port

```shell
nc -lvnp {PORT}
```

Must be run as root for any port below and including 1024.

<table><thead><tr><th width="89">Flag</th><th>Description</th></tr></thead><tbody><tr><td><code>-l</code></td><td>Tells netcat to operate as a listener</td></tr><tr><td><code>-n</code></td><td>Disables DNS. Numeric IPs only</td></tr><tr><td><code>-p</code></td><td>Specifies the port for listening</td></tr><tr><td><code>-v</code></td><td>Verbose</td></tr></tbody></table>

* Basic Netcat shells are generally not particularly stable
  * Pressing `Ctrl + C` kills the whole thing
  * They are non-interactive
  * Often have strange formatting errors
  * Due to Netcat "shells" really being processes running inside a terminal, rather than being full, bona-fide terminals in their own right

### Stabilization Techniques

#### Utilize Python

Linux only.

After a basic Netcat shell has been obtained, the attacker can run these commands on the target machine to launch the Python-stabilized shell

```shell
python -c 'import pty;pty.spawn("/bin/bash")'
export TERM=xterm
```

At this point the shell can be backgrounded on the target using `Ctrl + Z`

The attacker can then run the following on their machine and run

```shell
stty raw -echo; fg
```

<table><thead><tr><th width="183">Command</th><th>Description</th></tr></thead><tbody><tr><td><code>stty raw echo</code></td><td>Sets I/O to standard device, turns off attacker's own terminal echo</td></tr><tr><td><code>fg</code></td><td>Foregrounds the terminal on attacker's machine</td></tr></tbody></table>

#### RLWrap

Linux & Windows

Program which gives user access to history, tab auto-completion and the arrow keys immediately upon receiving a shell.&#x20;

Some manual stabilization must be done in order to use `Ctrl+C` command in the terminal to stop commands on target (without exiting reverse shell)

This technique is particularly useful with Windows shells which are typically difficult to stabilize.

RLWrap is simply included before `nc` command when spawning the listener.

```shell
rlwrap nc -lvnp {PORT}
```
