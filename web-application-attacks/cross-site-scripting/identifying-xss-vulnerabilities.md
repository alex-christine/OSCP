---
description: Basic techniques for locating XSS vulnerabilities in sites
---

# Identifying XSS Vulnerabilities

One an find potential entry points for XSS by examining a web application and identifying input fields (such as search fields) that accept unsanitized input which is displayed as output in subsequent pages.

## Testing Methods

### Manual

Manually testing for reflected and stored XSS normally involves submitting some simple unique input (such as a short alphanumeric string) into every entry point in the application, identifying every location where the submitted input is returned in HTTP responses, and testing each location individually to determine whether suitably crafted input can be used to execute arbitrary JavaScript. In this way, you can determine the context in which the XSS occurs and select a suitable payload to exploit it.

After an entry point is determined, special characters can be submitted to determine if any return unfiltered (and could potentially be used to break out of the page). The most common characters for this purpose are:

```
< > ' " { } ; /
```

These characters are useful because they are all structural elements of HTML/JavaScript and could potentially be used to inject script into the webpage.

### Automated

Tools such as BurpSuite's web vulnerability scanner, OpenVAS, Nessus, etc. can be used to assist in testing for XSS vulnerabilities.
