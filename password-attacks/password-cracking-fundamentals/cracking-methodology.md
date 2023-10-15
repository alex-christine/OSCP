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

# Cracking Methodology

The rough steps of cracking can be described in the following steps:

1. Extracts hashes
2. Format hashes
3. Calculate cracking time
4. Prepare wordlist
5. Attack the hash

In a penetration test one may find hashes in various locations. E.g. if they get access to a database system, the could dump the database table containing the hashed user passwords.

Any extracted hashes must then be formatted as expected by the cracking tool of the attacker's choice. This includes **identifying the type of hash used** on the passwords. This can be accomplished with `hash-identifier` or `hashid` both of which are installed on Kali. Depending on the hashing algorithm and the source of the hash, it may need to be modified to match the expected format for the hashing tool.

It is often worth double checking the determined hash type as some hashes may be indeterminate and tools can also just be wrong sometimes. For example, `hashid` can't automatically determine if `b08ff247dc7c5658ff64c53e8b0db462` is MD2, MD4, or MD5. An incorrect choice will obviously waste time. This situation can potentially be avoided by double-checking the results with other tools and doing some extra research.

The cracking time is then calculated to determine the feasibility of this approach. If the cracking time (on the hardware available) is longer than a human lifetime this may be a less-than-helpful approach. Cracking time is calculated, in the worst case scenario, as the key space divided by the hash rate.

Standard wordlists will almost never contain the password matching the hash. For starters, most workplaces have password policies that the wordlist passwords do not match. Because of this, it will almost always be necessary to modify the wordlist with some rules. If possible, it can also be hugely beneficial to add some entries to the wordlist that are specific to the target. E.g. the names of their pets, spouse, children; the schools they attended; words related to their hobbies.

At this point it is time to start the cracking tool and wait. Take care when copy/pasting the hash as an extra space or character will render all this effort worthless.
