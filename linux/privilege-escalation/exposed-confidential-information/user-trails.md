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

# User Trails

Attackers are often time-constrained during our engagements. For this reason, they should focus their efforts first on low-hanging fruit.

One such target is users' history files. These files often hold clear-text user activity that might include sensitive information such as passwords or other authentication material.

This page will include examples that again leverage a machine (`192.168.210.217`) to which an attacker has gained access through a compromised account (credentials `joe:offsec`) and is able to interact via SSH.

## Environment Variables

The `env` command will list any environment variables that are set for the current bash (or zsh) session. This can sometimes give an indication of some modifications that were made either by a user or administrator.

In the case of the example machine:

{% code lineNumbers="true" %}
```shell-session
joe@debian-privesc:~$ env
SHELL=/bin/bash
PWD=/home/joe
LOGNAME=joe
XDG_SESSION_TYPE=tty
HOME=/home/joe
LANG=en_US.UTF-8
LS_COLORS=rs=0:di=01;34:ln=01;36:mh=00:pi=40;33:so=01;35:do=01;35:bd=40;33;01:cd=40;33;01:or=40;31;01:mi=00:su=37;41:sg=30;43:ca=30;41:tw=30;42:ow=34;42:st=37;44:ex=01;32:*.tar=01;31:*.tgz=01;31:*.arc=01;31:*.arj=01;31:*.taz=01;31:*.lha=01;31:*.lz4=01;31:*.lzh=01;31:*.lzma=01;31:*.tlz=01;31:*.txz=01;31:*.tzo=01;31:*.t7z=01;31:*.zip=01;31:*.z=01;31:*.dz=01;31:*.gz=01;31:*.lrz=01;31:*.lz=01;31:*.lzo=01;31:*.xz=01;31:*.zst=01;31:*.tzst=01;31:*.bz2=01;31:*.bz=01;31:*.tbz=01;31:*.tbz2=01;31:*.tz=01;31:*.deb=01;31:*.rpm=01;31:*.jar=01;31:*.war=01;31:*.ear=01;31:*.sar=01;31:*.rar=01;31:*.alz=01;31:*.ace=01;31:*.zoo=01;31:*.cpio=01;31:*.7z=01;31:*.rz=01;31:*.cab=01;31:*.wim=01;31:*.swm=01;31:*.dwm=01;31:*.esd=01;31:*.jpg=01;35:*.jpeg=01;35:*.mjpg=01;35:*.mjpeg=01;35:*.gif=01;35:*.bmp=01;35:*.pbm=01;35:*.pgm=01;35:*.ppm=01;35:*.tga=01;35:*.xbm=01;35:*.xpm=01;35:*.tif=01;35:*.tiff=01;35:*.png=01;35:*.svg=01;35:*.svgz=01;35:*.mng=01;35:*.pcx=01;35:*.mov=01;35:*.mpg=01;35:*.mpeg=01;35:*.m2v=01;35:*.mkv=01;35:*.webm=01;35:*.ogm=01;35:*.mp4=01;35:*.m4v=01;35:*.mp4v=01;35:*.vob=01;35:*.qt=01;35:*.nuv=01;35:*.wmv=01;35:*.asf=01;35:*.rm=01;35:*.rmvb=01;35:*.flc=01;35:*.avi=01;35:*.fli=01;35:*.flv=01;35:*.gl=01;35:*.dl=01;35:*.xcf=01;35:*.xwd=01;35:*.yuv=01;35:*.cgm=01;35:*.emf=01;35:*.ogv=01;35:*.ogx=01;35:*.aac=00;36:*.au=00;36:*.flac=00;36:*.m4a=00;36:*.mid=00;36:*.midi=00;36:*.mka=00;36:*.mp3=00;36:*.mpc=00;36:*.ogg=00;36:*.ra=00;36:*.wav=00;36:*.oga=00;36:*.opus=00;36:*.spx=00;36:*.xspf=00;36:
SSH_CONNECTION=192.168.45.201 42164 192.168.210.214 22
XDG_SESSION_CLASS=user
TERM=xterm-256color
SCRIPT_CREDENTIALS=lab
USER=joe
SHLVL=1
XDG_SESSION_ID=29
XDG_RUNTIME_DIR=/run/user/1000
SSH_CLIENT=192.168.45.201 42164 22
PATH=/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games
DBUS_SESSION_BUS_ADDRESS=unix:path=/run/user/1000/bus
MAIL=/var/mail/joe
SSH_TTY=/dev/pts/0
_=/usr/bin/env
```
{% endcode %}

While most of this output is standard, the entry at line 12 (`SCRIPT_CRED...`) appears to be a custom addition.

## Dotfiles

On Linux systems, applications frequently store user-specific configuration files and subdirectories within a user's home directory. These files are often called _dotfiles_ because they are prepended with a period. The prepended dot character instructs the system not to display these files when inspecting by basic listing commands.

One example of a dotfile is `.bashrc`. The `.bashrc` bash script is executed when a new terminal window is opened from an existing login session or when a new shell instance is started from an existing login session. From inside this script, additional environment variables can be specified to be automatically set whenever a new user's shell is spawned.

Sometimes system administrators store credentials inside environment variables as a way to interact with custom scripts that require authentication.

In the case of the example machine the .bashrc file contains a refernce to the environment variable of interest found [above](user-trails.md#environment-variables) (file has been shortened to only interesting sections):

{% code title=".bashrc" lineNumbers="true" %}
```
# ~/.bashrc: executed by bash(1) for non-login shells.
# see /usr/share/doc/bash/examples/startup-files (in the package bash-doc)
# for examples

# If not running interactively, don't do anything
case $- in
    *i*) ;;
      *) return;;
esac

# don't put duplicate lines or lines starting with space in the history.
# See bash(1) for more options
export SCRIPT_CREDENTIALS="lab"
HISTCONTROL=ignoreboth
```
{% endcode %}

In this case it seems the credentials were placed right at the front of the file and it was easy to find. Given the length of the file it can sometimes be helpful to give it a first pass with `grep`:

```shell-session
joe@debian-privesc:~$ cat .bashrc | grep -i export
export SCRIPT_CREDENTIALS="lab"
#export GCC_COLORS='error=01;31:warning=01;35:note=01;36:caret=01;32:locus=01:quote=01'
```

Regardless, once found the credentials can be tried against the `root` account:

```shell-session
joe@debian-privesc:~$ su - root
Password: 
root@debian-privesc:~# whoami
root
```

While this was all convenient for the example, it usually isn't that easy.

## Sudo Permissions

As discussed in the [manual enumeration](../enumeration/manual-enumeration.md#sudo-permissions) section, existing `sudo` permissions (listed via `sudo -l`) can be helpful for elevating privileges.
