# İstek Yaşam Döngüsü

- [Genel Bakış](#overview)
- [Request Lifecycle](#request-lifecycle)
- [Start Dosyaları](#start-files)
- [Application Olayları (Events)](#application-events)

<a name="overview"></a>
## Genel Bakış

When using any tool in the "real world", you feel more confidence if you understand how that tool works. Application development is no different. When you understand how your development tools function, you feel more comfortable and confident using them. The goal of this document is to give you a good, high-level overview of how the Laravel framework "works". By getting to know the overall framework better, everything feels less "magical" and you will be more confident building your applications. In addition to a high-level overview of the request lifecycle, we'll cover "start" files and application events.

If you don't understand all of the terms right away, don't lose heart! Just try to get a basic grasp of what is going on, and your knowledge will grow as you explore other sections of the documentation.

<a name="request-lifecycle"></a>
## Request Lifecycle

All requests into your application are directed through the `public/index.php` script. When using Apache, the `.htaccess` file that ships with Laravel handles the passing of all requests to `index.php`. From here, Laravel begins the process of handling the requests and returning a response to the client. Getting a general idea for the Laravel bootstrap process will be useful, so we'll cover that now!

By far, the most important concept to grasp when learning about Laravel's bootstrap process is **Service Providers**. You can find a list of service providers by opening your `app/config/app.php` configuration file and finding the `providers` array. These providers serve as the primary bootstrapping mechanism for Laravel. But, before we dig into service providers, let's go back to `index.php`. After a request enters your `index.php` file, the `bootstrap/start.php` file will be loaded. This file creates the new Laravel `Application` object, which also serves as an [IoC container](/docs/ioc).

After creating the `Application` object, a few project paths will be set and [environment detection](/docs/configuration#environment-configuration) will be performed. Then, an internal Laravel bootstrap script will be called. This file lives deep within the Laravel source, and sets a few more settings based on your configuration files, such as timezone, error reporting, etc. But, in addition to setting these rather trivial configuration options, it also does something very important: registers all of the service providers configured for your application.

Simple service providers only have one method: `register`. This `register` method is called when the service provider is registered with the application object via the application's own `register` method. Within this method, service providers register things with the [IoC container](/docs/ioc). Essentially, each service provider binds one or more [closures](http://us3.php.net/manual/en/functions.anonymous.php) into the container, which allows you to access those bound services within your application. So, for example, the `QueueServiceProvider` registers closures that resolve the various [Queue](/docs/queues) related classes. Of course, service providers may be used for any bootstrapping task, not just registering things with the IoC container. A service provider may register event listeners, view composers, Artisan commands, and more.

After all of the service providers have been registered, your `app/start` files will be loaded. Lastly, your `app/routes.php` file will be loaded. Once your `routes.php` file has been loaded, the Request object is sent to the application so that it may be dispatched to a route.

So, let's summarize:

1. Request enters `public/index.php` file.
2. `bootstrap/start.php` file creates Application and detects environment.
3. Internal `framework/start.php` file configures settings and loads service providers.
4. Application `app/start` files are loaded.
5. Application `app/routes.php` file is loaded.
6. Request object sent to Application, which returns Response object.
7. Response object sent back to client.

Now that you have a good idea of how a request to a Laravel application is handled, let's take a closer look at "start" files!

<a name="start-files"></a>
## Start Dosyaları

Uygulamanızın start dosyaları `app/start` dizininde bulunmaktadır. Varsayılan olarak bunlardan üçü uygulamanızın içine dahil edilmiştir. Bunlar `global.php`, `local.php`, ve `artisan.php`'dir. `artisan.php` hakkında daha fazla bilgiye sahip olmak için [Artisan komut satırı](/docs/commands#registering-commands) dökümanlarına bakınız.

Bunlardan `global.php` start dosyası [Günlüklerin](/docs/errors) kayda geçirilmesi ve `app/filters.php` dosyanızın dahil edilmesi gibi ön tanımlı birkaç temel öğe içerir. Ancak, bu dosyaya istediğiniz her şeyi ekleyebilirsiniz. Bu dosya ortam ne olursa olsun uygulamanıza gelen _her_ istekte otomatik olarak dahil edilecektir. Öte yandan `local.php` dosyası yalnızca uygulamanız `local` ortamda çalışırken çağrılır. Ortamlar hakkında daha fazla bilgi için [Yapılandırma](/docs/configuration) belgelerine bakınız.

`local`'e ilaveten başka ortamlarınız da varsa, pek tabii bu ortamlar için de start dosyaları oluşturabilirsiniz. Uygulamanız o ortamda çalıştığı zaman bunlar otomatik olarak dahil edileceklerdir. So, for example, if you have a `development` environment configured in your `bootstrap/start.php` file, you may create a `app/start/development.php` file, which will be included when any requests enter the application in that environment.

### What To Place In Start Files

Start files serve as a simple place to place any "bootstrapping" code. For example, you could register a View composer, configure your logging preferences, set some PHP settings, etc. It's totally up to you. Of course, throwing all of your bootstrapping code into your start files can get messy. For large applications, or if you feel your start files are getting messy, consider moving some bootstrapping code into [service providers](/docs/ioc#service-providers).

<a name="application-events"></a>
## Application Olayları (Events)

Bunlara ek olarak `before`, `after`, `close`, `finish` ve `shutdown` uygulama olaylarını kayda geçirmek suretiyle istek öncesinde ve sonrasında bazı işlemler de yapabilirsiniz:

#### Uygulama Olaylarının Kayda Geçirilmesi

	App::before(function($request)
	{
		//İstek öncesi olayları
	});

	App::after(function($request, $response)
	{
		//İstek sonrası olayları
	});

Bu olay dinleyicileri, uygulamanıza yapılan her istek öncesinde `(before)` ve sonrasında `(after)` çalışacaktır. These events can be helpful for global filtering or global modification of responses. You may register them in one of your `start` files or in a [service provider](/docs/ioc#service-providers).

You may also register a listener on the `matched` event, which is fired when an incoming request has been matched to a route but that route has not yet been executed:

	Route::matched(function($route, $request)
	{
		//
	});

The `finish` event is called after the response from your application has been sent back to the client. This is a good place to do any last minute processing your application requires. The `shutdown` event is called immediately after all of the `finish` event handlers finish processing, and is the last opportunity to do any work before the script terminates. Most likely, you will not have a need to use either of these events.