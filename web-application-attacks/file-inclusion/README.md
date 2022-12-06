---
description: Leveraging file inclusion vulnerabilities
---

# File Inclusion

**File inclusion** vulnerabilities allow an attacker to include a file into the application's running code.

These vulnerabilities are most commonly found to affect web applications that rely on a scripting run time. This issue is caused when an application builds a path to executable code using an attacker-controlled variable in a way that allows the attacker to control which file is executed at run time.

Successful exploitation of a file inclusion vulnerability will _result in remote code execution_ on the web server that runs the affected web application.

## Local File Inclusion

Occur when the included file is loaded from the same web server. This issue can still lead to remote code execution by including a file that contains attacker-controlled data such as the web server's access logs.

## Remote File Inclusion

Occurs when the web application downloads and executes a remote file. These remote files are usually obtained in the form of an HTTP or FTP URI as a user-supplied parameter to the web application.

## Discovering File Inclusion

File inclusions can be discovered in the same way as directory traversals. First an attacker must find a parameter that can be manipulated to give access to a file (local or remote). File inclusion takes things a step further than directory traversals by attempting to execute the contents of the file within the application.

Once discovered, a file inclusion vulnerability should always be checked to see if it allows remote file inclusion as that is much easier to exploit. It is now less likely to find RFI vulnerabilities since the default configuration for modern PHP versions disables remote URL includes. Can use Netcat, Apache, or Python to handle the requests. It is worth testing on different ports since even a valid RFI vulnerability's requests are still subject to the victim's internal routing or firewall rules.
