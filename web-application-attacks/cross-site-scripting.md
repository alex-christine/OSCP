---
description: Basic information to conduct a cross site scripting attack
---

# Cross Site Scripting

One of the most important features of a well-defended web application is _data sanitization_, a process in which user input is processed, removing or transforming all dangerous characters or strings. Unsanitized data allows an attacker to inject and potentially execute malicious code. When this unsanitized input is displayed on a web page, this creates a **Cross-Site Scripting (XSS)** vulnerability.

There are three Cross-Site Scripting variants:

1. Stored
2. Reflected
3. DOM-based

## Stored XSS

### Background

**Stored XSS attacks** (also known as **Persistent XSS**) occurs when the exploit payload is stored in a database or otherwise cached by a server. The web application then retrieves this payload and displays it to anyone that views a vulnerable page. Arises when an application receives data from an untrusted source and includes that data within its later HTTP responses in an unsafe way.

A single Stored XSS vulnerability can therefore attack all users of the site. Stored XSS vulnerabilities often exist in:

* Forum software
  * Comment sections
  * Product reviews

### Exploiting Stored XSS

Simple example of a stored XSS vulnerability. A message board application lets users submit messages, which are displayed to other users:

```
<p>Hello, this is my message!</p>
```

The application doesn't perform any other processing of the data, so an attacker can easily send a message that attacks other users:

```
<p><script>/* Bad stuff here... */</script></p>
```

If this were successfully saved to the server as a forum message, any time other users viewed that message in their own browsers, the malicious JavaScript would execute.

## Reflected XSS

### Background

**Reflected XSS attacks** usually include the payload in a crafted request or link. The web application takes this value and places it into the page content. This variant only attacks the person submitting the request or viewing the link.

Arises when an application receives data in an HTTP request and includes that data within the immediate response in an unsafe way.

Reflected XSS vulnerabilities can often occur in

* Search fields and results
* Anywhere user input is included in error messages

### Exploiting Reflected XSS

Simple HTML example of a page vulnerable to reflected XSS:

```
https://insecure-website.com/status?message=All+is+well.
<p>Status: All is well.</p>
```

The application doesn't perform any other processing of the data, so an attacker can easily construct an attack like this:

```
https://insecure-website.com/status?message=<script>/*+Bad+stuff+here...+*/</script>
<p>Status: <script>/* Bad stuff here... */</script></p>
```

If the user visits the URL constructed by the attacker, then the attacker's script executes in the user's browser, in the context of that user's session with the application. At that point, the script can carry out any action, and retrieve any data, to which the user has access.

## DOM-Based XSS

**DOM-based XSS** _attacks_ are similar to the other two types, but take place solely within the page's Document Object Model (DOM). Details about the attack type can be found [here](https://portswigger.net/web-security/cross-site-scripting/dom-based).

### Background

XSS vulnerabilities were originally found in applications that performed all data processing on the server side. User input (including an XSS vector) would be sent to the server, and then sent back to the user as a web page. The need for an improved user experience resulted in popularity of applications that had a majority of the presentation logic (maybe written in JavaScript) working on the client-side that pulled data, on-demand, from the server using AJAX.

Under this model, a browser parses a page's HTML content and generates an internal DOM representation. JavaScript can programmatically interact with this DOM.

This variant occurs when a page's DOM is modified with user-controlled values. DOM-based XSS can be stored or reflected. The key difference is that DOM-based XSS attacks _occur when a browser parses the page's content and inserted JavaScript is executed_.

These types of problems usually arise when JavaScript takes data from an attacker-controllable source, such as the URL, and passes it to a sink that supports dynamic code execution, such as `eval()` or `innerHTML`.

### Testing for DOM-Based XSS

Because these attacks rely on user input being passed to a sink that supports dynamic code execution the tests largely rely on finding HTML and JavaScript-based sinks. For a list of sinks that can lead to DOM-Based XSS see [here](https://portswigger.net/web-security/cross-site-scripting/dom-based#which-sinks-can-lead-to-dom-xss-vulnerabilities).

#### Testing HTML Sinks

To test for DOM XSS in an HTML sink, place a random alphanumeric string into the source (such as `location.search`), then use developer tools to inspect the HTML and find where your string appears.

For each location where your string appears within the DOM, you need to identify the context. Based on this context, you need to refine your input to see how it is processed. For example, if your string appears within a double-quoted attribute then try to inject double quotes in your string to see if you can break out of the attribute.

Note that browsers behave differently with regards to URL-encoding, Chrome, Firefox, and Safari will URL-encode `location.search` and `location.hash`, while IE11 and Microsoft Edge (pre-Chromium) will not URL-encode these sources. _If your data gets URL-encoded before being processed, then an XSS attack is unlikely to work_.

#### Testing JavaScript Execution Sinks

Testing JavaScript execution sinks for DOM-based XSS is a little harder. With these sinks, your input doesn't necessarily appear anywhere within the DOM, so you can't search for it. Instead you'll need to use the JavaScript debugger to determine whether and how your input is sent to a sink.

For each potential source, such as `location`, you first need to find cases within the page's JavaScript code where the source is being referenced.

* In Chrome and Firefox's developer tools, you can use `Control+Shift+F` (or `Command+Alt+F` on MacOS) to search all the page's JavaScript code for the source

### Exploiting DOM-Based XSS

In principle, a website is vulnerable to DOM-based cross-site scripting if there is an executable path via which data can propagate from source to sink. In practice, different sources and sinks have differing properties and behavior that can affect exploitability, and determine what techniques are necessary. Additionally, the website's scripts might perform validation or other processing of data that must be accommodated when attempting to exploit a vulnerability.

#### Example

The `document.write` sink works with `script` elements, so you can use a simple payload, such as the one below:

```javascript
document.write('... <script>alert(document.domain)</script> ...');
```
