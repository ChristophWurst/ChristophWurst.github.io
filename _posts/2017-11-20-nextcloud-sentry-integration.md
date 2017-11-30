---
layout: single
title: Integrating Sentry crash reporting into Nextcloud
date: 2017-11-20
comments: false
tags: nextcloud sentry
---

[Sentry][1] is an application crash reporting tool that collects and aggregates
application crashes from various frameworks and programming languages. I've
been using Sentry's PHP SDK for a few [Laravel][2] [projects][3]. So far,
this has helped me track down bugs that possibly would have gotten lost in log
files which I rarely look at. Moreover, the Sentry SDK captures a lot of additional
information about the crash like user IDs, application host and URLs. You can even
set breadcrumbs to know which actions the users did before the app crashed.
After the gathered information has been processed you can inspect it via a web
user interface:

![](/assets/20171120_nextcloud_sentry/00dbffd7-6029-4bf1-b656-afde432b2e03.png)

## Integration into Nextcloud

I've been thinking about a Nextcloud integration for a long time, but never really
found the time to actually do it. Until last Sunday, where the weather was quite bad
and I had nothing better to do than spending the afternoon on building such an
integration 🤓.


The integration was implemented as a Nextcloud app. The app is pretty small, as only
a few lines of code were necessary to bootstrap the Sentry client and listen to
unhandled exceptions. The more tricky part was to find a spot in the Nextcloud server
component to hook into and listen to those exception reports. Since no such
mechanism existed, I had to add it myself. After digging into the exception handling
in Nextcloud, I've found out that almost all exceptions are somehow handled. That means
the default exception/error handlers PHP allows you to listen to don't get triggered.
However, Nextcloud almost always logs those exceptions to the `ILogger` interface. [A
little modification to the logger implementation][4] allows apps to register
crash reporters that will be triggered for any handled and logged exception. Nextcloud
13 will be the first release that supports these reporters.


The Sentry crash reporter app for Nextcloud will be available on the [app store][5] as
soon as the [app certificate has been generated][8]. In the meanwhile, you can check out
the code [on GitHub][6].


If you have any questions or suggestions, please let me know in the [GitHub issue tracker][7]!


[1]: https://sentry.io/
[2]: https://github.com/ChristophWurst/weinstein_server
[3]: https://github.com/winzerhof-wurst/web
[4]: https://github.com/nextcloud/server/pull/7151
[5]: https://apps.nextcloud.com/
[6]: https://github.com/christophwurst/nextcloud_sentry
[7]: https://github.com/ChristophWurst/nextcloud_sentry/issues
[8]: https://github.com/nextcloud/app-certificate-requests/pull/107