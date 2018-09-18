---
layout: single
title: Minimal Travis CI Config for Nextcloud Apps
comments: false
date: 2018-09-18
tags:
  - foss
  - nextcloud
  - travis
  - testing
---

[Travis CI](https://travis-ci.org/) offers free continuous integration services
for open source projects on GitHub, so it's a natural fit for Nextcloud
apps. In this blog post, I'd like to give an introduction to continuous integration
(CI) testing of an arbitrary Nextcloud app.

## Prerequisites

This post assumes that

* you know the basics of [Nextcloud app development](https://docs.nextcloud.com/server/14/developer_manual/index.html)
* you are familiar with [Composer](https://getcomposer.org/)
* your app uses [PHPUnit](https://phpunit.de/) or a [derivate as covered in a previous post](/2018/02/27/testing-nextcloud-app.html)

## Build lifecycle

Travis has a defined [build lifecycle](https://docs.travis-ci.com/user/customizing-the-build/#the-build-lifecycle) we
can use to structure the CI build.

### `before_install`

Nextcloud apps aren't standalone applications, as in they always require an installation of the Nextcloud
Server component. Travis will just check out the app's git repository, so we have to tweak this a bit.

The simplest approach is to clone the Server repository as well and move the app's directory into Nextcloud's `apps`
directory:


```yml
before_install:
  # Check out the Nextcloud server via git
  - cd ..
  - git clone https://github.com/nextcloud/server.git --recursive --depth 1 -b $SERVER_BRANCH

  # Move the app under test into Nextcloud's `apps` directory
  - mv <your-app> server/apps/
```

As you might have noticed, there's a environment variable `$SERVER_BRANCH` used in the snippet above. We'll cover 
its purpose later in this post.
{: .notice--info }

The Nextcloud Server does not have to be actually runnable via a web server unless you want to run acceptance tests.
{: .notice--info }


### `install`

In this step we first install app dependencies and enable the app in the Nextcloud Server:

```yml
install:
  # Install the server
  - php -f server/occ maintenance:install --database-name nc_autotest --database-user nc_autotest --admin-user admin --admin-pass admin --database $DB --database-pass=''

  # Enable debug mode to get more info in case a test fails
  - php -f server/occ config:system:set debug --value=true --type boolean

  # Install composer dependencies
  - pushd server/apps/<your-app>
  - composer install
  - popd

  # Set up app
  - php -f server/occ app:enable <your-app>
```

### `before_script`

As a preparation for our actual test execution, we have to make sure to switch back
to the app directory:

```yml
before_script:
  # Switch bach to the app directory
  - cd server/apps/<your-app>
```

### `script`

This is the heart of CI builds. In this step you'll run all checks and tests:

```yml
script:
  # Lint php files
  - find . -name \*.php -not -path './vendor/*' -exec php -l "{}" \;

  # Run unit/integration tests
  - ./vendor/bin/phpunit lib
```

Now we have defined the necessary build instructions for our Tests. What's still
missing is telling Travis some basic configuration about the type of build we
want to do and what version of php it shall install.

## Build configuration

Travis supports various programming languages and versions of these, so we have to
tell it that we're building a php application. Moreover, we'll specify the php versions
we want to have covered and some global variables that allow more complex test scenarios:

```yml
language: php
php:
  - 7.0
  - 7.1
  - 7.2

env:
  global:
  - SERVER_BRANCH=master
  matrix:
  - DB=sqlite
```

The `$SERVER_BRANCH` variable determines the branch to check out of the Server repository.
This is typically the `master` branch as well as one of the latest stable releases like
`stable14.0.0`. In this minimal config, Travis will only test against Server `master`, but
you can add more specific options via the [build matrix](https://docs.travis-ci.com/user/customizing-the-build/#build-matrix).

The same applies to the `$DB` variable which tells Nextcloud to install on a `sqlite` database.
You can (and should) test your app against all supported databases, hence a more complete Travis
config will set up MariaDB and PostgreSQL as well and add those to the [build matrix](https://docs.travis-ci.com/user/customizing-the-build/#build-matrix).

## Bonus: Cache rules everything around me

You might notice that your builds spend a noticeably (long) time on installing dependencies. We
can speed that up by [caching composer files](https://blog.travis-ci.com/2013-12-05-speed-up-your-builds-cache-your-dependencies):

```yml
cache:
  directories:
  - "$HOME/.composer/cache/files"
```

## Conclusion

This simple config should be a good starting point for testing your Nextcloud app on
Travis. The full `travis.yml` file covered in this post looks like this:

```yml
language: php
php:
  - 7.0
  - 7.1
  - 7.2

env:
  global:
  - SERVER_BRANCH=master
  matrix:
  - DB=sqlite

branches:
  only:
  - master

cache:
  directories:
  - "$HOME/.composer/cache/files"

before_install:
  # Check out the Nextcloud server via git
  - cd ..
  - git clone https://github.com/nextcloud/server.git --recursive --depth 1 -b $SERVER_BRANCH

  # Move the app under test into Nextcloud's `apps` directory
  - mv <your-app> server/apps/

install:
  # Install the server
  - php -f server/occ maintenance:install --database-name nc_autotest --database-user nc_autotest --admin-user admin --admin-pass admin --database $DB --database-pass=''

  # Enable debug mode to get more info in case a test fails
  - php -f server/occ config:system:set debug --value=true --type boolean

  # Install composer dependencies
  - pushd server/apps/<your-app>
  - composer install
  - popd

  # Set up app
  - php -f server/occ app:enable <your-app>

before_script:
  # Switch bach to the app directory
  - cd server/apps/<your-app>

script:
  # Lint php files
  - find . -name \*.php -not -path './vendor/*' -exec php -l "{}" \;

  # Run unit/integration tests
  - ./vendor/bin/phpunit lib
```

## References
* [Testing Nextcloud Apps](/2018/02/27/testing-nextcloud-app.html)
* [Building a PHP project on Travis](https://docs.travis-ci.com/user/languages/php)
* [Customizing the Travis Build](https://docs.travis-ci.com/user/customizing-the-build/)
