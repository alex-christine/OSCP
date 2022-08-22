---
description: List of useful Linux commands
---

# Commands

## General

| Command             | Action                                                                                                                                                                                                                               |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `apropos <command>` | Search the list of `man` pages for a particular keyword                                                                                                                                                                              |
| `man <command>`     | <p>Get a manual for a command<br><br>Specify section of the manual with <code>man &#x3C;section> &#x3C;command></code></p>                                                                                                           |
| `sudo <command>`    | <p><em>SuperUser Do</em>. Elevates privileges of the command that follows to root.<br><br>Can be used with <code>-l</code> flag (<code>sudo -l</code>) in order to list all commands a user is able to run with root permissions</p> |

## Networking

| Command    | Action                                                                      |
| ---------- | --------------------------------------------------------------------------- |
| `ifconfig` | Used to configure a network interface                                       |
| `ss`       | **Socket statistics** tool is a CLI command used to show network statistics |

## Search

| Command           | Action                                                                                                                                                                                                                                           |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `locate <search>` | Find the locations of files and directories in Kali using `<search>` term                                                                                                                                                                        |
| `which <search>`  | <p>Searches through the directories that are defined in the <code>$PATH</code> environment variable for a given file name.<br><br>Used to search for executables.</p>                                                                            |
| `find`            | <p>Used to find files and directories. Can search based on type, name, size, etc. (<a href="https://linuxize.com/post/how-to-find-files-in-linux-using-the-command-line/">Examples</a>)<br><br>Most flexible and versatile tool search tool.</p> |

## Services

| Command     | Action                                                                                                                                                                                                                                |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `systemctl` | <p>Controlling interface and inspection tool for the widely-adopted init system and service manager <code>systemd</code><br><code></code><br><code></code>List all available services with <code>systemctl list-unit-files</code></p> |

## Remote Access

| Command | Action                                    |
| ------- | ----------------------------------------- |
| `ssh`   | Used to remotely access a machine via CLI |
