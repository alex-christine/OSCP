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

