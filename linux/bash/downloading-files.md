---
description: Techniques used to download files via CLI
---

# Downloading Files

## `wget`

Downloads files using the HTTP/HTTPS and FTP protocol suites.

```bash
kali@kali:~$ wget -O report_wget.pdf https://www.offensive-security.com/reports/penetration-testing-sample-report-2013.pdf
--2018-01-28 20:30:04--  https://www.offensive-security.com/reports/penetration-testin
Resolving www.offensive-security.com (www.offensive-security.com)... 192.124.249.5
Connecting to www.offensive-security.com (www.offensive-security.com)|192.124.249.5|:4
HTTP request sent, awaiting response... 200 OK
Length: 27691955 (26M) [application/pdf]
Saving to: ‘report_wget.pdf’

report_wget.pdf     100%[===================>]  26.41M   766KB/s    in 28s     

2018-01-28 20:30:33 (964 KB/s) - ‘report_wget.pdf’ saved [27691955/27691955]
```

## `curl`

Tool to transfer data _to or from_ a server using a host of protocols including IMAP/S, POP3/S, SCP, SFTP, SMB/S, SMTP/S, TELNET, TFTP, and others.

Its most basic use case is very similar to `wget` (though `curl` has significantly more functionality in addition)

```bash
kali@kali:~$ curl -o report.pdf https://www.offensive-security.com/reports/penetration-testing-sample-report-2013.pdf
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 26.4M  100 26.4M    0     0  1590k      0  0:00:17  0:00:17 --:--:--  870k
```

## `axel`

Download accelerator that transfers a file from a FTP or HTTP server through multiple connections.

### Common Flags

<table><thead><tr><th width="114">Flag</th><th>Description</th></tr></thead><tbody><tr><td><code>-a</code></td><td>Used to request a more concise progress indicator</td></tr><tr><td><code>-n &#x3C;X></code></td><td>Specify the number (<code>X</code>) of connections to be used</td></tr><tr><td><code>-o</code></td><td>Send output to a specific file</td></tr></tbody></table>

