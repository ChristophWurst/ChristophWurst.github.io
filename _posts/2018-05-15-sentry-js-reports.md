---
layout: single
title: JavaScript Error Reporting in Nextcloud
comments: false
date: 2018-05-15
tags:
  - foss
  - nextcloud
  - sentry
  - javascript
header:
  image: /assets/20180515_sentry_js_reports/banner.png
  image_description: "Sentry + Nextcloud"
---

A couple of months ago, I [created a Sentry integration for Nextcloud](/2017/11/20/nextcloud-sentry-integration.html).
Back then, I focussed on integrating the PHP SDK to automatically report all logged PHP exceptions to a Sentry instance.


## JavaScript Exception Reporting

This week, I've extended the app to also track JavaScript exceptions via the [Raven.js client](https://docs.sentry.io/clients/javascript/).
I hope that this will help us discover and fix some bugs that hadn't been noticed.


Right now, the Sentry client does its magic in detecting those exceptions. The APIs to
[manually report errors](https://docs.sentry.io/clients/javascript/#manually-reporting-errors) are not yet exposed.


## Installation

The app is available via the [app store](https://apps.nextcloud.com/apps/sentry). If you have already used the previous
version, Nextcloud will soon ask you to update to 2.0.0.


If you have any questions, discovered a bug or want to request a new feature, feel free to create a ticket [on GitHub](https://github.com/ChristophWurst/nextcloud_sentry/issues/new/choose).
