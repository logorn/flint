Flint
=====

[![Build Status](https://travis-ci.org/henrikbjorn/Flint.png?branch=master)](https://travis-ci.org/henrikbjorn/Flint)

Flint is build on top of [Silex](http://silex.sensionlabs.org) and tries to bring structure, conventions and a couple of [Symfony](http://symfony.com) features.

What is Different from Silex
----------------------------

Nothing and everything. Everything Silex does, Flint does aswell. This means that it is fully backwards compatible. So if closures are your thing, then dont stop using does but still get some benefit.

* Uses the full router instead of the url matcher. This gives the more power to the developer.
* Supports using `xml|yml|php` files for router configuration.
* Custom `ControllerResolver` that knows how to inject your `Application` into controllers.
* A base `Controller` with helper methods that matches the one found in [FrameworBundle](http://github.com/symfony/frameworkbundle).
* Custom error pages by using the default exception handler from Symfony.
* [Twig](http://twig.sensiolabs.org) is enabled by default.

Getting started
---------------

For starting up a new project the easiest way is to use [Composer](http://getcomposer.org) and [Flint-Skeleton](http://github.com/henrikbjorn/flint-skeleton).

``` bash
$ php composer.phar create-project -s dev henrikbjorn/flint-skeleton my-flint-application
```

Or if you are migrating from a Silex project change your `composer.json` file to require Flint and change the Application class that is used.

``` bash
$ php composer.phar require henrikbjorn/flint:~1.0
```

``` php
<?php

use Flint\Application;

$application = new Application($rootDir, $debug);
```

It is recommended to subclass `Flint\Application` instead of using the application class directly.

Controllers
-----------

Flint tries to make Silex more like Symfony. And by using closures it is hard to seperate controllers in a logic way when you have more than a
couple of them. To make it better it is recommended to use classes and methods for controllers. The basics is [explained here](http://silex.sensiolabs.org/doc/usage.html#controllers-in-classes)
but Flint takes it further and allows the application to be injected into a controller.

The first way to accomplish this is by implementing `ApplicationAwareInterface` or extending `ApplicationAware`. This works exactly [as described in Symfony](http://symfony.com/doc/2.0/book/controller.html#the-base-controller-class).
With the only exception that the property is called `$app` instead of `$container`.

``` php
<?php

namespace Acme\Controller;

use Flint\ApplicationAware;

class HelloController extends ApplicationAware
{
    public function indexAction()
    {
        return $this->app['twig']->render('Hello/index.html.twig');
    }
}
```

The other way is to use a base controller. Flint ships with one that mimics most of the one provider with Symfony. To see what methods it implements
go look at the source code for `Flint\Controller\Controller`.

``` php
<?php

namespace Acme\Controller;

use Flint\Controller\Controller;

class HelloController extends Controller
{
    public function indexAction()
    {
        return $this->render('Hello/index.html.twig');
    }
}
```

Routing
-------

Because Flint replaces the url matcher used in Symfony with the full router implementation a lot of new things is possible.

Caching is one of thoose things. It makes your application faster as it does not need to register routes on every request.
Together with loading your routes from a configuration file like Symfony it will also bring more structure to your application.

To enable caching you just need to point the router to the directory you want to use and if it should cache or not. By default the
`debug` parameter will be used as to determaine if cache should be used or not.

``` php
<?php

// .. create a $app before this line
$app->inject(array(
    'routing.options' => array(
        'cache_dir' => '/my/cache/directory/routing',
    ),
));
```

Before it is possible to use the full power of caching it is needed to use configuration files because Silex will always call
add routes via its convenience methods `get|post|delete|put`. Furtunately this is baked right in.

```
<?php

// .. create $app
$app->inject(array(
    'routing.resource' => 'config/routing.xml',
));
```

This will make the router load that resource by default. Here xml is used as an example but `php` is also supported together with
`yml` if `Symfony\Component\Yaml\Yaml' is autoloadable.

The benefit from doing it this way is of course they can be cached but also it allows you to import routing files that are included
in libraries and even other Symfony bundles such as the [WebProfilerBundle](https://github.com/symfony/webprofilerbundle). Also it will make it easier to generate routes from
inside your views.

``` jinja
<a href="{{ app.router.generate('homepage') }}">Homepage</a>
```

This is also possible with Silex but with a more verbose syntax.

Error Pages
-----------

When finished a project or application it is the small things that matter the most. Such as having a custom error page instead of the one
Silex provides by default. Also it can help a lost user navigate back. Flint makes this possible by using the exception handler from Symfony 
and a dedicated controller. Both the views and the controller can be overrriden.

This will only work when debug is turned off.

To override the error pages the same logic is used as inside Symfony.
The logic is very well described [in their documentation](http://symfony.com/doc/master/cookbook/controller/error_pages.html).

Only difference from Symfony is the templates must be created inside `views/Excetion/` directory. Inside the templates there is
access to `app` which in turns gives you access to all of the services defined. 

To override the controller used by the exception handler change the `exception_controller` parameter. This parameter will by default
be set to `Flint\\Controller\\ExceptionController::showAction`.

``` php
<?php

// .. create $app
$app->inject(array(
    'exception_controller' => 'Acme\\Controller\\ExceptionController::showAction',
));
```

To see what parameter the controller action takes look at the one provided by default. Normally it should not be overwritten as it already
gives a lot of flexibilty with the template lookup.

Feedback
--------

Please provide feedback on everything (code, structure, idea etc) over twitter or email.

Who
---

Build by [@henrikbjorn](http://twitter.com/henrikbjorn), [Peytz & Co](http://peytz.dk) and [other contributors](https://github.com/henrikbjorn/flint/graphs/contributors).
