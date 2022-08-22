---
description: Information on various Linux services
---

# Services

## Overview

A service is a program that runs in the background outside the interactive control of system users as they lack an interface.

In systems like Unix or Linux, the services are also known as **daemons**.&#x20;

* Sometimes the name of these services or daemons ends with the letter d
  * E.g. `sshd` is the name of the service that handles SSH.

### Listing Services

Command to list all services `sudo systemctl list-unit-files --type service --all`

#### Project States

| Status      | Meaning                                                                                   |
| ----------- | ----------------------------------------------------------------------------------------- |
| `enabled`   | Currently running services                                                                |
| `disabled`  | Services that are not currently running but can be activated at any time with no problems |
| `masked`    | Masked services will not run unless the `masked` property is removed from them            |
| `static`    | Service will only be used if another service or unit needs it                             |
| `generated` | Services generated through a SysV or LSB initscript with systemd generator                |

In order to see all running processes use `sudo systemctl | grep running`

### Managing Services

`systemctl` provides the functionality for controlling services. It has several verbs that can be used in the form `sudo systemctl <verb> <service>` where `<service>` is the name of the service being operated upon.

#### Verbs for `systemctl`

| Verb      | Action                                          |
| --------- | ----------------------------------------------- |
| `start`   | Start a service                                 |
| `stop`    | Stop a service                                  |
| `status`  | Display the current status of service           |
| `enable`  | Set a service to run at boot time               |
| `disable` | Remove a service from those loaded at boot time |

## SSH Service

Most commonly used to remotely access a computer, using a secure, encrypted protocol.&#x20;

The SSH service is **TCP** based and listens by default on **port 22**.

Started with `sudo systemctl start ssh`

Can be validated as running with `sudo ss -antlp | grep ssh`

## HTTP Service

&#x20;Apache HTTP often is used during a penetration test for things such as

* Hosting a site
* Providing a platform for downloading exploits to victim machines

Started with `sudo systemctl start apache2`

Can be validated as running with `sudo ss -antlp | grep apache`

