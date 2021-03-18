# Icinga Web Development Guide

## Introduction

We should start off with a little explanation on how Icinga web development looks like and which layers are necessary in order to build a web application:

* Explain frontend development (client-side)
* Explain backend development (server-side)
* Explain storage features (files, database)

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
    