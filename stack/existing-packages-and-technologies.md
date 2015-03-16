---
layout: page
title: Existing Packages and Their Technologies
date: 2014-9-18
---

Universal Core includes a number of web applications, services and utility libraries. These provide functionality ranging from serving content to end users to managing surveys and authenticating users.

# List of packages

1. [unicore.ask](https://github.com/universalcore/unicore.ask) - service to store and manage polls and surveys
2. [unicore-cms](https://github.com/universalcore/unicore-cms) - end user application to serve and interact with UC's content (to be deprecated by [springboard](https://github.com/universalcore/springboard))
3. [unicore.content](https://github.com/universalcore/unicore.content) - utility package containing UC's primary content models
4. [unicore.distribute](https://github.com/universalcore/unicore.distribute) - service to expose primary content via a web API for distribution purposes
5. [unicore.google](https://github.com/universalcore/unicore.google) - utility package containing tools for interfacing with Google APIs
6. [unicore.hub](https://github.com/universalcore/unicore.hub) - service to authenticate users and store user profile data
7. [unicore-thumbor](https://github.com/universalcore/unicore-thumbor) - service to store and serve images

# Technologies used

The following technologies are being used in a number of applications and services. Services like `unicore.ask`, `unicore.distribute` and `unicore.hub` interact with other services via a RESTful API. Thus they can use any technology stack that suits their purpose. As more services are added, this list of technologies will become more diverse.

## Pyramid & Cornice

We are using the [Pyramid web framework](http://docs.pylonsproject.org/en/latest/docs/pyramid.html#pyramid-documentation) with [Cornice](https://cornice.readthedocs.org/en/latest/) to build certain web applications and services. Cornice provides us with the building blocks for RESTful APIs. These APIs allow for interaction between different services and user-facing applications.

## Jinja2

When we need to serve some HTML, we prefer using the [Jinja2](http://jinja.pocoo.org/docs/dev/) templating language.

## Babel & i18n

All Universal Core applications have to be internationalised. In order to do this, we use [Babel](http://babel.pocoo.org/) with Pyramid and Jinja2. In our Pyramid code we use `TranslationString` objects everywhere (more info [here](http://docs.pylonsproject.org/docs/pyramid/en/latest/narr/i18n.html)). These get translated in templates using Jinja2's `trans` tag and `_` function (more info [here](http://jinja.pocoo.org/docs/dev/extensions/#i18n-extension)).
