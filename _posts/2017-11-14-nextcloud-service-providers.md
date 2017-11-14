---
layout: post
title: Adding service providers to Nextcloud
date: 2017-11-14
comments: true
tags: nextcloud software-design software-architecture
---

Nextcloud apps need some level of bootstrapping code to be run when a request
reaches the Nextcloud server. This code wires up interfaces and classes for the
dependency injection container, registers hook and event listeners and runs
other important tasks.

Currently, most of this can be done by implementing an `Application` class that
represents your app. In its constructor, you do whatever is required for your
app to run. That looks something like this:

{% highlight PHP %}
<?php

namespace OCA\MyApp\AppInfo;

class Application extends \OCP\AppFramework\App {
    public function __construct(array $urlParams = []) {
        parent::__construct('my_app', $urlParams);

        // Setup up DI container
        $container = $this->getContainer();
        $container->registerAlias(IService::class, ServiceImpl::class);
        $container->registerAlias(IAnotherService::class, AnotherImpl::class);
        $container->registerParameter('logLevel', 3);
        $container->registerService('logger', function($c) {
            // …
        });

        // Setu up middleware
        $container->registerMiddleWare(AuthorizationMiddleware::class);
        $container->registerMiddleWare(ExceptionReportingMiddlware::class);
        $container->registerMiddleware(ValidationMiddleware::class);

        // Register event listener
        $userSession = $container->getServer()->getUserSession();

        $userSession->listen('\OC\User', 'postLogin', function ($user) {
            // Do something when a user logs in
        });
    }
}
{% endhighlight %}

While this approach is perfectly valid and good enough for small to
medium-sized apps, the bootstrapping code of larger Nextcloud apps can become badly organized, especially if you want to group code into logical units or blocks.

*Service providers* can help decoupling and splitting this code into several classes following the [single responsibility principle][6]. I first learned about service providers when working on a larger application using the [Laravel framework][1]. Laravel is a PHP web framework with a wide range of available services, which are all bootstrapped and loaded via service provider classes. This makes the code base more manageable and prevents code being loaded that is actually not needed for a particular request. Thus, this not only helps organize code but speeds up the application.

Another important aspect of service providers is that they are run to a known state in the application lifecycle. The two available methods to overwrite and thus to hook into this process are `register` and `boot`. The former is meant to be used to register services for the dependency injection container. The latter one is meant for tasks like registering events. This logical separation makes sure that event aren't fired before their dependencies (other services which service provider has not been invoked yet) are ready.

I studied the exposed interface of Laravel's service providers and some of the implementation and realised that it's actually pretty straight forward to implement such an infrastructure for Nextcloud apps. Therefore I've taken those ideas and created a similar `ServiceProvider` class. To allow partial implementation, I chose to go with an `abstract` class. The two methods bodies of `register` and `boot` are empty by default and shall be overwritten as needed. In contrast to Laravel service providers, which are clever enough to have dependency injection for the `boot` method, I've chosen to pass the `Application` instance and and instance of the `IAppContainer` as parameter of both methods. This should give enough flexibility as the app container even allows to query the server container, which is the container for services of the Nextcloud server. This is the resulting base class:

{% highlight PHP %}
<?php

namespace ChristophWurst\Nextcloud\ServiceProviders;

use OCP\AppFramework\App;
use OCP\AppFramework\IAppContainer;

abstract class ServiceProvider {

    /**
     * Overwrite this method to register your services
     *
     *
     * @param App $app
     * @param IAppContainer $container
     */
    public function register(App $app, IAppContainer $container) {
        // empty by default
    }

    /**
     * This method is called after all services have been registerd
     *
     * @param App $app
     * @param IAppContainer $container
     */
    public function boot(App $app, IAppContainer $container) {
        // empty by default
    }

}
{% endhighlight %}

And an implementation of this class would look like this:

{% highlight PHP %}
<?php

namespace OCA\ServiceProviders\Provider;

use ChristophWurst\Nextcloud\ServiceProviders\ServiceProvider;
use OCA\ServiceProviders\Http\Middleware\AuthorizationMiddleware;
use OCA\ServiceProviders\Http\Middleware\ExceptionReportingMiddleware;
use OCA\ServiceProviders\Http\Middleware\ValidationMiddleware;
use OCP\AppFramework\App;
use OCP\AppFramework\IAppContainer;

class MiddlewareServiceProvider extends ServiceProvider {

    public function register(App $app, IAppContainer $container) {
        $container->registerMiddleWare(ExceptionReportingMiddleware::class);
        $container->registerMiddleWare(AuthorizationMiddleware::class);
        $container->registerMiddleWare(ValidationMiddleware::class);
    }

}
{% endhighlight %}

In my opinion that is clean code which is easy to read and understand. For those who target 100% code coverage this gives the opportunity to easily unit test the service registration mocks could be passed as parameters.

There is one thing left to bring this to life: we have to wire it up with our `Application` class. For this, I chose to implement a convenience *trait*, that provides two methods for application classes. Again, inspired by how well the Laravel framework was designed, there should be very little code needed to do this. Hence I chose to read an instance property `providers`, which is an array of provider class names. The trait reads it and loads the service provider instances to call the methods. The implementation is as simple as this:

{% highlight PHP %}
<?php

namespace ChristophWurst\Nextcloud\ServiceProviders;

use Generator;
use OCP\AppFramework\IAppContainer;

trait ServiceProviders {

    /**
     * @return Generator|ServiceProvider[]
     */
    private function getProviders() {
        if (property_exists($this, 'providers')) {
            $providers = $this->providers;
        } else {
            $providers = [];
        }

        foreach ($providers as $provider) {
            $providerInstance = new $provider();
            yield $providerInstance;
        }
    }

    /**
     * Call 'register' an all registered providers
     */
    protected function registerServiceProviders() {
        /* @var $container IAppContainer */
        $container = $this->getContainer();

        foreach ($this->getProviders() as $provider) {
            $provider->register($this, $container);
        }
    }

    /**
     * Call 'boot' on all registered providers
     */
    protected function bootServiceProviders() {
        /* @var $container IAppContainer */
        $container = $this->getContainer();

        foreach ($this->getProviders() as $provider) {
            $provider->boot($this, $container);
        }
    }

}
{% endhighlight %}

I created a demo app to demonstrate the advanced usage of this mechanism which you can find [on GitHub][3]. It registers various [service providers][5] that bootstrap a larger application.

```
lib
├── AppInfo
│   └── Application.php
├── Command
│   └── AddCommand.php
├── Contracts
│   └── ICalculator.php
├── Http
│   └── Middleware
│       ├── AuthorizationMiddleware.php
│       ├── ExceptionReportingMiddleware.php
│       └── ValidationMiddleware.php
├── Provider
│   ├── CalculatorServiceProvider.php
│   ├── EventServiceProvider.php
│   ├── MiddlewareServiceProvider.php
│   └── RouteServiceProvider.php
└── Service
    └── CalculatorService.php
```

You can find the source code [on GitHub][2]. To allow the usage of the above code in apps, I've also published it as composer package on [packagist.org][4].

[1]: https://laravel.com/
[2]: https://github.com/ChristophWurst/nextcloud_serviceproviders
[3]: https://github.com/ChristophWurst/nextcloud_experiments/tree/master/service_providers
[4]: https://packagist.org/packages/christophwurst/nextcloudserviceprovider
[5]: htps://github.com/ChristophWurst/nextcloud_experiments/tree/master/service_providers/lib/Provider
[6]: https://en.wikipedia.org/wiki/Single_responsibility_principle
