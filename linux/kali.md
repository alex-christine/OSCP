---
description: Information about the Kali distribution
---

# Kali

## Overview

Kali is a Debian-based Linux distribution with a penetration testing focus. It comes with many common tools preinstalled.

## Services

The default Kali installation ships with several services preinstalled, such as SSH, HTTP, MySQL, etc. Consequently, these services would load at boot time, which would result in Kali exposing several open ports by default.

Kali deals with this issue by updating its settings to **prevent network services from starting at boot** time.

Kali also **contains a mechanism to both whitelist and blacklist various services**.

## Installing and Removing Tools

This section is applicable to all Debian-based distros and not just Kali.

### Advanced Package Tool (APT)

Set of tools that helps manage packages, or applications, on a Debian-based system. Can use APT to install and remove applications, update packages, and even upgrade the entire system.

The magic of APT lies in the fact that it is a complete package management system that installs or removes the requested package by recursively satisfying its requirements and dependencies.

#### Update

```bash
$ sudo apt update
```

Information regarding APT packages is cached locally to speed up any sort of operation that involves querying the APT database.

It is **always good practice to update the list of available packages**, including information related to their versions, descriptions, etc. Update is particularly useful just prior to an `upgrade` command. This can be thought of as the "staging" command for updating tools.

#### Upgrade

```bash
$ sudo apt upgrade
```

After the APT database has been updated, we can upgrade the installed packages and core system to the latest versions using the `apt upgrade` command

A single package can be upgraded with `apt upgrade <package>`

The `-y` flag can be used to automatically download material (normally APT asks if it is okay to download files and store them requiring user interaction during install).

#### Install

```bash
$ sudo apt install <package>
```

Used to add a package to the system.

#### Uninstall

```bash
$ sudo apt remove --purge <package>
```

Used to remove a package from the system. The `--purge` flag removes all the leftovers that remove leaves by default (e.g. package configuration files)

#### Distro Upgrade

```bash
$ sudo apt dist-upgrade
```

Upgrades the Linux kernel as opposed to the packages on the system.

Package and kernel upgrades can be combined with `sudo apt full-upgrade`

#### Cache Search

```bash
$ apt-cache search <keyword>
```

Search and display the information about the available packages from the internet repositories. This can be used to find the correct package name based on a keyword.&#x20;

E.g. to install Apache one could `apt-cache search apache` and determine that `apache2` is the correct APT package name.

#### Show

```bash
$ apt show <package>
```

Show package information.

### Debian Package Manager (DPKG)

Core tool used to install a package (either directly or indirectly, through APT).

Preferred to APT when working offline as it does not require an internet connection.

dpkg **will not install any dependencies** a package may require.

#### Install

```bash
$ sudo dpkg -i <path_to_package.deb>
```

The `-i` or `--install` flags are used, in conjunction with a path to a local _.deb_ package file, in order to install a package.
