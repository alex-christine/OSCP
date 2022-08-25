---
description: Techniques to configure environment variables of Bash
---

# Configuring Environment

## History

### `HISTCONTROL`

The `HISTCONTROL` variable defines whether or not to remove duplicate commands, commands that begin with spaces from the history, or both. By default, both are removed but one may find it more useful to only omit duplicates.

```bash
kali@kali:~$ export HISTCONTROL=ignoredups
```

### `HISTIGNORE`

Used for filtering basic commands out that are run frequently. Acts as a list of commands that will not be included in history files.

```bash
kali@kali:~$ export HISTIGNORE="&:ls:[bf]g:exit:history"
```

### `HISTTIMEFORMAT`

Controls date and/or time stamps in the output of the history command.

```bash
kali@kali:~/test$ export HISTTIMEFORMAT='%F %T '

kali@kali:~/test$ history
    1  2018-02-12 13:37:33 export HISTIGNORE="&:ls:[bf]g:exit:history"
    2  2018-02-12 13:37:38 mkdir test
    3  2018-02-12 13:37:40 cd test
    4  2018-02-12 13:37:43 pwd
    5  2018-02-12 13:37:51 export HISTTIMEFORMAT='%F %T '
```

In this example, %F (Year-Month-Day ISO 8601 format) and %T (24-hour time) were used0. Other formats can be found in the `strftime` man page.

## Alias

An alias is a string we can define that replaces a command name. Aliases are useful for replacing commonly-used commands and switches with a shorter command, or alias, that the user defines.

```bash
kali@kali:~$ alias lsa='ls -la'

kali@kali:~$ lsa
total 8308

........
-rw-------  1 kali kali     5542 Jan 22 09:56 .bash_history
-rw-r--r--  1 kali kali     3391 Apr 25  2017 .bashrc
drwx------  9 kali kali     4096 Oct  2 21:29 .cache
........
```

In order to remove a previously-created alias, the `unalias` command can be used.

```bash
kali@kali:~$ unalias lsa
```

## Persistent Customization

The behavior of interactive shells in Bash is determined by the system-wide `bashrc` file located in `/etc/bash.bashrc`. The system-wide Bash settings can be overridden by editing the `.bashrc` file located in any user's home directory.

The `.bashrc` script is executed any time that user logs in. Since this file is a shell script, one can insert any command that could be executed from the command prompt (notice the alias from the previous examples could be included in line 16 to make it a persistent change).

{% code lineNumbers="true" %}
```bash
kali@kali:~$ cat ~/.bashrc
# ~/.bashrc: executed by bash(1) for non-login shells.
# see /usr/share/doc/bash/examples/startup-files (in the package bash-doc)
# for examples
...
# for setting history length see HISTSIZE and HISTFILESIZE in bash(1)
HISTSIZE=1000
HISTFILESIZE=2000

# enable color support of ls and also add handy aliases
if [ -x /usr/bin/dircolors ]; then
    test -r ~/.dircolors && eval "$(dircolors -b ~/.dircolors)" || eval "$(dircolors -
    alias ls='ls --color=auto'
...

alias lsa='ls -la'
```
{% endcode %}

