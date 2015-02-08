# Contracts

- [Giriş](#introduction)
- [Neden sözleşmeleri kullanıyoruz?](#why-contracts)
- [Contract Reference](#contract-reference)
- [How To Use Contracts](#how-to-use-contracts)

<a name="introduction"></a>
## Giriş

Laravel'in Sözleşmeleri (Contracts) Laravel'in çekirdeğindeki komponentleri açıklayan interfacelerdir. Örneğin, bir `Queue` sözleşmesi, Laravel'in kuyruk sistemindeki gerekli methodları belirtir. Aynı şekilde, `Mailer` sözleşmesi de e-mail gönderimi için gerekli olan methodları belirtir.

Her sözleşmenin Laravel çekirdeğinde bir implementasyonu bulunmaktadır. Örneğin, Laravel `Queue` (Kuyruk) fonksiyonalitesini, `Queue` sözleşmesine sadık kalan birçok driver implementasyonu üzerinden gerçekleştirirken, `Mailer` fonksiyonalitesini ise [SwiftMailer](http://swiftmailer.org/) paketi üzerinden gerçekleştirir.

Tüm Laravel sözleşmeleri [kendi Github ambarlarında](https://github.com/illuminate/contracts) bulunmaktadır. Böylece tüm sözleşmeler için sabit bir referans noktası sağlanmış olup, paket geliştiricilerin kendi projelerinde kullanabileceği bağımsız bir komponent ortaya çıkmıştır.

<a name="why-contracts"></a>
## Neden sözleşmeleri kullanıyoruz?

Sözleşmeler hakkında bazı soru işaretleri bulunabilir. Neden interfaceleri kullanıyoruz? Interfaceleri kullanmak uygulamayı daha da komplike bir hale getirmiyor mu?

Sözleşme kullanmaktaki amaçlarımızı 2 başlık altında inceleyelim: bağımsızlık ve basitlik.

### Bağımsızlık

Öncelikle, Cache implementasyonuna sıkıca bağlanmış bir kod örneği görelim.

	<?php namespace App\Orders;

	class Repository {

		/**
		 * The cache.
		 */
		protected $cache;

		/**
		 * Create a new repository instance.
		 *
		 * @param  \Package\Cache\Memcached  $cache
		 * @return void
		 */
		public function __construct(\SomePackage\Cache\Memcached $cache)
		{
			$this->cache = $cache;
		}

		/**
		 * Retrieve an Order by ID.
		 *
		 * @param  int  $id
		 * @return Order
		 */
		public function find($id)
		{
			if ($this->cache->has($id))
			{
				//
			}
		}

	}

Bu sınıf, cache sınıfına sıkıca bağlanmıştır. Sıkıca bağlanmıştır çünkü bir cache paketini sınıfımız üzerinde direkt olarak kullanmaktayız. Eğer bu paketin API'i değişirse, bizim de kodlarımızı değiştirmemiz gerekecektir.

Aynı şekilde, eğer halihazırda kullandığımız bir (Memcached) özelliğini farklı bir teknoloji ile (Redis gibi) kullanmaya kalkarsak, yine kodlarımızı değiştirmemiz gerekecektir. Bizim sınıfımız önbellekleme işleminin nasıl yapılması gerektiği konusunda bilgi sahibi olmak zorunda değildir.

**Belirli bir sınıfa bağlı kalmak yerine, bir sözleşmeye (interfaceye) bağlı kalan herhangi bir implementasyonu kullanabilmeliyiz:**

	<?php namespace App\Orders;

	use Illuminate\Contracts\Cache\Repository as Cache;

	class Repository {

		/**
		 * Create a new repository instance.
		 *
		 * @param  Cache  $cache
		 * @return void
		 */
		public function __construct(Cache $cache)
		{
			$this->cache = $cache;
		}

	}

Artık yazdığımız kod herhangi bir pakete bağlı değildir, Laravel'e bile. Contracts komponenti herhangi bir bağımlılığa sahip olmadığı için, siz kolayca her sözleşme için alternatif bir implementasyon oluşturabilirsiniz.

### Basitlik

When all of Laravel's services are neatly defined within simple interfaces, it is very easy to determine the functionality offered by a given service. **The contracts serve as succinct documentation to the framework's features.**

In addition, when you depend on simple interfaces, your code is easier to understand and maintain. Rather than tracking down which methods are available to you within a large, complicated class, you can refer to a simple, clean interface.

<a name="contract-reference"></a>
## Contract Reference

This is a reference to most Laravel Contracts, as well as their Laravel "facade" counterparts:

Contract  |  Laravel 4.x Facade
------------- | -------------
[Illuminate\Contracts\Auth\Guard](https://github.com/illuminate/contracts/blob/master/Auth/Guard.php)  |  Auth
[Illuminate\Contracts\Auth\PasswordBroker](https://github.com/illuminate/contracts/blob/master/Auth/PasswordBroker.php)  |  Password
[Illuminate\Contracts\Cache\Repository](https://github.com/illuminate/contracts/blob/master/Cache/Repository.php) | Cache
[Illuminate\Contracts\Cache\Factory](https://github.com/illuminate/contracts/blob/master/Cache/Factory.php) | Cache::driver()
[Illuminate\Contracts\Config\Repository](https://github.com/illuminate/contracts/blob/master/Config/Repository.php) | Config
[Illuminate\Contracts\Container\Container](https://github.com/illuminate/contracts/blob/master/Container/Container.php) | App
[Illuminate\Contracts\Cookie\Factory](https://github.com/illuminate/contracts/blob/master/Cookie/Factory.php) | Cookie
[Illuminate\Contracts\Cookie\QueueingFactory](https://github.com/illuminate/contracts/blob/master/Cookie/QueueingFactory.php) | Cookie::queue()
[Illuminate\Contracts\Encryption\Encrypter](https://github.com/illuminate/contracts/blob/master/Encryption/Encrypter.php) | Crypt
[Illuminate\Contracts\Events\Dispatcher](https://github.com/illuminate/contracts/blob/master/Events/Dispatcher.php) | Event
[Illuminate\Contracts\Filesystem\Cloud](https://github.com/illuminate/contracts/blob/master/Filesystem/Cloud.php) | &nbsp;
[Illuminate\Contracts\Filesystem\Factory](https://github.com/illuminate/contracts/blob/master/Filesystem/Factory.php) | File
[Illuminate\Contracts\Filesystem\Filesystem](https://github.com/illuminate/contracts/blob/master/Filesystem/Filesystem.php) | File
[Illuminate\Contracts\Foundation\Application](https://github.com/illuminate/contracts/blob/master/Foundation/Application.php) | App
[Illuminate\Contracts\Hashing\Hasher](https://github.com/illuminate/contracts/blob/master/Hashing/Hasher.php) | Hash
[Illuminate\Contracts\Logging\Log](https://github.com/illuminate/contracts/blob/master/Logging/Log.php) | Log
[Illuminate\Contracts\Mail\MailQueue](https://github.com/illuminate/contracts/blob/master/Mail/MailQueue.php) | Mail::queue()
[Illuminate\Contracts\Mail\Mailer](https://github.com/illuminate/contracts/blob/master/Mail/Mailer.php) | Mail
[Illuminate\Contracts\Queue\Factory](https://github.com/illuminate/contracts/blob/master/Queue/Factory.php) | Queue::driver()
[Illuminate\Contracts\Queue\Queue](https://github.com/illuminate/contracts/blob/master/Queue/Queue.php) | Queue
[Illuminate\Contracts\Redis\Database](https://github.com/illuminate/contracts/blob/master/Redis/Database.php) | Redis
[Illuminate\Contracts\Routing\Registrar](https://github.com/illuminate/contracts/blob/master/Routing/Registrar.php) | Route
[Illuminate\Contracts\Routing\ResponseFactory](https://github.com/illuminate/contracts/blob/master/Routing/ResponseFactory.php) | Response
[Illuminate\Contracts\Routing\UrlGenerator](https://github.com/illuminate/contracts/blob/master/Routing/UrlGenerator.php) | URL
[Illuminate\Contracts\Support\Arrayable](https://github.com/illuminate/contracts/blob/master/Support/Arrayable.php) | &nbsp;
[Illuminate\Contracts\Support\Jsonable](https://github.com/illuminate/contracts/blob/master/Support/Jsonable.php) | &nbsp;
[Illuminate\Contracts\Support\Renderable](https://github.com/illuminate/contracts/blob/master/Support/Renderable.php) | &nbsp;
[Illuminate\Contracts\Validation\Factory](https://github.com/illuminate/contracts/blob/master/Validation/Factory.php) | Validator::make()
[Illuminate\Contracts\Validation\Validator](https://github.com/illuminate/contracts/blob/master/Validation/Validator.php) | &nbsp;
[Illuminate\Contracts\View\Factory](https://github.com/illuminate/contracts/blob/master/View/Factory.php) | View::make()
[Illuminate\Contracts\View\View](https://github.com/illuminate/contracts/blob/master/View/View.php) | &nbsp;

<a name="how-to-use-contracts"></a>
## How To Use Contracts

So, how do you get an implementation of a contract? It's actually quite simple. Many types of classes in Laravel are resolved through the [service container](/docs/master/container), including controllers, event listeners, filters, queue jobs, and even route Closures. So, to get an implementation of a contract, you can just "type-hint" the interface in the constructor of the class being resolved. For example, take a look at this event handler:

	<?php namespace App\Handlers\Events;

	use App\User;
	use App\Events\NewUserRegistered;
	use Illuminate\Contracts\Redis\Database;

	class CacheUserInformation {

		/**
		 * The Redis database implementation.
		 */
		protected $redis;

		/**
		 * Create a new event handler instance.
		 *
		 * @param  Database  $redis
		 * @return void
		 */
		public function __construct(Database $redis)
		{
			$this->redis = $redis;
		}

		/**
		 * Handle the event.
		 *
		 * @param  NewUserRegistered  $event
		 * @return void
		 */
		public function handle(NewUserRegistered $event)
		{
			//
		}

	}

When the event listener is resolved, the service container will read the type-hints on the constructor of the class, and inject the appropriate value. To learn more about registering things in the service container, check out [the documentation](/docs/master/container).
