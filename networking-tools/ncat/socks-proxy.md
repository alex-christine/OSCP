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

# SOCKS Proxy

Ncat can be used to proxy a connection. This is done with the following command structure:

```bash
ncat --proxy <host>:<port> --proxy-type <type> <dest-host> <dest-port> 
```

* `--proxy <host>:<port>` requests proxying of the connection through the host and port specified
* `--proxy-type <type>` specifies the type of proxy. `<type>` can be `http`, `socks4`, or `socks5` in connect mode
* Connection destination provided through `<dest-host>` and `<dest-port>`

## Examples

Connect to `smtphost` at port `25` via a SOCKSv5 proxy through `socks5host`:

{% code overflow="wrap" %}
```bash
ncat --proxy socks5host --proxy-type socks5 smtphost 25
```
{% endcode %}

* If no port is provided with `socks5host` the well-known SOCKSv5 port (`1080`) will be assumed

Connect to `smtphost` at port `25` via a SOCKSv4 proxy at `socks4host` port `9999`. Authorize as user `joe`:

```
ncat --proxy socks4host:9999 --proxy-type socks4 --proxy-auth joe smtphost 25
```

Connect to `smtphost` at port `25` via a SOCKSv5 proxy at `socks5host`. Authorize as user `joe` and password `secret`:

```
ncat --proxy socks5host --proxy-type socks5 --proxy-auth joe:secret smtphost 25
```

* Password should only be provided with `--proxy-auth` flag if using SOCKSv5, not SOCKSv4
