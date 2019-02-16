## Routing

Beautiful URLs are an absolute must for any serious web application. This means leaving behind ugly URLs like `index.php?article_id=57` in favor of something like `/read/intro-to-symfony`.

Having flexibility is even more important. What if you need to change the URL of a page from `/blog` to `/news`? How many links should you need to hunt down and update to make the change? If you're using Symfony's router, the change is simple.

The Symfony router lets you define creative URLs that you map to different areas of your application. By the end of this article, you'll be able to:

- Create complex routes that map to controllers
- Generate URLs inside templates and controllers
- Load routing resources from bundles (or anywhere else)
- Debug your routes

## Routing Examples

A route is a map from a URL path to a controller. For example, suppose you want to match any URL like `/blog/my-post` or `/blog/all-about-symfony` and send it to a controller that can look up and render that blog post. The route is simple:

```config
~~~php {.annotation}
// src/AppBundle/Controller/BlogController.php
namespace AppBundle\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\Controller;
use Symfony\Component\Routing\Annotation\Route;

class BlogController extends Controller
{
    /**
     * Matches /blog exactly
     *
     * @Route("/blog", name="blog_list")
     */
    public function listAction()
    {
        // ...
    }

    /**
     * Matches /blog/*
     *
     * @Route("/blog/{slug}", name="blog_show")
     */
    public function showAction($slug)
    {
        // $slug will equal the dynamic part of the URL
        // e.g. at /blog/yay-routing, then $slug='yay-routing'

        // ...
    }
}
~~~

~~~yaml
# app/config/routing.yml
blog_list:
    path:     /blog
    defaults: { _controller: AppBundle:Blog:list }

blog_show:
    path:     /blog/{slug}
    defaults: { _controller: AppBundle:Blog:show }
~~~

~~~xml
<!-- app/config/routing.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<routes xmlns="http://symfony.com/schema/routing"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/routing
        http://symfony.com/schema/routing/routing-1.0.xsd">

    <route id="blog_list" path="/blog">
        <default key="_controller">AppBundle:Blog:list</default>
    </route>

    <route id="blog_show" path="/blog/{slug}">
        <default key="_controller">AppBundle:Blog:show</default>
    </route>
</routes>
~~~

~~~php
// app/config/routing.php
use Symfony\Component\Routing\RouteCollection;
use Symfony\Component\Routing\Route;

$routes = new RouteCollection();
$routes->add('blog_list', new Route('/blog', [
    '_controller' => 'AppBundle:Blog:list',
]));
$routes->add('blog_show', new Route('/blog/{slug}', [
    '_controller' => 'AppBundle:Blog:show',
]));

return $routes;
~~~
```

Thanks to these two routes:

- If the user goes to `/blog`, the first route is matched and `listAction()` is executed;
- If the user goes to `/blog/*`, the second route is matched and `showAction()` is executed. Because the route path is `/blog/{slug}`, a `$slug` variable is passed to `showAction()` matching that value. For example, if the user goes to `/blog/yay-routing`, then `$slug` will equal `yay-routing`.
Whenever you have a `{placeholder}` in your route path, that portion becomes a wildcard: it matches *any* value. Your controller can now *also* have an argument called `$placeholder` (the wildcard and argument names *must* match).

Each route also has an internal name: `blog_list` and `blog_show`. These can be anything (as long as each is unique) and don't have any meaning yet. Later, you'll use it to generate URLs.

```sidebar
Routing in Other Formats

The `@Route` above each method is called an `*annotation*`. If you'd rather configure your routes in YAML, XML or PHP, that's no problem!

In these formats, the `_controller` "defaults" value is a special key that tells Symfony which controller should be executed when a URL matches this route. The `_controller` string is called the [logical name](https://symfony.com/doc/3.4/routing.html#controller-string-syntax). It follows a pattern that points to a specific PHP class and method, in this case the `AppBundle\Controller\BlogController::listAction` and `AppBundle\Controller\BlogController::showAction` methods.
```

This is the goal of the Symfony router: to map the URL of a request to a controller. Along the way, you'll learn all sorts of tricks that make mapping even the most complex URLs easy.

