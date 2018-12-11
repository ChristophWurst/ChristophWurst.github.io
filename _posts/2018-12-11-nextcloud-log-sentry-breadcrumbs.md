---
layout: single
title: Using Nextcloud Logs as Sentry Breadcrumbs
comments: true
date: 2018-12-11
last_modified_at: 2018-12-11
tags:
  - nextcloud
  - sentry
  - foss
header:
  image: /assets/20181211_nextcloud_log_sentr_breadcrumbs/banner.png
---

The [Sentry](https://sentry.io) integration for [Nextcloud](https://nextcloud.com/) has been
available for more than a year now. During this year, we gradually improved the data that is
reported and the errors it can capture.

For the new Nextcloud 15 we added support for Sentry's breadcrumbs. These had been collected
by the Sentry client on the client side already. However, they needed special handling on the
server side to work with Nextcloud's logging service. A small adjustment to the logger now
passes logged messages on to the "crash reporters" (an abstract term for integrations like
Sentry). The Sentry integration app then registers the messages as Sentry breadcrumbs.

![](/assets/20181211_nextcloud_log_sentr_breadcrumbs/breadcrumbs.png)
