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

# Attacking Network Services Logins

In the last decade, **brute-force** and dictionary attacks against publicly-exposed network services have increased dramatically. The common **Secure Shell** (**SSH**), **Remote Desktop Protocol** (**RDP**), and **Virtual Network Computing** (**VNC**) services as well as **web-based login forms** are often attacked seconds after they are launched.

Brute-force attacks attempt every possible password variation, working systematically through every combination of letters, digits and special characters. Although this may take a considerable amount of time depending on the length of the password and the protocol in use, these attacks could theoretically bypass any ill-protected password-based authentication system.

On the other hand, dictionary attacks attempt to authenticate to services with passwords from lists of common words (**wordlists**). If the correct password is not contained in the wordlist, the dictionary attack will fail.