## Adding {wildcard} Requirements
Imagine the `blog_list` route will contain a paginated list of blog posts, with URLs like `/blog/2` and `/blog/3` for pages 2 and 3. If you change the route's path to `/blog/{page}`, you'll have a problem:

- blog_list: `/blog/{page}` will match `/blog/*`
- blog_show: `/blog/{slug}` will also match `/blog/*`

When two routes match the same URL, the *first* route that's loaded wins. Unfortunately, that means that `/blog/yay-routing` will match the `blog_list`. No good!

To fix this, add a requirement that the `{page}` wildcard can *only* match numbers (digits):

```config
~~~php {.annotation}
// src/AppBundle/Controller/BlogController.php
namespace AppBundle\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\Controller;
use Symfony\Component\Routing\Annotation\Route;

class BlogController extends Controller
{
    /**
     * @Route("/blog/{page}", name="blog_list", requirements={"page"="\d+"})
     */
    public function listAction($page)
    {
        // ...
    }

    /**
     * @Route("/blog/{slug}", name="blog_show")
     */
    public function showAction($slug)
    {
        // ...
    }
}
~~~

~~~yaml
# app/config/routing.yml
blog_list:
    path:      /blog/{page}
    defaults:  { _controller: AppBundle:Blog:list }
    requirements:
        page: '\d+'

blog_show:
    # ...
~~~

~~~xml
<!-- app/config/routing.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<routes xmlns="http://symfony.com/schema/routing"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/routing
        http://symfony.com/schema/routing/routing-1.0.xsd">

    <route id="blog_list" path="/blog/{page}">
        <default key="_controller">AppBundle:Blog:list</default>
        <requirement key="page">\d+</requirement>
    </route>

    <!-- ... -->
</routes>
~~~

~~~php
// app/config/routing.php
use Symfony\Component\Routing\RouteCollection;
use Symfony\Component\Routing\Route;

$routes = new RouteCollection();
$routes->add('blog_list', new Route('/blog/{page}', [
    '_controller' => 'AppBundle:Blog:list',
], [
    'page' => '\d+'
]));

// ...

return $routes;
~~~
```

The `\d+` is a regular expression that matches a digit of any length. Now:

URL | Route | Parameters
-----|-----|-----
/blog/2 | blog_list | $page = 2
/blog/yay-routing | blog_show | $slug = yay-routing

