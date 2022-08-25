# File Transfer

Netcat can be used to transfer files between machines.

## Example

Transferring file (`wget.exe`) from an attacker's (Kali) machine to a target's (Windows) machine.

#### Target

Set up netcat listener with output redirected to file

```powershell
C:\Users\offsec> nc -nlvp 4444 > incoming.exe
listening on [any] 4444 ...
```

#### Attacker

Will push `wget.exe` to target machine over port 4444.

```bash
kali@kali:~$ locate wget.exe
/usr/share/windows-resources/binaries/wget.exe

kali@kali:~$ nc -nv 10.11.0.22 4444 < /usr/share/windows-resources/binaries/wget.exe
(UNKNOWN) [10.11.0.22] 4444 (?) open
```

#### Target

Incoming file is received by the listener.&#x20;

* There are no progress indicators in the download
  * Attacker will have to either
    1. Wait a bit and guess at download completion time
    2. Implement some other means of determining download completion
