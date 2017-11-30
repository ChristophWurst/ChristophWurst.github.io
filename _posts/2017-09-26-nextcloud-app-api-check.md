---
layout: single
title: Automated Nextcloud public API compatibility analysis
date: 2017-09-26
comments: true
tags: nextcloud
---

Most [Nextcloud][1] apps I contribute to seem to be using [Scrutinizer][2] as
code quality analysis tool for their pull requests on GitHub. I really like how
they analyze the source code and report issues neither I nor my IDE would have
spotted. Thus it helps a lot to produce maintainable and stable Nextcloud apps.

However, I've often seen Scrutinizer complaining about missing classes or
interfaces from the `OCP` namespace, which is Nextcloud's public API namespace.
Since Scrutinizer just checks out the app's repository to do their checks, it
does not have access to Nextcloud server sources. Hence, it's impossible for
the analyzers to detect API breakage.

To circumvent this, I've created a composer package that contains Nextcloud's
public API. It's meant to be added as dev dependency and it does not contain
any autoloading configuration, therefore it shouldn't cause any runtime errors
or conflicts. Having it as dev dependency also means that it's not included
in any app release tarballs, provided that you don't include them when you
build the app archives.

You can see how I've added it to [the Mail app][3]. If you're already using
Scrutinizer (or similar) you just have to add it as (another) dev dependency
in your `composer.json` file or use the `composer` command like

```
composer require --dev christophwurst/nextcloud
```

That's it -- you now have an automated CI check
for your Nextcloud public API compatibility 🚀

Currently I've only built a package for Nextcloud 12, as it's the most recent
stable release of the server component. As soon as Nextcloud 13 is out, I'll
update the package for the new release.

You can find the source code [on GitHub][4]. The composer package is
distributed via [packagist.org][5]

[1]: https://nextcloud.com/
[2]: https://scrutinizer-ci.com/
[3]: https://github.com/nextcloud/mail/pull/539
[4]: https://github.com/ChristophWurst/nextcloud_composer
[5]: https://packagist.org/packages/christophwurst/nextcloud