To learn about other route requirements - like HTTP method, hostname and dynamic expressions - see [_How to Define Route Requirements_](https://symfony.com/doc/3.4/routing/requirements.html).

## Giving {placeholders} a Default Value

In the previous example, the `blog_list` has a path of `/blog/{page}`. If the user visits `/blog/1`, it will match. But if they visit `/blog`, it will **not** match. As soon as you add a `{placeholder}` to a route, it *must* have a value.

So how can you make `blog_list` once again match when the user visits `/blog`? By adding a default value:

```config
~~~php {.annotation}
// src/AppBundle/Controller/BlogController.php
namespace AppBundle\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\Controller;
use Symfony\Component\Routing\Annotation\Route;

class BlogController extends Controller
{
    /**
     * @Route("/blog/{page}", name="blog_list", requirements={"page"="\d+"})
     */
    public function listAction($page = 1)
    {
        // ...
    }
}
~~~

~~~yaml
# app/config/routing.yml
blog_list:
    path:      /blog/{page}
    defaults:  { _controller: AppBundle:Blog:list, page: 1 }
    requirements:
        page: '\d+'

blog_show:
    # ...
~~~

~~~xml
<!-- app/config/routing.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<routes xmlns="http://symfony.com/schema/routing"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/routing
        http://symfony.com/schema/routing/routing-1.0.xsd">

    <route id="blog_list" path="/blog/{page}">
        <default key="_controller">AppBundle:Blog:list</default>
        <default key="page">1</default>

        <requirement key="page">\d+</requirement>
    </route>

    <!-- ... -->
</routes>
~~~

~~~php
// app/config/routing.php
use Symfony\Component\Routing\RouteCollection;
use Symfony\Component\Routing\Route;

$routes = new RouteCollection();
$routes->add('blog_list', new Route(
    '/blog/{page}',
    [
        '_controller' => 'AppBundle:Blog:list',
        'page'        => 1,
    ],
    [
        'page' => '\d+'
    ]
));

// ...

return $routes;
~~~
```

Now, when the user visits `/blog`, the `blog_list` route will match and `$page` will default to a value of `1`.

## Advanced Routing Example

With all of this in mind, check out this advanced example:

```config
~~~php {.annotation}
// src/AppBundle/Controller/ArticleController.php

// ...
class ArticleController extends Controller
{
    /**
     * @Route(
     *     "/articles/{_locale}/{year}/{slug}.{_format}",
     *     defaults={"_format": "html"},
     *     requirements={
     *         "_locale": "en|fr",
     *         "_format": "html|rss",
     *         "year": "\d+"
     *     }
     * )
     */
    public function showAction($_locale, $year, $slug)
    {
    }
}
~~~

~~~yaml
# app/config/routing.yml
article_show:
  path:     /articles/{_locale}/{year}/{slug}.{_format}
  defaults: { _controller: AppBundle:Article:show, _format: html }
  requirements:
      _locale:  en|fr
      _format:  html|rss
      year:     \d+
~~~

~~~xml
<!-- app/config/routing.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<routes xmlns="http://symfony.com/schema/routing"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/routing
        http://symfony.com/schema/routing/routing-1.0.xsd">

    <route id="article_show"
        path="/articles/{_locale}/{year}/{slug}.{_format}">

        <default key="_controller">AppBundle:Article:show</default>
        <default key="_format">html</default>
        <requirement key="_locale">en|fr</requirement>
        <requirement key="_format">html|rss</requirement>
        <requirement key="year">\d+</requirement>

    </route>
</routes>
~~~

~~~php
// app/config/routing.php
use Symfony\Component\Routing\RouteCollection;
use Symfony\Component\Routing\Route;

$routes = new RouteCollection();
$routes->add(
    'article_show',
    new Route('/articles/{_locale}/{year}/{slug}.{_format}', [
        '_controller' => 'AppBundle:Article:show',
        '_format'     => 'html',
    ], [
        '_locale' => 'en|fr',
        '_format' => 'html|rss',
        'year'    => '\d+',
    ])
);

return $routes;
~~~
```

As you've seen, this route will only match if the `{_locale}` portion of the URL is either `en` or `fr` and if the `{year}` is a number. This route also shows how you can use a dot between placeholders instead of a slash. URLs matching this route might look like:

- `/articles/en/2010/my-post`
- `/articles/fr/2010/my-post.rss`
- `/articles/en/2013/my-latest-post.html`

```sidebar
The Special _format Routing Parameter

This example also highlights the special `_format` routing parameter. When using this parameter, the matched value becomes the "request format" of the `Request` object.

Ultimately, the request format is used for such things as setting the `Content-Type` of the response (e.g. a `json` request format translates into a `Content-Type` of `application/json`). It can also be used in the controller to render a different template for each value of `_format`. The `_format` parameter is a very powerful way to render the same content in different formats.

In Symfony versions previous to 3.0, it is possible to override the request format by adding a query parameter named `_format` (for example: `/foo/bar?_format=json`). Relying on this behavior not only is considered a bad practice but it will complicate the upgrade of your applications to Symfony 3.
```

```note
Sometimes you want to make certain parts of your routes globally configurable. Symfony provides you with a way to do this by leveraging service container parameters. Read more about this in "[How to Use Service Container Parameters in your Routes](https://symfony.com/doc/3.4/routing/service_container_parameters.html)".
```

## Special Routing Parameters

As you've seen, each routing parameter or default value is eventually available as an argument in the controller method. Additionally, there are four parameters that are special: each adds a unique piece of functionality inside your application:

- `_controller`  
    As you've seen, this parameter is used to determine which controller is executed when the route is matched.
- `_format`  
    Used to set the request format ([read more](https://symfony.com/doc/3.4/routing.html#routing-format-param)).
- `_fragment`  
    Used to set the fragment identifier, the optional last part of a URL that starts with a `#` character and is used to identify a portion of a document.

    ```sidebar
    New in version 3.2: The _fragment parameter was introduced in Symfony 3.2.
    ```

- `_locale`  
    Used to set the locale on the request ([read more](https://symfony.com/doc/3.4/translation/locale.html#translation-locale-url)).

## Redirecting URLs with Trailing Slashes

Historically, URLs have followed the UNIX convention of adding trailing slashes for directories (e.g. `https://example.com/foo/`) and removing them to refer to files (`https://example.com/foo`). Although serving different contents for both URLs is OK, nowadays it's common to treat both URLs as the same URL and redirect between them.

Symfony follows this logic to redirect between URLs with and without trailing slashes (but only for `GET` and `HEAD` requests):

Route path | If the requested URL is /foo | If the requested URL is /foo/
-----------|------------------------------|------------------------------
`/foo` | It matches (`200` status response) | It doesn't match (`404` status response)
`/foo/` | It makes a `301` redirect to `/foo/` | It matches (`200` status response)

In summary, adding a trailing slash in the route path is the best way to ensure that both URLs work. [_Read the Redirect URLs_](https://symfony.com/doc/3.4/routing/redirect_trailing_slash.html) with a Trailing Slash article to learn how to avoid the `404` error when the request URL contains a trailing slash and the route path does not.

## Controller Naming Pattern

If you use YAML, XML or PHP route configuration, then each route must have a `_controller` parameter, which dictates which controller should be executed when that route is matched. This parameter uses a simple string pattern called the *logical controller name*, which Symfony maps to a specific PHP method and class. The pattern has three parts, each separated by a colon:

> bundle:controller:action

For example, a `_controller` value of `AppBundle:Blog:show` means:

Bundle | Controller Class |Method Name
-------|------------------|-----------
AppBundle | BlogController | showAction()

The controller might look like this:

```php
// src/AppBundle/Controller/BlogController.php
namespace AppBundle\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\Controller;

class BlogController extends Controller
{
    public function showAction($slug)
    {
        // ...
    }
}
```

Notice that Symfony adds the string `Controller` to the class name (`Blog` => `BlogController`) and `Action` to the method name (`show` => `showAction()`).

You could also refer to this controller using its fully-qualified class name and method: `AppBundle\Controller\BlogController::showAction`. But if you follow some simple conventions, the logical name is more concise and allows more flexibility.

```tip
To refer to an action that is implemented as the `__invoke()` method of a controller class, you do not have to pass the method name, but can just use the fully qualified class name (e.g. `AppBundle\Controller\BlogController`).
```

```note
In addition to using the logical name or the fully-qualified class name, Symfony supports a third way of referring to a controller. This method uses just one colon separator (e.g. `service_name:indexAction`) and refers to the controller as a service (see [_How to Define Controllers as Services_](https://symfony.com/doc/3.4/controller/service.html)).
```

## Loading Routes

Symfony loads all the routes for your application from a single routing configuration file: `app/config/routing.yml`. But from inside of this file, you can load any *other* routing files you want. In fact, by default, Symfony loads annotation route configuration from your AppBundle's `Controller/` directory, which is how Symfony sees our annotation routes:

```config {.yaml .xml .php}
~~~yaml
# app/config/routing.yml
app:
    resource: "@AppBundle/Controller/"
    type:     annotation
~~~

~~~xml
<!-- app/config/routing.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<routes xmlns="http://symfony.com/schema/routing"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/routing
        http://symfony.com/schema/routing/routing-1.0.xsd">

    <!-- the type is required to enable the annotation reader for this resource -->
    <import resource="@AppBundle/Controller/" type="annotation"/>
</routes>
~~~

~~~php
// app/config/routing.php
use Symfony\Component\Routing\RouteCollection;

$routes = new RouteCollection();
$routes->addCollection(
    // second argument is the type, which is required to enable
    // the annotation reader for this resource
    $loader->import("@AppBundle/Controller/", "annotation")
);

return $routes;
~~~
```

For more details on loading routes, including how to prefix the paths of loaded routes, see [_How to Include External Routing Resources_](https://symfony.com/doc/3.4/routing/external_resources.html).

## Generating URLs

The routing system should also be used to generate URLs. In reality, routing is a bidirectional system: mapping the URL to a controller and a route back to a URL.

To generate a URL, you need to specify the name of the route (e.g. `blog_show`) and any wildcards (e.g. `slug = my-blog-post`) used in the path for that route. With this information, any URL can easily be generated:

```php
class MainController extends Controller
{
    public function showAction($slug)
    {
        // ...

        // /blog/my-blog-post
        $url = $this->generateUrl(
            'blog_show',
            ['slug' => 'my-blog-post']
        );
    }
}
```

```note
The `generateUrl()` method defined in the base [Controller](https://github.com/symfony/symfony/blob/3.4/src/Symfony/Bundle/FrameworkBundle/Controller/Controller.php) class is just a shortcut for this code:

~~~php
$url = $this->container->get('router')->generate(
    'blog_show',
    ['slug' => 'my-blog-post']
);
~~~
```

## Generating URLs with Query Strings

The `generate()` method takes an array of wildcard values to generate the URI. But if you pass extra ones, they will be added to the URI as a query string:

```php
$this->get('router')->generate('blog', [
    'page' => 2,
    'category' => 'Symfony',
]);
// /blog/2?category=Symfony
```

## Generating URLs from a Template

To generate URLs inside Twig, see the templating article: [Linking to Pages](https://symfony.com/doc/3.4/templating.html#templating-pages). If you also need to generate URLs in JavaScript, see [_How to Generate Routing URLs in JavaScript_](https://symfony.com/doc/3.4/routing/generate_url_javascript.html).

## Generating Absolute URLs

By default, the router will generate relative URLs (e.g. `/blog`). From a controller, pass `UrlGeneratorInterface::ABSOLUTE_URL` to the third argument of the `generateUrl()` method:

```php
use Symfony\Component\Routing\Generator\UrlGeneratorInterface;

$this->generateUrl('blog_show', ['slug' => 'my-blog-post'], UrlGeneratorInterface::ABSOLUTE_URL);
// http://www.example.com/blog/my-blog-post
```

```note
The host that's used when generating an absolute URL is automatically detected using the current `Request` object. When generating absolute URLs from outside the web context (for instance in a console command) this doesn't work. See [_How to Generate URLs from the Console_](https://symfony.com/doc/3.4/console/request_context.html) to learn how to solve this problem.
```

## Troubleshooting

Here are some common errors you might see while working with routing:

> Controller "AppBundle\Controller\BlogController::showAction()" requires that you provide a value for the "$slug" argument.

This happens when your controller method has an argument (e.g. `$slug`):

```php
public function showAction($slug)
{
    // ..
}
```

But your route path does not have a `{slug}` wildcard (e.g. it is `/blog/show`). Add a `{slug}` to your route path: `/blog/show/{slug}` or give the argument a default value (i.e. `$slug = null`).

> Some mandatory parameters are missing ("slug") to generate a URL for route "blog_show".

This means that you're trying to generate a URL to the `blog_show` route but you are *not* passing a `slug` value (which is required, because it has a `{slug}`) wildcard in the route path. To fix this, pass a `slug` value when generating the route:

```php
$this->generateUrl('blog_show', ['slug' => 'slug-value']);

// or, in Twig
// {{ path('blog_show', {'slug': 'slug-value'}) }}
```

## Translating Routes

Symfony doesn't support defining routes with different contents depending on the user language. In those cases, you can define multiple routes per controller, one for each supported language; or use any of the bundles created by the community to implement this feature, such as [JMSI18nRoutingBundle](https://github.com/schmittjoh/JMSI18nRoutingBundle) and [BeSimpleI18nRoutingBundle](https://github.com/BeSimple/BeSimpleI18nRoutingBundle).

## Summary

Routing is a system for mapping the URL of incoming requests to the controller function that should be called to process the request. It both allows you to specify beautiful URLs and keeps the functionality of your application decoupled from those URLs. Routing is a bidirectional mechanism, meaning that it should also be used to generate URLs.

## Keep Going!

Routing, check! Now, uncover the power of [_controllers_](https://symfony.com/doc/3.4/controller.html).