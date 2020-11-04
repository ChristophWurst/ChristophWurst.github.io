---
layout: single
title: "Rock Me Amadeus: Escaping the php Dependency Hell"
comments: true
date: 2020-05-13
last_modified_at: 2020-05-13
tags:
  - nextcloud
  - development
  - php
  - foss
---

Similar to some other programming language and environments, php is subject a form of the [dependency hell](https://en.wikipedia.org/wiki/Dependency_hell). That means only one version of a class can be loaded at a time. For simple applications that is less of a problem as you will most likely composer to manage your dependencies. Composer flattens your dependency hierarchy so all required packages and their sub-packages can live next to each other while all version constraints are satisfied. If one library is required with two or more disjoint version ranges, composer will show an error and refuse the installation.

## Nextcloud and the Dependency Hell

With Nextcloud apps it's even a bit more complicated. We use composer for our libraries, but the pluggable architecture allows apps to ship additional code. This code is not only the application code, but also often additional packages via the app's composer. As the Nextcloud composer and any app's composer are not aware of each other, both will register a class autoloader therefore there is the chance that two or more app load a dependency multiple times. In the lucky case, nobody notices as both apps use the same (latest) version of the dependency. In the less lucky case, there will be strange bugs as parts of the one version of the package and parts of the other are loaded. When admins who detect the problem report it, it can typically not be reproduced because the developer's instance might not have both conflicting applications installed.

## A Promising Workaround

Again, a typical php application won't ever run into this problem. But it's not only Nextcloud which is affected by this problem – Wordpress has the exact same issue with its plugins. Mozart is a small package that promises to help. It rewrites source file from composer's vendor directory and prefixes the namespaces with a configurable prefix. This means every Wordpress plugin or Nextcloud app can have its own set of dependencies, while they will never cause a conflict thanks to the unambiguous namespaces.

I think this is brilliant, and possibly the only real solution to this problem unless that would be ever addresses on a language level where more than one autoloader could handle the loading of classes inside encapsulated bundles.

A few experiments with the tool in a Nextcloud app have shown that it does work. The only composer package where it broke is Guzzle with HTTPlug, as some libraries are not correctly re-written and thus the service locator logic fails.

*Und alles ruft* [*Come and rock me Amadeus*](https://www.youtube.com/watch?v=cVikZ8Oe_XA)

For the discussion around Mozart for Nextcloud apps please see [my topic on the community forum](https://help.nextcloud.com/t/escaping-the-dependency-hell/81407).
