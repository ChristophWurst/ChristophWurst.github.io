---
layout: post
title: Testing a Nextcloud app with Karma, Jasmine and PhantomJS
comments: false
tags: nextcloud, javascript, testing, howto
---

Having automated (unit) tests is an important aspect of a high quality Nextcloud app. Tests will help you detect bugs and regressions earlier, furthermore it helps a lot with refactoring and maintaining an app in the long run. In this blog post I want to show the basic steps to set up a JavaScript testing environment with the help of Karma, Jasmine and PhantomJS.

Goals of this guide:
* [Install all required tools and plugins](#installing-tools-and-plugins)
* [Configure the testing framework](#configure-the-testing-framework)
* [Write some tests](#write-some-tests)
* [Run tests as part of the continuous integration system (Travis CI)](#run-tests-as-part-of-the-continous-integration-system)
* [Collect test code coverage](#collect-test-code-coverage)
* [Track coverage with an external service (Coveralls)](#track-coverage-with-coveralls)

I will assume that we want test our fancy new app called **Rocket**. You can find all the
sample code [on GitHub][ghrepo]. The project is structured as followes:
```
rocket
├── appinfo
├── css
├── img
├── js
│   ├── controller
│   ├── service
│   ├── templates
│   ├── util
│   └── views
├── l10n
├── lib
├── screenshots
├── templates
├── tests
└── vendor
```

## Installing tools and plugins
<!--There are many different testing tools available in the JavaScript universe.-->
First of all, we have to install the NPM packages that provide the tools and
frameworks, their plugins and the testing browser. If you have not yet set
up NPM for your project, see the [NPM documenation on how to create package.json](https://docs.npmjs.com/getting-started/using-a-package.json#creating-a-packagejson).

We need the following packages:
* jasmine
* jasmine-core
* karma
* karma-jasmine
* karma-phantomjs-launcher

You can install those with 
```bash
npm install --save-dev jasmine jasmine-core karma karma-jasmine karma-phantomjs-launcher
```

## Configure the testing framework
Before we can start writing our first test, we have to tell Karma where to find
source and test files, which are then loaded into the browser (phantomjs) for execution
with Jasmine.


Before source and test files of the app are included, we load the frameworks the source
files depend on, e.g. jQuery for DOM manipulation, Backbone for views and Handlebars to
render view templates. This configuration is found under the `files` section.

``init.js`` is excluded because this file is used to bootstrap the app when it's loaded
in the browser. However, since we do not want the app to start when we load it for testing,
we ignore it here.

I've added a little helper script `specHelper.js` which is a polyfill for the `OCA`
namespace that is normally defined by the Nextcloud server. This simple script looks
like this:
```js
(function(global) {

	global.OCA = {};

})(window);
```

The browser to run the tests in is [PhantomJS](http://phantomjs.org/). We add it in the
`browsers` section.

```js
module.exports = function (config) {
	config.set({
		// frameworks to use
		frameworks: [
			'jasmine'
		],

		// list of files / patterns to load in the browser
        files: [
			'../../../../core/vendor/es6-promise/dist/es6-promise.js',
			'../../../../core/vendor/jquery/dist/jquery.js',
			'../../../../core/vendor/underscore/underscore.js',
			'../../../../core/vendor/backbone/backbone.js',
			'../../../../core/vendor/handlebars/handlebars.js',
			'specs/specHelper.js', // Include spec helper before other source/test files
			{pattern: '../**/*.js', included: true}
		],


		// list of files to exclude
		exclude: [
			'../vendor/**/*.js',
			'../init.js'
		],

		// preprocess matching files before serving them to the browser
		preprocessors: {},

		// test results reporter to use
		// possible values: 'dots', 'progress'
		// available reporters: https://npmjs.org/browse/keyword/karma-reporter
		reporters: [
			'progress'
		],

		// start these browsers
		browsers: [
			'PhantomJS'
		]
	});
};
```
This config file is saved to `js/tests/karma.conf.js`.

## Write some tests
Test files (also called spec files) live in the `js/tests` and `js/tests/spec`
directory.

Let's add a simple test for `js/views/rocketview.js`. The source code looks as follows
```js
(function(OCA, $, Backbone) {
	'use strict';

	OCA.Rocket = OCA.Rocket || {};
	OCA.Rocket.Views = OCA.Rocket.Views || {};

	/**
	 * @class OCA.Rocket.Views.RocketView
	 */
	OCA.Rocket.Views.RocketView = Backbone.View.extend({

		events: {
			'click button.launch': '_launch'
		},

		/**
		 * @private
		 * @returns {undefined}
		 */
		_launch: function() {
			// Launch your rocket
			console.info('🚀');
		}
	});

})(OCA, $, Backbone);
```

Withing our test spec `js/tests/spec/rocketviewspec.js` we can create an instance
of the view, invoke methods on it and assert afterwards:
```js
describe('Rocket view', function() {
	var $el,
		view;

	beforeEach(function() {
		$el = $('<div/>');
		view = new OCA.Rocket.Views.RocketView({
			$el: $el
		});
	});

	it('launches a rocket', function() {
		spyOn(console, 'info');

		view._launch(); // Alternative: $el.find('button.launch').click();

		expect(console.info).toHaveBeenCalledWith('🚀');
	});

});
```

You can execute all tests with
```bash
./node_modules/karma/bin/karma start js/tests/karma.conf.js
```
Karma can watch file changes on your filesystem and automatically execute tests.
You may want to add this command to your Makefile if you have one.

## Run tests as part of the continous integration system
Running tests locally is great, but you might also want to run them on your continuous
integration platform (e.g. Travis) to ensure that pull requests do not introduce
regressions.

In case you use Travis, the config looks like this:
```yaml
language: php
php:
  - 7.1

env:
  global:
  - CORE_BRANCH=master

cache:
  directories:
  - "$HOME/.npm"

before_install:
  # Install npm dependencies
  - npm install

  # Check out Nextcloud server
  - cd ..
  - git clone https://github.com/nextcloud/server.git --recursive --depth 1 -b $CORE_BRANCH server
  - mv rocket server/apps/

before_script:
  # Working directory for tests should be the app's root directory
  - cd server/apps/rocket

script:
  # Run JS tests
  - ./node_modules/karma/bin/karma start js/tests/karma.conf.js --single-run
```

A sample test run can be found [here](https://travis-ci.org/ChristophWurst/rocket/builds/208745539).

## Collect test code coverage
To get an overview of which lines of code are being hit by our test code, we can
enable coverage reports for Karma. An additional NPM dependency has be be installed:
``karma-coverage``.

Moreover we have to adjust the Karma config:
```js
// Karma configuration
module.exports = function (config) {
	config.set({
		...

		// preprocess matching files before serving them to the browser
		preprocessors: {
			'../{*.js,!(tests)/**/*.js}': ['coverage']
		},

		// test results reporter to use
		// possible values: 'dots', 'progress'
		// available reporters: https://npmjs.org/browse/keyword/karma-reporter
		reporters: [
			'progress',
			'coverage'
		],

		coverageReporter: {
			type: 'lcov',
			dir: '../../coverage'
		},

		...
	});
};
```

When you run your tests again, a directory `coverage` will be created in
your app's root directory, containing detailed reports about your test run.
You can view the HTML version in your browser.

## Track coverage with Coveralls
I recommend to publish your coverage results to services like [Coveralls](http://coveralls.io/),
which visualizes the result on their website and show the coverage diff for
pull requests.

Simply add the NPM package `coveralls` and run
```bash
cat ./coverage/*/lcov.info | ./node_modules/coveralls/bin/coveralls.js
```
after tests were run on Travis. Make sure you have enabled the repository
on Coveralls before you push your next commit.

A sample coverage report can be found [here](https://coveralls.io/github/ChristophWurst/rocket).

## References
* [GitHub repository with the sample code][ghrepo]
* [Jasmine](https://jasmine.github.io/)


[ghrepo]: https://github.com/ChristophWurst/rocket "GitHub repository"
