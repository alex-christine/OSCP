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

| Argument             | Description                                                       |
| -------------------- | ----------------------------------------------------------------- |
| `TCP4-LISTEN:{PORT}` | Specifies an IPv4 listener and which port it is operating on      |
| `fork`               | Creates a child process once a connection is made to the listener |
| `file`               | Specifies the name of a file to be transferred                    |

#### Receiving Machine

```powershell
C:\Users\offsec> socat TCP4:{IP}:{PORT} file:received_example.txt,create

C:\Users\offsec> type received_example.txt
"Text"
```

| Argument           | Description                                         |
| ------------------ | --------------------------------------------------- |
| `TCP4:{IP}:{PORT}` | Specifies the location to reach out to for transfer |
| `file`             | Specifies the location of the file to be saved      |
