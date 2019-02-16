建立一個新頁面，不論是是一個 HTML 頁面或是一個 JSON 的接口，都需要進行兩個步驟：

1. **建立 Route**：Route 是指你頁面網址，並指向一個 Controller
2. **建立 Controller**：Controller 是一個用來建構頁面的 PHP function，接收傳進來的 Request 資訊並產生一個 Symfony Response，Response 包含 HTML 內容、JSON 字串、或者甚至像圖片或 PDF 這樣的二進制檔案。

```see-also
Symfony *embraces* the HTTP Request-Response lifecycle. To find out more, see [_Symfony and HTTP Fundamentals_](https://symfony.com/doc/3.4/introduction/http_fundamentals.html).
```

## Creating a Page: Route and Controller

```tip
Before continuing, make sure you've read the [Setup](https://symfony.com/doc/3.4/setup.html) article and can access your new Symfony app in the browser.
```

Suppose you want to create a page - `/lucky/number` - that generates a lucky (well, random) number and prints it. To do that, create a "Controller class" and a "controller" method inside of it that will be executed when someone goes to `/lucky/number`:

```php
<?php
// src/AppBundle/Controller/LuckyController.php
namespace AppBundle\Controller;

use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;

class LuckyController
{
    /**
     * @Route("/lucky/number")
     */
    public function numberAction()
    {
        $number = random_int(0, 100);

        return new Response(
            '<html><body>Lucky number: '.$number.'</body></html>'
        );
    }
}
```

Before diving into this, test it out! If you are using PHP's internal web server go to:

> [http://localhost:8000/lucky/number](http://localhost:8000/lucky/number)

If you see a lucky number being printed back to you, congratulations! But before you run off to play the lottery, check out how this works. Remember the two steps to creating a page?

1. *Create a route*: The `@Route` above `numberAction()` is the route: it defines the URL pattern for this page. You'll learn more about [routing](/doc/symfony/3.4/routing.html) in its own section, including how to make *variable* URLs;
2. *Create a controller*: The method below the route - `numberAction()` - is called the *controller*. This is a function where *you* build the page and ultimately return a `Response` object. You'll learn more about [controllers](https://symfony.com/doc/3.4/controller.html) in their own section, including how to return JSON responses.

## The Web Debug Toolbar: Debugging Dream

If your page is working, then you should *also* see a bar along the bottom of your browser. This is called the Web Debug Toolbar: and it's your debugging best friend. You'll learn more about all the information it holds along the way, but feel free to experiment: hover over and click the different icons to get information about routing, performance, logging and more.

## Rendering a Template (with the Service Container)

If you're returning HTML from your controller, you'll probably want to render a template. Fortunately, Symfony comes with [Twig](https://twig.symfony.com/): a templating language that's easy, powerful and actually quite fun.

First, import the base [Controller](https://github.com/symfony/symfony/blob/3.4/src/Symfony/Bundle/FrameworkBundle/Controller/Controller.php) class as shown on line 5 below. Then, let your `LuckyController` class extend the base class:

```php
// src/AppBundle/Controller/LuckyController.php

// ...
// --> add this new use statement
use Symfony\Bundle\FrameworkBundle\Controller\Controller;

class LuckyController extends Controller
{
    // ...
}
```

Now, use the handy `render()` function to render a template. Pass it our `number` variable so we can render that:

```php
// src/AppBundle/Controller/LuckyController.php

// ...
class LuckyController extends Controller
{
    /**
     * @Route("/lucky/number")
     */
    public function numberAction()
    {
        $number = random_int(0, 100);

        return $this->render('lucky/number.html.twig', [
            'number' => $number,
        ]);
    }
}
```

Finally, template files should live in the `app/Resources/views` directory. Create a new `app/Resources/views/lucky` directory with a new `number.html.twig` file inside:

```php
{# app/Resources/views/lucky/number.html.twig #}

<h1>Your lucky number is {{ number }}</h1>
```

The `{{ number }}` syntax is used to *print* variables in Twig. Refresh your browser to get your *new* lucky number!

> [http://localhost:8000/lucky/number](http://localhost:8000/lucky/number)

In the [_Creating and Using Templates_](https://symfony.com/doc/3.4/templating.html) article, you'll learn all about Twig: how to loop, render other templates and leverage its powerful layout inheritance system.

## Checking out the Project Structure

Great news! You've already worked inside the two most important directories in your project:

- `app/`  
    Contains things like configuration and templates. Basically, anything that is *not* PHP code goes here.
- `src/`  
    Your PHP code lives here.  

    99% of the time, you'll be working in `src/` (PHP files) or `app/` (everything else). As you keep reading, you'll learn what can be done inside each of these.  

    So what about the other directories in the project?
- `bin/`  
    The famous `bin/console` file lives here (and other, less important executable files).
- `tests/`  
    The automated tests (e.g. Unit tests) for your application live here.
- `var/`  
    This is where automatically-created files are stored, like cache files (`var/cache/`), logs (`var/logs/`) and sessions (`var/sessions/`).
- `vendor/`  
    Third-party (i.e. "vendor") libraries live here! These are downloaded via the [Composer](https://getcomposer.org/) package manager.
- `web/`  
    This is the document root for your project: put any publicly accessible files here (e.g. CSS, JS and images).

## Bundles & Configuration

Your Symfony application comes pre-installed with a collection of *bundles*, like `FrameworkBundle` and `TwigBundle`. Bundles are similar to the idea of a *plugin*, but with one important difference: *all* functionality in a Symfony application comes from a bundle.

Bundles are registered in your `app/AppKernel.php` file (a rare PHP file in the `app/` directory) and each gives you more *tools*, sometimes called *services*:

```php
class AppKernel extends Kernel
{
    public function registerBundles()
    {
        $bundles = [
            new Symfony\Bundle\FrameworkBundle\FrameworkBundle(),
            new Symfony\Bundle\TwigBundle\TwigBundle(),
            // ...
        ];
        // ...

        return $bundles;
    }

    // ...
}
```

For example, `TwigBundle` is responsible for adding the Twig tool to your app!

Eventually, you'll download and add more third-party bundles to your app in order to get even more tools. Imagine a bundle that helps you create paginated lists. That exists!

You can control how your bundles behave via the `app/config/config.yml` file. That file - and other details like environments & parameters - are discussed in the [_Configuring Symfony (and Environments)_](https://symfony.com/doc/3.4/configuration.html) article.

## What's Next?

Congrats! You're already starting to master Symfony and learn a whole new way of building beautiful, functional, fast and maintainable apps.

Ok, time to finish mastering the fundamentals by reading these articles:

- [Routing](https://symfony.com/doc/3.4/routing.html)
- [Controller](https://symfony.com/doc/3.4/controller.html)
- [Creating and Using Templates](https://symfony.com/doc/3.4/templating.html)
- [Configuring Symfony (and Environments)](https://symfony.com/doc/3.4/configuration.html)

Then, learn about other important topics like the [_service container_](https://symfony.com/doc/3.4/service_container.html), the [_form system_](https://symfony.com/doc/3.4/forms.html), using [_Doctrine_](https://symfony.com/doc/3.4/doctrine.html) (if you need to query a database) and more!

Have fun!