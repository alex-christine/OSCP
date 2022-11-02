# Nikto

**Nikto** is a highly configurable Open Source web server scanner that tests for thousands of dangerous files and programs, vulnerable server versions and various server configuration issues. It performs well, but is not designed for stealth as it will send many requests and embed information about itself in the _User-Agent_ header.

Nikto can scan multiple servers and ports and will scan as many pages as it can find. Nikto is especially useful for catching low-hanging fruit, reporting non-standard server headers, and catching server configuration errors.

#### Example

```bash
$ nikto -host=https://www.megacorpone.com/ -maxtime=30s
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          149.56.244.87
+ Target Hostname:    www.megacorpone.com
+ Target Port:        443
---------------------------------------------------------------------------
+ SSL Info:        Subject:  /CN=www.megacorpone.com
                   Ciphers:  TLS_AES_256_GCM_SHA384
                   Issuer:   /C=US/O=Let's Encrypt/CN=R3
+ Start Time:         2022-10-17 16:40:23 (GMT-6)
---------------------------------------------------------------------------
+ Server: Apache/2.4.38 (Debian)
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The site uses SSL and the Strict-Transport-Security HTTP header is not defined.
+ The site uses SSL and Expect-CT header is not present.
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ ERROR: Host maximum execution time of 30 seconds reached
+ Scan terminated:  0 error(s) and 5 item(s) reported on remote host
+ End Time:           2022-10-17 16:40:54 (GMT-6) (31 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
```
