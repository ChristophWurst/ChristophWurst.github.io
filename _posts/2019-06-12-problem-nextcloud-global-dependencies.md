---
layout: single
title: The Problem of Using Nextcloud's Globally Loaded Dependencies
comments: true
date: 2019-06-12
last_modified_at: 2019-06-12
tags:
  - nextcloud
  - development
  - javascript
  - foss
---

Many Nextcloud apps that serve a front-end component have been rewritten in Vue.js recently. With this change to a more modern framework and concept for building user interfaces, more apps have also started using webpack to bundle scripts, styles and even icons. In this post I would like to reflect on why this is important and why more apps should do it.

## 🕵️ Writing Nextcloud app -- the old approach

Before bundlers like webpack became popular in the Nextcloud community, most apps did not use any bundling for their assets. This means scripts and style sheets were included via HTML's `<script>` and `<style>` tags.
This style of front-end programming comes with a few problems and limitations.

### 🐌 Increased number of requests
With the modular architecture of Nextcloud and having loaded many apps, this design results in numerous requests for a page load without a populated cache. The more files (modules) an app has, the more requests the browser has to wait for before the page can fully work.

Some developers start with small modules, others split larger onces when they refactor they code that might otherwise become unmaintainable. However, if you are aware of the problem of too many requests on page load, you might rather inline a small module than separating it into a self-contained module. Eventually, this leads to a lower code quality and less maintainable code. [js.js as example]

### 🧮 Loading order
With splitting code into modules, you will face problems with module dependencies. That means you have to load all your script files in an order in which each module is loaded after the modules it depends on. You can solve this problem with topological sorting, but not all dependency graphs are easy to identify, nor is it always possible to prevent cyclic dependencies. Moreover, these modules are very fragile to refactor, because moving code around can break out due to loosely defined dependence on another module, usually via the assumption of the existence of global variables.

### 💥 Using custom libraries and frameworks
A very simple Nextcloud app might only need plain JavaScript. Maybe it needs jQuery. But how can that app use jQuery? Luckily jQuery is already loaded with Nextcloud as the global ˋ$ˋ variable. The app requires library X? Well, it can just load it with another script tag. All fine, right? Not quite, actually.
The problem we're seeing here is called *dependency hell*. In the context of Nextcloud it means that apps have a hard dependency on the libraries shipped with Nextcloud. They work or break depending on if and which version of a library Nextcloud loads. In return, Nextcloud server developers are very limited in changing (updating/removing) those libraries as the product improves and libraries get outdated. A simple upgrade from jQuery 2.x to 3.x will break all apps that are incompatible with that version, making the Nextcloud server unable to migrate away from that outdated version.

In the same context, apps loading their own libraries will pollute the global namespace as libraries register themselves there 🤦. Should two apps try to load the same library, they will overwrite each other's instance. Again, this is the *dependency hell* problem. With this type of module design, it is impossible to use two version of the same library on the very same page.

### 💣 Browser compatibility
If source files are loaded in the browser directly, they have to be written in a syntax that is compatible with all the browsers that are supported. Sometimes this is easy, sometimes this is hard to test because many developers only develop and test on fast, modern browsers while legacy browsers should be supported as well.

## 🙌 The Solution
In general, the solution to the problems above is to use a bundler tool that combines scripts, styles and other assets into fewer files. In the extreme case, it will combine all front-end assets of an app into a single file.
Right now, one of the widely used tools is *webpack*. Webpack can take JavaScript modules in different formats, analyze their dependency tree and combine them to a bundle. In essence, you just have to tell webpack your *entry point* and it will figure out the rest, based on the dependcy declarations in the source modules.
Therefore this solves the problem of too many request on page load. With a bundler like webpack, having many small modules does not have any negative impact on your page performance anymore. Developers can properly structure chunks of codes into small, reusable modules and webpack will do its magic and combine them for the browser.

In terms of dependency hell with more than one version of a library, webpack can load modules installed via packages of *npm*, the node package manager. With this apps can install an arbitrary version of their software dependencies and bundle them with webpack, without having any impact on other code. The dependencies won't be registered in the global namespace anymore. Instead, webpack will inject them wherever they are imported in modules. This not only keeps the global namespace clean, it will also perform better as the JavaScript interpreter can find the references to libraries quicker when they are declared locally.


## 🤔 The Long-term Goal
In the long run, it would be great if more apps switch to webpack or a similar tool. Because then the server component will be independent of apps again, so that its code and dependencies can be managed, updated and removed independently without breaking any apps that rely on them.
