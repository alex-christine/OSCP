---
description: Various utilities to assist in DNS reconnaissance
---

# Tools

## DNSRecon

DNSRecon is an advanced, modern DNS enumeration script written in Python.

### Zone Transfer

DNSRecon can be used to automate zone transfer attempts. Run `dnsrecon` with `-d {domain}` flag to specify domain and `-t axfr` to specify _type of attack_ as zone transfer.

#### Example

```
kali@kali:~$ dnsrecon -d megacorpone.com -t axfr
[*] Testing NS Servers for Zone Transfer
[*] Checking for Zone Transfer for megacorpone.com name servers
[*] Resolving SOA Record
[+] 	 SOA ns1.megacorpone.com 38.100.193.70
[*] Resolving NS Records
[*] NS Servers found:
[*] 	NS ns1.megacorpone.com 38.100.193.70
[*] 	NS ns2.megacorpone.com 38.100.193.80
[*] 	NS ns3.megacorpone.com 38.100.193.90
[*] Removing any duplicate NS server IP Addresses...
[*]
[*] Trying NS server 38.100.193.80
[+] 38.100.193.80 Has port 53 TCP Open
[+] Zone Transfer was successful!!
[*] 	 NS ns1.megacorpone.com 38.100.193.70
[*] 	 NS ns2.megacorpone.com 38.100.193.80
[*] 	 NS ns3.megacorpone.com 38.100.193.90
[*] 	 MX @.megacorpone.com fb.mail.gandi.net 217.70.178.215
[*] 	 MX @.megacorpone.com fb.mail.gandi.net 217.70.178.217
[*] 	 MX @.megacorpone.com fb.mail.gandi.net 217.70.178.216
[*] 	 MX @.megacorpone.com spool.mail.gandi.net 217.70.178.1
[*] 	 A admin.megacorpone.com 38.100.193.83
[*] 	 A fs1.megacorpone.com 38.100.193.82
[*] 	 A www2.megacorpone.com 38.100.193.79
[*] 	 A test.megacorpone.com 38.100.193.67
[*] 	 A ns1.megacorpone.com 38.100.193.70
[*] 	 A ns2.megacorpone.com 38.100.193.80
[*] 	 A ns3.megacorpone.com 38.100.193.90
...
[*]
[*] Trying NS server 38.100.193.70
[+] 38.100.193.70 Has port 53 TCP Open
[-] Zone Transfer Failed!
[-] No answer or RRset not for qname
[*]
[*] Trying NS server 38.100.193.90
[+] 38.100.193.90 Has port 53 TCP Open
[-] Zone Transfer Failed!
[-] No answer or RRset not for qname
```

In this example, a zone transfer was performed successfully.

### Brute Force

DNSRecon is also capable of automating brute force discovery.

Run `dnsrecon` with the following:

| Flag          | Description                                                |
| ------------- | ---------------------------------------------------------- |
| `-d {domain}` | _Domain_ for brute forcing                                 |
| `-D {file}`   | Specifies a file containing a _wordlist_ for brute forcing |
| `-t brt`      | Specifies _type_ as brute force                            |

#### Example

```
kali@kali:~$ dnsrecon -d megacorpone.com -D ~/list.txt -t brt
[*] Performing host and subdomain brute force against megacorpone.com
[*] 	 A router.megacorpone.com 38.100.193.71
[*] 	 A www.megacorpone.com 38.100.193.76
[*] 	 A mail.megacorpone.com 38.100.193.84
[+] 3 Records Found
```

## DNSenum

DNSenum is another popular tool for performing DNS enumeration.

#### Example

In this example `dnsenum` is run against `zonetransfer.me` (a domain setup specifically to allow zone transfer for learning purposes)

```
kali@kali:~$ dnsenum zonetransfer.me
dnsenum.pl VERSION:1.2.2
-----   zonetransfer.me   -----

Host's addresses:
__________________

zonetransfer.me                          7200     IN    A        217.147.180.162


Name Servers:
______________

ns12.zoneedit.com                        3653     IN    A        209.62.64.46
ns16.zoneedit.com                        6975     IN    A        69.64.68.41


Mail (MX) Servers:
___________________

ASPMX5.GOOGLEMAIL.COM                    293      IN    A        173.194.69.26
ASPMX.L.GOOGLE.COM                       293      IN    A        173.194.74.26
ALT1.ASPMX.L.GOOGLE.COM                  293      IN    A        173.194.66.26
ALT2.ASPMX.L.GOOGLE.COM                  293      IN    A        173.194.65.26
ASPMX2.GOOGLEMAIL.COM                    293      IN    A        173.194.78.26
ASPMX3.GOOGLEMAIL.COM                    293      IN    A        173.194.65.26
ASPMX4.GOOGLEMAIL.COM                    293      IN    A        173.194.70.26


Trying Zone Transfers and getting Bind Versions:
_________________________________________________


Trying Zone Transfer for zonetransfer.me on ns12.zoneedit.com ...
zonetransfer.me                          7200     IN    SOA
zonetransfer.me                          7200     IN    NS
...
office.zonetransfer.me                   7200     IN    A        4.23.39.254
owa.zonetransfer.me                      7200     IN    A        207.46.197.32
info.zonetransfer.me                     7200     IN    TXT
asfdbbox.zonetransfer.me                 7200     IN    A         127.0.0.1
canberra_office.zonetransfer.me          7200     IN    A        202.14.81.230
asfdbvolume.zonetransfer.me              7800     IN    AFSDB
email.zonetransfer.me                    2222     IN    NAPTR
dzc.zonetransfer.me                      7200     IN    TXT
robinwood.zonetransfer.me                302      IN    TXT
vpn.zonetransfer.me                      4000     IN    A        174.36.59.154
_sip._tcp.zonetransfer.me                14000    IN    SRV
dc_office.zonetransfer.me                7200     IN    A        143.228.181.132

ns16.zoneedit.com Bind Version: 8.4.X

brute force file not specified, bay.
```
