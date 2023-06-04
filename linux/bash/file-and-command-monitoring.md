---
description: Techniques and commands for monitoring files
---

# File and Command Monitoring

## `tail`

Most common use of `tail` is using it to monitor log files and such.

```bash
$ sudo tail -f /var/log/apache2/access.log 
127.0.0.1 - - [02/Feb/2018:12:18:14 -0500] "GET / HTTP/1.1" 200 3380 "-" "Mozilla/5.0 (X11; Linux x86_64; rv:52.0) Gecko/20100101 Firefox/52.0"
127.0.0.1 - - [02/Feb/2018:12:18:14 -0500] "GET /icons/openlogo-75.png HTTP/1.1" 200 6040 "http://127.0.0.1/" "Mozilla/5.0 (X11; Linux x86_64; rv:52.0) Gecko/20100101 Firefox/52.0"
127.0.0.1 - - [02/Feb/2018:12:18:15 -0500] "GET /favicon.ico HTTP/1.1" 404 500 "-" "Mozilla/5.0 (X11; Linux x86_64; rv:52.0) Gecko/20100101 Firefox/52.0"
```

### Common Flags

<table><thead><tr><th width="126">Flag</th><th>Description</th></tr></thead><tbody><tr><td><code>-f</code></td><td>Follow. Used to continuously update the output as the target file grows</td></tr><tr><td><code>-nX</code></td><td>Constrains output to <code>X</code> lines</td></tr></tbody></table>

## `watch`

Used to run a designated command at regular intervals.&#x20;

By default, it runs every two seconds but we can specify a different interval by using the `-n X` option to have it run every "X" number of seconds.

For example a user could check the list of logged in users (via the `w` command) every 5 seconds with:

```bash
$ watch -n 5 w

............

Every 5.0s: w                                     kali: Tue Jan 23 21:06:03 2018

 21:06:03 up 7 days,  3:54,  1 user,  load average: 0.18, 0.09, 0.03
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
kali     tty2     :0               16Jan18  7days 16:29   2.51s /usr/bin/python
```

`Ctrl + C` is used to terminate the watch command and return the terminal to normal functionality.

