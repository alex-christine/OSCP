---
layout:
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# File Upload Vulnerabilities

Many web applications provide functionality to upload files. This section will cover how to identify, exploit, and leverage File Upload vulnerabilities to access the underlying system or execute code.

In general file upload vulnerabilities can be grouped into three categories:

1. Vulnerabilities enabling the upload of files that are **executable by the web application**
   * **Example:** If an attacker can upload a PHP script to a web server where PHP is enabled, they can then execute the script by accessing it via the browser or `curl`.
2. Vulnerabilities that must **combine the file upload mechanism with another vulnerability** (E.g. Directory Traversal)
   * **Example:** if the web application is vulnerable to Directory Traversal, one could use a relative path in the file upload request and try to overwrite files like `authorized_keys`.
   * Can also combine file upload mechanisms with XML External Entity (XXE) or [Cross Site Scripting](../cross-site-scripting/) (XSS) attacks.
     * **Example:** If a user is allowed to upload an avatar to a profile with an SVG file type, they may embed an XXE attack to display file contents or even execute code
3. Vulnerabilities that **rely on user interaction**
   * **Example:** If an attacker discovers an upload form for job applications, they can try to upload a CV in `.docx` format with malicious macros integrated.
