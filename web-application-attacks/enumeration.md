---
description: Techniques for web application enumeration
---

# Enumeration

## URL Inspection

File extensions, which are sometimes a part of a URL, can reveal the programming language the application was written in.&#x20;

Some of these, like `.php`, are straightforward, but other extensions are more cryptic and vary based on the frameworks in use. For example, a Java-based web application might use `.jsp`, `.do`, or `.html`.

However, file extensions on web pages are becoming less common since many languages and frameworks now support the concept of _routes_, which allow developers to map a URI to a section of code. Applications leveraging routes use logic to determine what content is returned to the user and make URI extensions largely irrelevant.

## Page Inspection

This like URL inspection is often a fairly laborious manual process (though some automated tools can help speed this process up a bit).

This can be done in-browser with the Inspector or Debugger tool (`Ctrl` + `Shift` + `K` for Firefox).

It is important to understand the key web frameworks for this type of inspection.The most popular frameworks are (in no particular order):

* [React](https://reactjs.org/)
* Angular
* Vue.JS
* Ember JS
* JQuery
* Ruby on Rails
* Django
* Laravel
* ASP.NET
* Express

## Response Headers

HTTP response headers can be viewed both in the web inspector and via a proxy such as BurpSuite. They can provide valuable information about the structure of the underlying application.

The `Server` header displayed above will often reveal at least the name of the web server software. In many default configurations, it also reveals the version number.

Historically, headers that started with "`X-`" were called non-standard HTTP headers. However, RFC6648 now deprecates the use of "`X-`" in favor of a clearer naming convention.

The names or values in the response header often reveal additional information about the technology stack used by the application. Some examples of non-standard headers include `X-Powered-By`, `x-amz-cf-id`, and `X-Aspnet-Version`. Further research into these names could reveal additional information, such as that the `x-amz-cf-id` header indicates the application uses Amazon CloudFront.

## Sitemaps

The most common sitemap locations are `robots.txt` and `sitemap.xml`.

For example one could retrieve the `robots.txt` file for `google.com` using the `curl` command.

```bash
kali@kali:~$ curl https://www.google.com/robots.txt
User-agent: *
Disallow: /search
Allow: /search/about
Allow: /search/static
Allow: /search/howsearchworks
Disallow: /sdch
Disallow: /groups
Disallow: /index.html?
Disallow: /?
Allow: /?hl=
...
```

`Allow` and `Disallow` are directives for web crawlers indicating pages or directories that "polite" web crawlers may or may not access, respectively. Although the listed pages and directories in most cases may not be interesting and some may even be invalid, sitemap files should not be overlooked as they may contain clues about the website layout or other interesting information.

## Locating Administration Consoles

Web servers often ship with remote administration web applications, or consoles, which are accessible via a particular URL and often listening on a specific TCP port.

Two common examples are the **manager** application for _Tomcat_ hosted at `/manager/html` and **phpMyAdmin** for _MySQL_ hosted at  and `/phpmyadmin` respectively.

While these consoles can be restricted to local access or may be hosted on custom TCP ports, one often finds them externally exposed by default configurations.

