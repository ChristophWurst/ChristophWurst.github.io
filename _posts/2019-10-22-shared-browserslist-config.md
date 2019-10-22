---
layout: single
title: A Shared Browserslist Configuration for Nextcloud Apps
comments: true
date: 2019-10-22
last_modified_at: 2019-10-22
tags:
  - nextcloud
  - development
  - javascript
  - foss
---

In times where most JavaScript is not written for the browser directly but runs through one (or many) tools that transform the syntax and APIs used, it has become a lot easier to be compatible with a wider range of browsers. In order to keep the size of the generated code reasonable, it's necessary to limit the set of browsers and browser versions supported.

[Browserslist](https://github.com/browserslist/browserslist) allows developers to declare these browsers and versions, from which it generates the support matrix. Other tools can use this information when the transpile or generate code. One of them is [Babel](https://babeljs.io/), the JavaScript compiler.

If you develop an app for Nextcloud, it makes sense to use the same support matrix to ensure that your code is compatible with the same browsers. For this purpose we created and released a shared Browserslist config: [`@nextcloud/browserslist-config`](https://www.npmjs.com/package/@nextcloud/browserslist-config).

To use the shared config, install it via `npm` and let the share config extend your specific one (if any). Details about installation and usage can be found [on the package website](https://www.npmjs.com/package/@nextcloud/browserslist-config#installation).

I hope this is useful for Nextcloud app developers. Any feedback or changes on the support matrix may be submitted [on Github](https://github.com/nextcloud/browserslist-config) :v:
