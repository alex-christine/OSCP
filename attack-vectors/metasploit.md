---
layout:
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# Metasploit

[Metasploit](https://www.metasploit.com/) is so useful it gets its own page here. It is owned and maintained by Rapid7 but the code is open-source. The **Metasploit Framework** is a tool for developing and executing exploit code against a remote target machine. It is installed by default on Kali.

Metasploit is a modular framework. A complete list of available modules can be found [on their website](https://docs.metasploit.com/docs/modules.html).

## Metasploit Console

The Metasploit console can be launched via the `msfconsole` command.

It is recommended to always update Metasploit prior to launching a session using `apt`:

```bash
sudo apt update; sudo apt install metasploit-framework -y
```

* If using a version of Metasploit that was not installed with `apt`, the `msfupdate` command can be used to update

From here the console can be launched and will appear like this:

<figure><img src="../.gitbook/assets/Metasploit-msfconsole.png" alt=""><figcaption><p>Metasploit Console launch </p></figcaption></figure>

Once launched all commands can be listed with the `help` or `?` commands.
