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
