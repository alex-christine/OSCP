---
description: Details about commonly-used web development frameworks
---

# Web Frameworks

## Front-End Frameworks

### AngularJS

Google introduced [AngularJS](https://angularjs.org/), an open-source JavaScript framework. With the release of the rewrite version “Angular 2.0”, which enables the development of high-performance and large-scale JavaScript based web applications.

With its robust set of features and ability to work with cross-platform and client-side frameworks, Angular applications run smoothly on both the web and mobile platforms. Angular promotes code consistency by using HTML, CSS, and TypeScript (a superset of JavaScript), in addition to web development tools.

#### Key Features

* Angular reduces development time by using boilerplate coding (code sections that appear repeatedly with little or without any changes)
* It reduces the build time by allowing developers to reuse components and even the architecture to simplify the development process. Additionally, it simplifies the testing process
* Encourages reusability and improves application scalability
* Using open source libraries such as Angular Material and AgGrid, it’s possible to create a responsive and dynamic user interface with many features
* Angular CLI (Command Line Interface) is regarded as one of the best command line interfaces for building, scaffolding, and maintaining web applications
* To debug web applications, Angular supports or provides  Chrome and Firefox Dev tools and extensions. The framework is also home to a wide range of third-party libraries

### Ember JS

[Ember JS](https://emberjs.com/) launched in late 2011 and is considered one of the most productive open-source JavaScript frameworks utilizing the [MVVM](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93viewmodel) paradigm. It contains HTML (Hypertext Markup Language) and CSS (Cascading Style Sheets) at the core of the development model.

It is well-known for its ability to create maintainable and reusable JavaScript web applications. Ember is designed to maximize developer productivity either by eliminating time-wasting features or implementing JS best practices in the core design.

One of the top frameworks used to build single-page applications and dynamic client-side applications that can be extended by using general idioms and advanced practices.

#### Key Features

* Similar to Angular, it provides two-way data binding. It aims to satisfy the growing demand for contemporary technologies flawlessly
* Because it is backwards compatible, old versions of applications will still function flawlessly even with new updates
* All tools associated with it are packaged well
* Takes parts from Angular and React and optimizes them
* A complete front end stack is provided by Ember JS, including a router, services, and asset pipeline. The router is a core feature of Ember.js, which is used to manage URLs
* In Ember.js, there is a tool called Ember Inspector, which helps debug Ember applications
* Using templates, embed.js automatically updates the model whenever the content of the application changes

### JQuery

[JQuery](https://jquery.com/) was released in 2006 as an open-source, lightweight JavaScript library to help developers build robust web applications.

JQuery is a small, fast JavaScript library that simplifies the interaction between HTML elements and JavaScript code. It simplifies CSS animations, event handling, and Ajax calls, which makes web pages more interactive.

#### Key Features

* It is easy to select DOM elements, traverse them, and modify their content with jQuery. The methods available in JQuery such as `.attr()`, `.html()`, etc., simplify DOM manipulation
* With jQuery, you can perform a series of actions on your website with a single click, such as starting an animation, sending data to servers, adding a class, etc. Thus, it shortens the process of binding and unbinding event handlers
* Plugins are an essential part of JQuery, allowing developers to extend the functionality of a web application. You can find a large number of useful plugins on the web to enhance jQuery with functions such as Ajax helpers, data grids, XML and XSTL tools, dynamic lists, etc. These make developing web apps easy and quick
* It combines markup technologies such as CSS, HTML, JavaScript, and AJAX to produce animations like Flash. Additionally, it comes with several built-in animation effects that help developers build feature-rich and responsive websites with AJAX
* It is possible to achieve seamless integration between JavaScript and JQuery by using JQuery’s utility functions

### React

Introduced by Meta in 2013, [React](https://reactjs.org/) is an open-source JavaScript library that can be used to build interactive user interfaces that would entice any developer or business to use it for their front-end development. Useful for building dynamic web applications or single-page applications, and can even be used for mobile applications.

#### Virtual Document Object Model

In basic HTML, the DOM (Document Object Model is a structured representation of the HTML elements that are present in a webpage or web-app. DOM represents the entire UI of an application. The DOM is represented as a tree data structure. It contains a node for each UI element present in the web document. It is very useful as it allows web developers to modify content through JavaScript, also it being in structured format helps a lot as we can choose specific targets and all the code becomes much easier to work with.

React uses **Virtual DOM (VDOM)** exists which is _like a lightweight copy of the actual DOM_ (a virtual representation of the DOM). So for every object that exists in the original DOM, there is an object for that in React Virtual DOM. It is exactly the same, but it does not have the power to directly change the layout of the document. Manipulating DOM is slow, but manipulating Virtual DOM is fast as nothing gets drawn on the screen. So each time there is a change in the state of our application, the virtual DOM gets updated first instead of the real DOM.

When anything new is added to the application:

1. A virtual DOM is created and it is represented as a tree. Each element in the application is a node in this tree.&#x20;
2. Whenever there is a change in the state of any element, a new Virtual DOM tree is created
   * React maintains two Virtual DOM at each time, one contains the updated Virtual DOM and one which is just the pre-update version of this updated Virtual DOM
3. This new Virtual DOM tree is then compared with the previous Virtual DOM tree and make a note of the changes
   * Process of comparing the current Virtual DOM tree with the previous one is known as **diffing**
4. React then finds the best possible ways to make these changes to the real DOM
   * Once React finds out what exactly has changed then it updated those objects only, on real DOM
   * React uses batch updates to update the real DOM
     * This means that the changes to the real DOM are sent in batches instead of sending any update for a single change in the state of a component
     * Re-rendering of the UI is the most expensive part and React manages to do this most efficiently by ensuring that the Real DOM receives batch updates to re-render the UI
   * This entire process of transforming changes to the real DOM is called **reconciliation**

While this may seem like unnecessary and duplicative work in reality it is time saving. Updating the VDOM takes little to no time in comparison with updating the actual DOM

#### Key Features

* Virtual Document Object Model (DOM)
  * With its virtual DOM, even heavy-load applications will perform smoothly and render quickly
* Bundles front-end code into components
* Organizes code and data to make code more reusable
* Has captured a significant share of the mobile market with React Native, a cross-platform mobile development framework
* In React applications, the information flow is unidirectional
  * One-way data bindings make React less prone to errors and easier to debug, making it an efficient framework
* Flexibility of React allows developers to create applications that dynamically adapt to any user interface
  * Software engineers have used React to build applications for all kinds of user interfaces, including web, mobile, desktop, smart TVs, etc.
* SEO-friendly

### Vue JS

Created by Google in 2014, [Vue](https://vuejs.org/) is an open-source JavaScript framework capable of creating stunning and interactive user interfaces.

Designed to be the viable alternative to React and Angular for developing SPAs (Single Page Applications), high performance progressive web apps, and visually appealing user interfaces. It is a progressive JavaScript framework that combines the best features of React (Virtual DOM) and Angular (View Layer).

#### Key Features

* Virtual DOM improves the performance of the application and the efficiency of DOM updates
  * Vue uses it to determine what parts of the DOM need to be re-rendered and which ones should be left intact
* By making scaffolding and prototyping easier, Vice CLI serves as an easy-to-use framework for web development
  * Reduces the amount of time used to develop the project
* There is reactive two-way data binding in Vue
  * Any change made to the UI (user interface) will affect the data and vice versa
* Can integrate this framework as a library or module into an existing application or build the entire application using it
  * Enhances the flexibility of the web application development process
* Lightweight, which means it can be easily downloaded and installed.
* &#x20;Applications are very fast to launch, resulting in a seamless user experience

## Back-End Frameworks

### ASP.NET

[ASP.NET](https://dotnet.microsoft.com/en-us/apps/aspnet) is a popular open-source web development framework that can be used to create dynamic web applications for PCs and mobile devices.

It was created by Microsoft to keep up with the latest trends in web development, so that programmers could create dynamic websites, applications, and services. In 2016, ASP.NET Core was introduced; this new version of ASP.NET is enticing developers and businesses around the world with its scalability, flexibility, and high performance features. Furthermore, it is compatible with JavaScript-based front-end frameworks.

#### Key Features

* A .NET Core application works seamlessly with any client-side framework, and it can be deployed throughout a range of platforms, including MacOS, Windows, and Linux
* Asynchronous development is supported in ASP.NET Core (async/await), making this framework fast and improving app performance
* ASP.NET Core combines Web API and MVC to simplify the process of creating APIs and make client-side implementation more feasible
* One more important aspect of ASP.NET Core that makes it less complex is Razor Pages
  * This feature allows developers to build server-side rendered apps more quickly and efficiently
* With JetBrains Rider, .NET Code Profiler, Visual Studio Code, and other excellent developer tools, it provides a seamless and fast application development experience

### Django

[Django](https://www.djangoproject.com/) is a popular, open source Python-based back-end web development framework that is gaining popularity among developers and enterprises by making it easier to develop complex, highly scalable, and data-driven web applications.

The framework was designed by experienced engineers, making it ideal for building API features like naive GraphQL integration. One can use this framework in any format, such as HTML, JSON, and XML.

#### Key Features

* By default, Django preempts various security risks and issues such as cross-site request forgery (CSRF), cross-site scripting (XSS), and SQL injections
* Due to its code reusability and caching options, Django is highly scalable, making the application capable of handling any traffic demand
* It is built on top of Python and uses Model-View-Template architecture, which allows for robust handling of asynchronous and reactive programming
* With Django, you can easily build any type of website, from news sites and social networks to content management systems (CMS)
* Django applications are SEO-friendly and easy to optimize because they are maintained through URLs rather than IP addresses

### Express

[Express](http://expressjs.com/) is an open-source, lightweight back-end framework for Node.js (JavaScript runtime environment), and is designed to build web applications, mobile applications, and APIs.

It provides the core features of a web application to an already feature-rich Node.js platform, making it a flexible framework. Express is one of the best back-end frameworks with features such as debugging, routing, and fast back-end programming. It allows rapid development of Web applications based on Node.js.

#### Key Features

* Through its caching potential, it can dramatically reduce a site’s loading time by eliminating the need to execute code repeatedly
* Express boasts an advanced routing system that allows preserving a webpage’s state using URLs
* Express makes debugging more convenient by offering a debugging technique to figure out the precise parts of a web app containing bugs
* Express comes with several template engines such as EJS, Jade, Pug, etc., to help you design HTML pages easily
* It provides a middleware system and several HTTP methods that can be used to build Node.JS apps and APIs quickly
* The framework can be used for creating multi-page, single-page, and hybrid web applications

### Ruby On Rails

[Ruby on Rails](https://rubyonrails.org/) or Rails is a most popular open-source back-end web development framework based on [MVC](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller) (Model-view-controller) that emphasizes using some worthwhile and well-known software engineering patterns and paradigms like CoC (Convention Over Configuration), DRY (Don’t repeat yourself), and the active record pattern.

#### Key Features

* It facilitates and promotes using web standards (such as  XML, JSON) for data transfer, as well as JavaScript, HTML, and CSS and for user interaction
* Ruby on Rails has a powerful and robust library called the active record, which simplifies designing database queries
* RSpec is a unit test setup included with Ruby on Rails that is easy to learn and use.&#x20;
  * Can be used to test the functions present in the application by calling them separately. In this way, you can ensure you have tested your application thoroughly
* There are numerous libraries included in Ruby on Rails that simplify the coding of common programming tasks like form validation, session management, and so on
* Equips developers with all essential tools to build a high-quality database access library, product AJAX library, and common tasks library
* Ruby on Rail syntax is simple, concise, more like the English language, and even flexible.&#x20;
  * Ruby on Rails is an object-oriented framework
