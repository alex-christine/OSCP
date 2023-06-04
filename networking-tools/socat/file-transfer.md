---
description: Socat's file transfer utilities
---

# File Transfer

Socat's file transfer services are more advanced and offer more utility than their Netcat counterparts.

## Example

Transferring a file to a Windows machine from a Kali machine

#### Transferring Machine

```bash
kali@kali:~$ sudo socat TCP4-LISTEN:{PORT},fork file:example.txt
```

<table><thead><tr><th width="231">Argument</th><th>Description</th></tr></thead><tbody><tr><td><code>TCP4-LISTEN:{PORT}</code></td><td>Specifies an IPv4 listener and which port it is operating on</td></tr><tr><td><code>fork</code></td><td>Creates a child process once a connection is made to the listener</td></tr><tr><td><code>file</code></td><td>Specifies the name of a file to be transferred</td></tr></tbody></table>

#### Receiving Machine

```powershell
C:\Users\offsec> socat TCP4:{IP}:{PORT} file:received_example.txt,create

C:\Users\offsec> type received_example.txt
"Text"
```

<table><thead><tr><th width="211">Argument</th><th>Description</th></tr></thead><tbody><tr><td><code>TCP4:{IP}:{PORT}</code></td><td>Specifies the location to reach out to for transfer</td></tr><tr><td><code>file</code></td><td>Specifies the location of the file to be saved</td></tr></tbody></table>
