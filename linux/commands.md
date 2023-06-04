---
description: List of useful Linux commands
---

# Commands

## General

<table><thead><tr><th width="221">Command</th><th>Action</th></tr></thead><tbody><tr><td><code>apropos &#x3C;command></code></td><td>Search the list of <code>man</code> pages for a particular keyword</td></tr><tr><td><code>man &#x3C;command></code></td><td>Get a manual for a command<br><br>Specify section of the manual with <code>man &#x3C;section> &#x3C;command></code></td></tr><tr><td><code>sudo &#x3C;command></code></td><td><em>SuperUser Do</em>. Elevates privileges of the command that follows to root.<br><br>Can be used with <code>-l</code> flag (<code>sudo -l</code>) in order to list all commands a user is able to run with root permissions</td></tr></tbody></table>

## Networking

<table><thead><tr><th width="209">Command</th><th>Action</th></tr></thead><tbody><tr><td><code>ifconfig</code></td><td>Used to configure a network interface</td></tr><tr><td><code>ss</code></td><td><strong>Socket statistics</strong> tool is a CLI command used to show network statistics</td></tr></tbody></table>

## Search

<table><thead><tr><th width="221">Command</th><th>Action</th></tr></thead><tbody><tr><td><code>locate &#x3C;search></code></td><td>Find the locations of files and directories in Kali using <code>&#x3C;search></code> term</td></tr><tr><td><code>which &#x3C;search></code></td><td>Searches through the directories that are defined in the <code>$PATH</code> environment variable for a given file name.<br><br>Used to search for executables.</td></tr><tr><td><code>find</code></td><td>Used to find files and directories. Can search based on type, name, size, etc. (<a href="https://linuxize.com/post/how-to-find-files-in-linux-using-the-command-line/">Examples</a>)<br><br>Most flexible and versatile tool search tool.</td></tr></tbody></table>

## Services

<table><thead><tr><th width="221">Command</th><th>Action</th></tr></thead><tbody><tr><td><code>systemctl</code></td><td>Controlling interface and inspection tool for the widely-adopted init system and service manager <code>systemd</code><br><br>List all available services with <code>systemctl list-unit-files</code></td></tr></tbody></table>

## Remote Access

<table><thead><tr><th width="210">Command</th><th>Action</th></tr></thead><tbody><tr><td><code>ssh</code></td><td>Used to remotely access a machine via CLI</td></tr></tbody></table>
