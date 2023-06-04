---
description: Bash environment variable properties and some important call-outs
---

# Environment Variables

When opening a terminal window, a new Bash process, which has its own **environment variables**, is initialized. These variables are a form of global storage for various settings inherited by any applications that are run during that terminal session.

## Notable Environment Variables

<table><thead><tr><th width="153">Variable</th><th>Description</th></tr></thead><tbody><tr><td><code>$HOME</code></td><td>Current user's home directory</td></tr><tr><td><code>$PATH</code></td><td>Colon-separated (<code>:</code>) list of directory paths that Bash will search through whenever a command is run without a full path</td></tr><tr><td><code>$PWD</code></td><td>Current directory.<br><br>Value returned by <code>pwd</code> command.</td></tr><tr><td><code>$USER</code></td><td>Current User</td></tr><tr><td><code>$$</code></td><td>Not an environment variable per-se.<br><br><code>echo $$</code> will return the <strong>process ID of the current shell</strong> session</td></tr></tbody></table>

This is not a complete list. In order to see a complete list of active environment variables in a session run the `env` command.

## Using Environment Variables

Current value of an environment variable can be viewed with the `echo` command.

```bash
$ echo $PATH        # displays $PATH
```

## Defining Environment Variables

Environment variables can be defined with the `export` command.

E.g. if a user wanted to create a variable representing a target's IP address

```bash
$ export IP_T=185.2.192.168    # Creates a variable holding target's IP
$ ping -c 2 $IP_T              # Can then be used instead of typing IP out
```

`export` makes the variable available in any sub-processes that may be spawned by the shell. Variables can be created without the `export` command but they will only be available in the current shell session.
