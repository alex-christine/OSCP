---
description: Techniques for automating DNS lookups on Linux
---

# Automating Lookups

Compare `host`'s output for a domain that exists and one that does not:

```bash
kali@kali:~$ host www.megacorpone.com
www.megacorpone.com has address 38.100.193.76

kali@kali:~$ host idontexist.megacorpone.com
Host idontexist.megacorpone.com not found: 3(NXDOMAIN)
```

The difference in results can be leveraged into automated scans for DNS enumeration.

## Forward Lookup Brute Force

Brute force is a trial-and-error technique that seeks to find valid information, including directories on a webserver, username and password combinations, or in this case, valid DNS records.

By using a _wordlist_ that contains common hostnames, one can attempt to guess DNS records and check the response for valid hostnames.

Forward lookups **transform a hostname into an IP address**. Therefore the wordlist should be a list of common/expected hostnames.

#### Example

1. Construct a simple word list&#x20;
   * For real enumeration use larger, generated wordlists (E.g. `seclists`)
2. Iterate through `list.txt` and try each item as a prefix to `megacorpone.com`

```bash
kali@kali:~$ cat list.txt
www
ftp
mail
owa
proxy
router

kali@kali:~$ for ip in $(cat list.txt); do host $ip.megacorpone.com; done
www.megacorpone.com has address 38.100.193.76
Host ftp.megacorpone.com not found: 3(NXDOMAIN)
mail.megacorpone.com has address 38.100.193.84
Host owa.megacorpone.com not found: 3(NXDOMAIN)
Host proxy.megacorpone.com not found: 3(NXDOMAIN)
router.megacorpone.com has address 38.100.193.71
```

This demonstrates a simple brute force lookup. Additional refining could be done to only output lines with valid IPs for example.

## Reverse Lookup Brute Force

To continue the example from above, the forward brute force lookup revealed a cluster of IPs in the `38.100.193.0/24` range. With that in mind, it is reasonable to assume the site administrator would have configured all other resources in that range (or at least have `PTR` records in that range).

#### Example

In this example, the approximate IP range (of found entries from forward brute force) will be checked. The script will check `38.100.193.50-100`

```bash
kali@kali:~$ for ip in $(seq  50 100); do host 38.100.193.$ip; done | grep -v "not found"
69.193.100.38.in-addr.arpa domain name pointer beta.megacorpone.com.
70.193.100.38.in-addr.arpa domain name pointer ns1.megacorpone.com.
72.193.100.38.in-addr.arpa domain name pointer admin.megacorpone.com.
73.193.100.38.in-addr.arpa domain name pointer mail2.megacorpone.com.
76.193.100.38.in-addr.arpa domain name pointer www.megacorpone.com.
77.193.100.38.in-addr.arpa domain name pointer vpn.megacorpone.com.
```

* Use `grep -v "{STRING}"` command to include only results that **do not match** `{STRING}`
  * In this case it eliminates results for IPs with no hostname associated
