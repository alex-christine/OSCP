---
description: Instructions for managing processes in Linux/Bash environment
---

# Managing Processes

The Linux kernel manages multitasking through the use of processes. The kernel maintains information about each process to help keep things organized, and each process is assigned a number called a **process ID (PID)**.

The Linux shell also introduces the concept of **jobs** to ease the user's workflow during a terminal session.&#x20;

As an example, `cat error.txt | wc -m` is a pipeline of two processes, which the shell considers a single job. Job control refers to the ability to selectively suspend the execution of jobs and resume their execution at a later time.

## Backgrounding Processes

Backgrounding processes allows a user to regain interactive access with the terminal while complex or longer-running jobs execute. If a long-running job is not backgrounded the console will be occupied, and thus unusable, until the job completes.

The quickest way to background a process is to append an ampersand (`&`) to the end of the command to send it to the background immediately after it starts.

```bash
$ ping -c 400 localhost > ping_results.txt &
```

Foreground processes can be killed with `Ctrl + C` or suspended with `Ctrl + Z`. Suspended jobs can be sent to the background with `bg` command.

```bash
kali@kali:~$ ping -c 400 localhost > ping_results.txt
^Z
[1]+  Stopped                 ping -c 400 localhost > ping_results.txt

kali@kali:~$ bg
[1]+ ping -c 400 localhost > ping_results.txt
kali@kali:~$ 
```

The job is now running in the background.

## Job Control

The built-in `jobs` utility lists the jobs that are running in the current terminal session, while `fg` returns a job to the foreground.

```bash
kali@kali:~$ ping -c 400 localhost > ping_results.txt
^Z
[1]+  Stopped                 ping -c 400 localhost > ping_results.txt

kali@kali:~$ find / -name sbd.exe
^Z
[2]+  Stopped                 find / -name sbd.exe

kali@kali:~$ jobs
[1]-  Stopped                 ping -c 400 localhost > ping_results.txt
[2]+  Stopped                 find / -name sbd.exe

kali@kali:~$ fg %1
ping -c 400 localhost > ping_results.txt
^C

kali@kali:~$ jobs
[2]+  Stopped                 find / -name sbd.exe

kali@kali:~$ fg
find / -name sbd.exe                        # Echoes fg command
/usr/share/windows-resources/sbd/sbd.exe    # Output
```

Note the `%1` in the first `fg` command.&#x20;

The `%` character followed by a JobID represents a job specification. The JobID can be a process ID (PID) number or you can use one of the following symbol combinations:

* `%Number` : Refers to a job number such as %1 or %2
* `%String` : Refers to the beginning of the suspended command's name such as:
  * `%ping`
  * `%commandNameHere`
* `%+` OR `%%` : Refers to the current job
* `%-` : Refers to the previous job

## Process Control

### Process Status (`ps`)

`ps` lists processes system-wide, not only for the current terminal session. This utility is considered a standard on Unix-like OSes and its name is so well-recognized that even on Windows PowerShell, `ps` is a predefined command alias for the `Get-Process` cmdlet, which essentially serves the same purpose.

As a penetration tester, **one of the first things to check after obtaining remote access to a system is to understand what software is currently running on the compromised machine**. This could help us elevate our privileges or collect additional information in order to acquire further access into the network.

#### Common Flags

| Flag        | Description                                        |
| ----------- | -------------------------------------------------- |
| `-C <name>` | Select processes by command name                   |
| `-e`        | Select all processes                               |
| `-f`        | Display full format listing (UID, PID, PPID, etc.) |
|             |                                                    |

#### Examples

Searching for a process spawned by the `leafpad` command.

```bash
kali@kali:~$ ps -fC leafpad
UID         PID   PPID  C STIME TTY          TIME CMD
kali       1307    938  0 10:57 ?        00:00:00 leafpad
```

### `kill`

Used to kill a specific process without GUI interaction.

Sends a specific signal to the process in order to end. The default signal used is `SIGTERM` (request termination).

* A full list of signals kill can send is available with `kill -l`
  * The signals are listed with numbers and can then be used `kill -<NUM>`

The PID of the process is passed as an argument to `kill`.

```bash
kali@kali:~$ kill 1307    # Kills leafpad process from previous example
```
