---
layout: single
title: Working Title I Have To Change
comments: false
date: 2018-03-20
tags:
  - foss
  - nextcloud
  - php
  - programming
#header:
#  image: /assets/20180313_state-of-firefox-u2f/header.png
---

Nextcloud 14 will be the first major version of Nextcloud that requires php7
or higher. Apps targeting this version can take full advantage of language
features that couldn't be used before due to backwards-compatibility to php5.6.


I've started to migrate two of my apps to php7 syntax, mostly by adding
primitive type hints for method arguments and return type hints. This blog post
shall give an overview of how I approached it.


This post is structured as sections of applied changes to different parts of
the code, of which many are specific to my apps and might not be necessary for
other apps. I recommend to commit after each step to be able to go back in case
some changes break your app or the CI setup. Moreover, it doesn't hurt to run
unit tests locally after changing the source code to detect failing tests
earlier.

## Update info.xml

First of all, the Nextcloud app metadata needs an update to reflect that the
app now targets Nextcloud 14 and php7. Thus, the `dependencies` section needs
an update to look like this:

```xml
<dependencies>
  <php min-version="7.0" max-version="7.2" />
  <nextcloud min-version="14" max-version="14" />
</dependencies>
```

## Update `composer.json`

Usually, I specify the minimum php version in `composer.json` and thus this
needs an update to reflect the changes above:
```json
"platform": {
  "php": ">=7.0.0"
},
```

Make sure your composer dependencies are compatible with all targeted php
versions.
{: .notice--info }

You might have to run `composer update` to update the `composer.lock` file.
{: .notice--info }

## Update OCP library

I've created a [composer library that bundles the public Nextcloud API](/2017/09/26/nextcloud-app-api-check.html)
as package, solely to allow checking API compatibility by static code analysis
tools like [Scrutinizer](https://scrutinizer-ci.com/) when the Nextcloud server
code is not available otherwise.


The library is up to date with the latest master (I've got a cron job that
warns me when it's outdated and thus I give it a bump about twice a week).


The `14.0.0.x-dev` version tracks the latest development version of the
Nextcloud server. Specify it in `composer.json` and run
```bash
composer update christophwurst/nextcloud
```
to update to the latest version.


## Update Travis Configuration

For apps using Travis CI as build service it is necessary to change the build
matrix in `.travis.yml` to only test with supported php versions. That would be
`7`, `7.1` and `7.2`:
```yml
php:
  - 7
  - 7.1
  - 7.2
```

## Type Hint Primitives

Now that the basic setup is ready for php7 code, we can start adding type hints
to function and method arguments that are primitives like `string`s and `int`s.

```php
<?php

namespace OCA\MyApp\Controller;

use OCP\AppFramework\Controller;
use OCP\AppFramework\Http\JSONResponse;

class MyController extends Controller {
	
	/**
	 * @return JSONResponse
	 */
	public function get(int $id) {
		// …
	}
	
}

```

## Return Type Hints

Php7 also lets us use type hints for return values. This is useful for method
return types like controllers methods, where you type hint the exact `Response`
type:

```php
<?php

namespace OCA\MyApp\Controller;

use OCP\AppFramework\Controller;
use OCP\AppFramework\Http\JSONResponse;

class MyController extends Controller {
	
	public function get(int $id): JSONResponse {
		// …
	}
	
}

```

## Strict Type Checking

To enforce [strict type checking](http://php.net/manual/en/functions.arguments.php#functions.arguments.type-declaration.strict)
and disable coercing, we have to add a declaration statement to every source
file:
```php
<?php

declare(strict_types = 1);

namespace OCA\MyApp\Controller;

use OCP\AppFramework\Controller;
use OCP\AppFramework\Http\JSONResponse;

class MyController extends Controller {
	
	public function get(int $id): JSONResponse {
		// …
	}
	
}
```

## Push and Pray

Now it's time to push the changes to your CI service and hope that all builds
finish successfully :robot:.
