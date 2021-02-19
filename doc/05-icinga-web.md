# Icinga Web Development Guide

## Introduction

We should start off with a little explanation on how Icinga web development looks like and which layers are necessary in order to build a web application:

* Explain frontend development (client-side)
* Explain backend development (server-side)
* Explain storage features (files, database)

## Programming, Markup and Database Languages

The guide should explain our used languages, i.e.

**Sever-side**

* PHP 5.6+, preferably PHP7

**Client-side**

* JavaScript: ES5 (ES6 once we updated our code base)

**Markup**

* HTML5
* CSS3

**Storage**

* INI and JSON files
* MySQL
* PostgreSQL

## Browser, OS and Web Server Support

The guide should state which browsers, operating systems and web servers the Icinga web ecosystem supports.

## PHP and JavaScript Style Guide

Of course, it is never bad to mention the style guides somewhere.

## Basics and Technologies

Before deep diving into writing code, we should explain necessary basics and technologies:

* HTTP protocol
    * This may include an explanation for our special HTTP headers and their handling, but there may also be an extra chapter for this
* Cookies
    * This may include information about our cookies and their handling, but there may also be an extra chapter for this
* Ajax
* MVC
* JavaScript and CSS delivery
* X-column layout
* URL handling
* Form redirects
* Flow of requests, i.e. first request is a full-page load including the delivery of JavaScript and CSS and every subsequent request is AJAX
* Convention over configuration
* Translation
* Docker

## Git

This chapter should cover branch naming conventions and workflows, commit message anatomy and provide a cheat sheet for common Git magic that we use every day, e.g. squash, rebase, interactive rebase, git add -p.

## SQL

We follow some conventions when designing databases and learned best practices during our developments which are definitely worth sharing.

## Development Environment

We should have a dedicated chapter for setting up a dev environment also integrating XDebug. I do think that this will be a docker setup.

## Development

This is an incomplete list of topics which the development chapter should cover, and it also only lists the main topics for the moment:

* Conventions, e.g. naming, directory structure, meta information
* Controllers
* Auto-refresh
* Views
* Custom CSS
* Forms
* Configuration
* Menu
* Dashboards
* Database access
* Permissions and restrictions
* Hooks
* Translation
* Custom JS
* CLI


We should try to avoid explaining topics in too much detail which are already well documented externally. This applies to the following:

* Used languages
    * That's e.g. PHP, besides mentioning its name (with a link php.org), its ecosystem (composer, packagist, etc), wanted style guide and a short description of how it's used should be enough.
    * I really don't want to explain syntax elements here.
* Basics and Technologies
    * HTTP, Cookies, XHR, MVC, etc. Yes we use them, yes we have customizations which need mentioning, but there are hopefully still more external links than actual documentation.

Things I'd like to see added:

* Custom JS
    * events and best practices handling them
    * what are behaviors, how do they work
* Custom CSS
    * Sandboxing and how/when to break out
    * Themes, what they should do, what not
    * Responsiveness, em/vh/etc VS px, generally how we utilize font-size here
* Unittests
    * How to write one
    * How to run it
    