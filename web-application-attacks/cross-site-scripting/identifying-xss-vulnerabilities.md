---
description: Basic techniques for locating XSS vulnerabilities in sites
---

# Identifying XSS Vulnerabilities

One an find potential entry points for XSS by examining a web application and identifying input fields (such as search fields) that accept unsanitized input which is displayed as output in subsequent pages.

## Encoding Considerations

There are many different types of encoding that occur in web applications. The two most prevalent however are:

1. HTML Encoding
2. URL Encoding

Attackers may need to use different sets of characters, depending on where their input is being included.&#x20;

For example, if the input is being added between `div` tags, they will need to include their own `script` tags and need to be able to inject "`<`" and "`>`" as part of the payload. However, if the input is being added within an existing JavaScript tag, they might only need quotes and semicolons to add their own code.

### HTML Encoding

HTML encoding (or _character references_) can be used to display characters that normally have special meanings, like tag elements. For example `&lt;` is the character reference for `<` which is used to denote the start of elements in HTML.

Wikipedia has compiled a [complete listing](https://en.wikipedia.org/wiki/List\_of\_XML\_and\_HTML\_character\_entity\_references) of all HTML character references.

### URL Encoding

URL encoding converts characters into a format that can be transmitted over the Internet.

URLs can only be sent over the Internet using the ASCII character-set.

URL encoding replaces unsafe ASCII characters with a "%" followed by two hexadecimal digits.

E.g. since URLs cannot contain spaces. URL encoding normally replaces a space with a plus (`+`) sign or with `%20`.

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
