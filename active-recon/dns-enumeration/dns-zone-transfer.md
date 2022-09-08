---
description: Description of, and techniques for performing, DNS zone transfer
---

# DNS Zone Transfer

A zone transfer is basically a database replication between related DNS servers in which the _zone file_ is copied from a master DNS server to a slave server.&#x20;

* Zone file contains a **list of all the DNS names configured for that zone**
* Zone transfers **should only be allowed to authorized slave DNS servers**
  * Many administrators misconfigure their DNS servers
    * In these cases, anyone asking for a copy of the DNS server zone will usually receive one

## Operation

Zone transfer consists of a preamble, followed by the actual data transfer.&#x20;

The preamble is comprised of:

* Lookup of the **Start of Authority (SOA) resource record** for the **zone apex**
  * Apex is the node of the DNS namespace that is at the top of the _zone_
* **Fields** of this SOA resource record
  * Most important is the **serial number**
    * Used to determine whether the actual data transfer need to occur at all

#### Steps

1. Client initiates transfer and receives preamble from server
2. The client compares the serial number of the SOA resource record with the serial number in the last copy of that resource record that it has
   * If the **serial numbers are identical**, the data in the zone are deemed _not to have changed_, and the client may continue to use the copy of the database that it already has, if it has one.
3. If the serial number of the record being transferred is greater, the data in the zone are deemed to have "changed" (in some fashion) and the secondary proceeds to request the actual zone data transfer
4. The actual data transfer process begins by the client sending a query (opcode 0) with the special query type `AXFR` (value 252) over the TCP connection to the server.&#x20;
   * Although DNS technically supports `AXFR` over User Datagram Protocol (UDP), it is considered not acceptable due to the risk of lost, or spoofed packets
5. Server responds with a series of response messages, comprising all of the resource records for every domain name in the _zone_.&#x20;
   * First response comprises the SOA resource record for the zone apex
   * Other data follows in no specified order.&#x20;
6. The end of the data is signaled by the server repeating the response containing the SOA resource record for the zone apex

### Serial Numbers

The preamble portion of zone transfer relies on the serial number, and _only_ the serial number, to determine whether a zone's data have changed, and thus whether the actual data transfer is required.

For some DNS server packages, the serial numbers of SOA resource records are maintained by administrators by hand. Every edit to the database involves making two changes, one to the record being changed and the other to the zone serial number. The process requires accuracy.

Some DNS server packages have overcome this problem by automatically constructing the serial number from the last modification timestamp of the database file on disk. The operating system ensures that the last modification timestamp is updated whenever an administrator edits the database file, effectively automatically updating the serial number, and thus relieving administrators of the need to make two edits for every single change.

Furthermore, **the paradigm of database replication for which the serial number check is designed**, which involves a single central DNS server holding the primary version of the database with all other DNS servers merely holding copies, **simply does not match that of many modern DNS server packages**.

* Modern DNS server packages with sophisticated database back ends (such as _SQL_ servers and _Active Directory_) allow administrators to make updates to the database in multiple places (such systems employ _multi-master replication_)
  * Database back end's own replication mechanism handling the replication to all other servers (as opposed to standard zone transfers)
  * This paradigm does not match that of a single, central, monotonically increasing number to record changes
    * Is incompatible with zone transfer to a large extent
  * DNS servers that use sophisticated database back ends rarely use zone transfer as their database replication mechanism in the first place

## Performing Zone Transfers

A successful zone transfer does not directly result in a network breach, although it does facilitate the process.

Syntax for performing a zone transfer with `host`:

```bash
host -l {domain_name} {dns_server_address}
```

#### Example

Continuing the example used in the [Automating Lookups](automating-lookups.md) section. In the first scan there were 3 DNS servers listed (`ns1`, `ns2`, `ns3`).

Utilize a `host -l` command to attempt the zone transfer:

```bash
kali@kali:~$ host -l megacorpone.com ns1.megacorpone.com
Using domain server:
Name: ns1.megacorpone.com
Address: 38.100.193.70#53
Aliases: 

Host megacorpone.com not found: 5(REFUSED)
; Transfer failed.
```

It would appear `ns1` is properly configured and did not allow the zone transfer. Retry with `ns2` (and `ns3` if needed):

```bash
kali@kali:~$ host -l megacorpone.com ns2.megacorpone.com
Using domain server:
Name: ns2.megacorpone.com
Address: 38.100.193.80#53
Aliases:

megacorpone.com name server ns1.megacorpone.com.
megacorpone.com name server ns2.megacorpone.com.
megacorpone.com name server ns3.megacorpone.com.
admin.megacorpone.com has address 38.100.193.83
beta.megacorpone.com has address 38.100.193.88
fs1.megacorpone.com has address 38.100.193.82
intranet.megacorpone.com has address 38.100.193.87
mail.megacorpone.com has address 38.100.193.84
mail2.megacorpone.com has address 38.100.193.73
ns1.megacorpone.com has address 38.100.193.70
...
```

This server allows zone transfers and provides a full dump of the zone file for the `megacorpone.com` domain, delivering a convenient list of IP addresses and corresponding DNS hostnames.

### Automating Zone Transfers

It is possible to write a Bash script to check a domain for all DNS servers (`NS` records), and then attempt a zone transfer against each found server.

{% code title="dns-axfr.sh" %}
```bash
#!/bin/bash

# Simple Zone Transfer Bash Script
# $1 is the first argument given after the bash script
# Check if argument was given, if not, print usage

if [ -z "$1" ]; then
  echo "[*] Simple Zone transfer script"
  echo "[*] Usage   : $0 <domain name> "
  exit 0
fi

# if argument was given, identify the DNS servers for the domain

for server in $(host -t ns $1 | cut -d " " -f4); do
  # For each of these servers, attempt a zone transfer
  host -l $1 $server |grep "has address"
done

```
{% endcode %}

```
kali@kali:~$ ./dns-axfr.sh megacorpone.com
admin.megacorpone.com has address 38.100.193.83
beta.megacorpone.com has address 38.100.193.88
fs1.megacorpone.com has address 38.100.193.82
intranet.megacorpone.com has address 38.100.193.87
mail.megacorpone.com has address 38.100.193.84
mail2.megacorpone.com has address 38.100.193.73
ns1.megacorpone.com has address 38.100.193.70
ns2.megacorpone.com has address 38.100.193.80
ns3.megacorpone.com has address 38.100.193.90
router.megacorpone.com has address 38.100.193.71
...
```

The example `dns-asfr.sh` script is quite simplistic, but could be extended if desired.
