---
description: Leveraging directory traversal vulnerabilities into actionable attacks
---

# Directory Traversal

**Directory traversal** vulnerabilities, also known as **path traversal** vulnerabilities, allow attackers to gain unauthorized access to files within an application or files normally not accessible through a web interface, such as those outside the application's web root directory.

This vulnerability occurs when input is poorly validated, subsequently granting an attacker the ability to manipulate file paths with `../` or `..\` characters.

These attacks can expose sensitive information but they _do not execute code on the application server_. On certain application servers written in specific programming languages, directory traversal attacks can be used to help facilitate file inclusion attacks.

## Identifying Directory Traversal

A search for directory traversals begins with the examination of URL query strings and form bodies in search of values that appear as file references, including the most common indicator: file extensions in URL query strings.

Once some likely candidates have been identified, the entry points can be probed to check if files generally accessible to all users can be read (e.g. `/etc/passwd` or `C:\Windows\System32\drivers\etc\hosts`).

Once directory traversal has been verified, it is worth exploring if the web application is running as root or admin. The easiest way is to check if files normally accessible only with those permissions can be read via the directory traversal (e.g. `/etc/shadow`).

## Linux

In Linux systems, the `/var/www/html/` directory is often used as the web root. When a web application displays a page, `http://example.com/file.html` for example, it will try to access `/var/www/html/file.html`. The HTTP link doesn't contain any part of the path except the filename because the web root also serves as a base directory for a web server.

If a web application is vulnerable to directory traversal, a user may access files outside of the web root by using relative paths, thus accessing sensitive files like SSH private keys (usually at user's home directory `/.ssh`) or configuration files.

### Standard Attack Vector

In Linux systems, a pretty standard vector for directory traversal is to:

1. List the users of the system by displaying the contents of `/etc/passwd`
2. Check for private keys in the users' home directories
   * The default key name when generating a key is `id_rsa` therefore it is relatively common to find a key called `/home/user/.ssh/id_rsa`
3. Use any found private keys to access the system via SSH

While these steps will not guarantee a compromise, they are a good general understanding of the attack steps.

## Windows

On Windows, one can use the file `C:\Windows\System32\drivers\etc\hosts` to test directory traversal vulnerabilities, as it is readable by all local users regardless of permissions.

In general, it is more difficult to leverage a directory traversal vulnerability for system access on Windows than Linux. Unfortunately there is no "standard" attack path as there is for Linux systems.

Additionally, sensitive files are often not easily found on Windows without being able to list the contents of directories. This means to identify files containing sensitive information, one needs to closely examine the web application and collect information about the web server, framework, and programming language.

Once information is gathered about the running application or service, an attacker can research paths leading to sensitive files.

#### Example

For example, if one learns that a target system is running the _Internet Information Services_ (IIS) web server, they can research its log paths and web root structure.&#x20;

Reviewing the Microsoft documentation, they would learn that the logs are located at `C:\inetpub\logs\LogFiles\W3SVC1\`.&#x20;

Another file always worth checking when the target is running an IIS web server is `C:\inetpub\wwwroot\web.config`, which may contain sensitive information like passwords or usernames.

### Additional Considerations

Windows uses backslashes (`\`) instead of forward slashes (`/`) for file paths. Therefore, `..\` is an important alternative to `../` on Windows targets.

While RFC 1738 specifies to always use slashes in a URL, one may encounter web applications on Windows which are only vulnerable to directory traversal using backslashes. Therefore, **one should always try to leverage both forward slashes and backslashes** when examining a potential directory traversal vulnerability in a web application running on Windows

## Encoding Special Characters

Special characters will often be filtered by web servers (or other software). E.g. `../` is often exploited in relative-path attacks, as a result this sequence is often filtered by either the web server, web application firewalls, or the web application itself. In these instances it will be necessary to encode `../` in some way as to slip it past the filters.

Consider the following example `curl` command run against a sample web server running Apache 2.4.9:

```shell-session
kali@kali:~$ curl http://192.168.50.16/cgi-bin/../../../../etc/passwd

<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>404 Not Found</title>
</head><body>
<h1>Not Found</h1>
<p>The requested URL was not found on this server.</p>
</body></html>


kali@kali:~$ curl http://192.168.50.16/cgi-bin/../../../../../../../../../../etc/passwd

<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>404 Not Found</title>
</head><body>
<h1>Not Found</h1>
<p>The requested URL was not found on this server.</p>
</body></html>
```

Despite making a couple attempts with varying numbers of `../` the path is not found. In this case it is because the server is filtering `../` and thus not finding the `etc/passwd` file as it is looking in the web server's subdirectories.

### URL Encoding

One of the most obvious and basic method for attempting to slip past the filters is [URL encoding](https://www.w3schools.com/tags/ref\_urlencode.asp) (sometimes referred to as percent encoding).&#x20;

This is normally used to encode non-ASCII characters in URLs for HTTP/S. However, if the writer of the web server used some form of text matching that only checks for "`../`" as a whole substring it could be evaded.

Consider the example from above, for the next attempt the "`.`" characters will be replaced with "`%2e`"

```shell-session
kali@kali:~$ curl http://192.168.50.16/cgi-bin/%2e%2e/%2e%2e/%2e%2e/%2e%2e/etc/passwd

root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
...
_apt:x:100:65534::/nonexistent:/usr/sbin/nologin
alfred:x:1000:1000::/home/alfred:/bin/bash
```

In this particular example that was all that was needed to bypass the filters. Unfortunately it will not always be this easy but the above is a basic illustration of the concept.
