---
layout: single
title: Testing Nextcloud Apps
comments: false
date: 2018-02-27
tags:
  - nextcloud
  - programming
  - foss
  - testing
header:
  image: /assets/20180227_testing_nextcloud_app/test_pyramid_banner.png
---

I like having my Nextcloud apps tested automatically both locally and on
continuous interation build services like [Travis CI]. While those automated
tests will never cover 100% of the code base, they give me some level of
confidence that a changeset does not break any features.


Testing is a complex topic with various aspects. I like the idea of
[the test pyramid] that categorizes tests into three categories:

![The Test Pyramid: Unit tests, system tests and UI tests.](/assets/20180227_testing_nextcloud_app/test_pyramid_banner.png)

* Unit tests are the foundation of automated tests. They ensure that each
component (e.g. class or function) works in isolation. The *subject under
test* is not allowed to talk to any other components. Instead, they are
*mocked* to ensure isolation and prevent side effects.
* System tests or *integration tests* ensure components also work when put
together. These tests are allowed to talk to other components and external
systems like databases.
* UI tests are the most complex tests as these test all layers of the software.
For Nextcloud apps that have a user interface this means clicking
around in the web interface.


## Writing Test Cases

The Nextcloud server repository offers a versatile base class for automated
test cases, the ``\Test\Testcase`` class. This class is recommended by the
[development manual]. While this class offers a lot of great advantages,
I don't use it in my apps because of the following reasons.

### Speed

I want to test often, ideally after every change in either my source or
test code. If tests are slow, you tend to execute them rarely or maybe
event don't run them locally at all. If you're lucky all tests pass
on CI on your first push. But in many cases, you break a test case you
didn't even consider relevant for you changes. And having to wait for the
CI system to pick up your pull request's builds may take a few minutes
during working ours and you'll do other work inbetween, loosing focus on
your current task.


Thus, I like to have *fast* tests. Especially unit tests should be fast as
they run in isolation and thus don't need any preparation/cleanup steps
that otherwise tend to come with a certain overhead.

### Unexpected Side Effects

If you've ever tried to *just run* any tests that are based on the server
test case than you've probably noticed that your test instance is borked.
It took me a while to figure that out but it has to do with the cleanup
steps taken in the base class which seem to be too agressive for my use
cases. The class is meant to be run via the provided [shell script](https://github.com/nextcloud/server/blob/master/autotest.sh)
which will create a separate data directory for your tests.


The apps I develop don't really use any files and thus this extensive cleanup
is unnecessary.


## Writing my own Testing Framework

Since I haven't been using the Nextcloud test case for a while and instead,
used PHPUnit classes directly, I've started to write small abstractions
of those classes for the [Mail app] and the [TOTP two-factor auth app]. For
Mail I had developed integration tests that essentially were unit tests that
ran in a database transaction to reset the instance after each test run to
the state before the test ran. For the TOTP app I wrote my own test base
class to run Selenium UI tests on [Sauce Labs].


Then I realized that I could merge those two and create my own (micro) testing
framework that is both fitting my needs and still generic enough to be used
in other apps.

### One class to rule them all

This micro framework consists of a single abstract base class and a few traits.
The goal is to use the same base class for *all* tests and depending on which
traits are included, specific tasks are executed before or after the test.


### Unit Tests

For unit tests, ``TestCase`` doesn't have any noticable overhead compared to
running tests purely on PHPUnit.

### Integration Tests

To run tests that aren't isolated and thus produce side effects, I've created
``DatabaseTransaction`` trait, that when included starts a transaction before
each test in the test class that is rolled back after each test execution. Not
only is it much easier to reset a database via a transaction than via manual
queries, it has shown to be a lot faster.


This concept was taken from the Laravel framework where integration tests
are handled [in a similar way](https://laravel.com/docs/5.1/testing#resetting-the-database-after-each-test).

### Selenium Tests

The [TOTP two-factor auth app] is UI tested based on Selenium and [Sauce
Labs]. When the ``Selenium`` trait is used, a ``WebDriver`` instance can be
accessed via ``$this->webDriver`` - ready to execute any actions on the web
interface. The configuration (credentials for SL) is read from environment
variables so that it works both locally and on CI where the password is
encryted and injected via the environment.


## Recap

There are a few aspects I dislike on the default test base class provided by
the Nextcloud server, so I've created my own micro testing framework. It's
simple, fast and flexible and allows to write unit, integration and UI tests.

If you'd like to give this framework a try, you can install it via composer:

```bash
composer install --save-dev christophwurst/nextcloud_testing
```

All classes are autoloaded. See the [repo readme](https://github.com/ChristophWurst/nextcloud_testing)
for examples of all three types of test cases. If there are any questions or
bugs, please submit them as [issues on GitHub](https://github.com/ChristophWurst/nextcloud_testing/issues/new).


[Travis CI]: https://travis-ci.org/
[the test pyramid]: https://martinfowler.com/articles/practical-test-pyramid.html#TheTestPyramid
[development manual]: https://docs.nextcloud.com/server/13/developer_manual/app/
[Mail app]: https://github.com/nextcloud/mail/
[TOTP two-factor auth app]: https://github.com/nextcloud/twofactor_totp
[Sauce Labs]: https://saucelabs.com/
