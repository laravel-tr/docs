# Cepheler (Facades)

- [Giriş](#introduction)
- [Açıklama](#explanation)
- [Pratik Kullanım](#practical-usage)
- [Cephe Oluşturma](#creating-facades)
- [Cepheleri Taklit Etme](#mocking-facades)
- [Facade Class Reference](#facade-class-reference)

<a name="introduction"></a>
## Giriş

Cepheler uygulamanızın [IoC konteynerinde](/docs/ioc) bulunan sınıflar için "statik" bir arayüz sağlar. Laravel birçok cephe ile gelmektedir ve büyük bir ihtimalle daha ne olduklarını bilmeden onları kullanıyorsunuzdur!

Zaman zaman, uygulama ve paketleriniz için kendi cephelerinizi oluşturmak isteyebilirsiniz, bu itibarla bu sınıfların kavramlarını, geliştirilmesini ve kullanımını inceleyelim.

> **Not:** Cepheler konusunu incelemeden önce Laravel [IoC konteyneri](/docs/ioc) ile çok aşina olmanız kuvvetle önerilir.

<a name="explanation"></a>
## Açıklama

Bir Laravel uygulaması bağlamında bir cephe bir nesneye onun konteynerinden erişim sağlayan bir sınıf demektir. Bu işi yapan mekanizmalar `Facade` sınıfında tanımlıdır. Laravel'in cepheleri ve sizin oluşturduğunuz kendi cepheleriniz bu temel `Facade` sınıfından türeyecektir.

Sizin cephe sınıfınız sadece tek bir metoda tatbikat getirmesi gerekiyor: `getFacadeAccessor`. `getFacadeAccessor` methodunun tanımlayacağı iş konteynerden ne çözeceğidir. `Facade` temel sınıfı sizin cephelerinizden, çözülmüş nesneye yapılan çağrıları ertelemek için `__callStatic()` sihirli-metodunu kullanır.

So, when you make a facade call like `Cache::get`, Laravel resolves the Cache manager class out of the IoC container and calls the `get` method on the class. In technical terms, Laravel Facades are a convenient syntax for using the Laravel IoC container as a service locator.

<a name="practical-usage"></a>
## Pratik Kullanım

Aşağıdaki örnekte, Laravel önbellekleme sistemine bir çağrı yapılmış. Bu koda göz attığınızda, `Cache` sınıfında statik bir metod olan `get`'in çağrılıyor olduğunu düşünebilirsiniz.

	$deger = Cache::get('anahtar');

Ancak, eğer `Illuminate\Support\Facades\Cache` sınıfına bakacak olursak, orada `get` adında statik bir metod olmadığını görürüz:

	class Cache extends Facade {

		/**
		 * Get the registered name of the component.
		 *
		 * @return string
		 */
		protected static function getFacadeAccessor() { return 'cache'; }

	}

Bu Cache sınıfı temel `Facade` sınıfından türetilmiş ve `getFacadeAccessor()` adında bir metod tanımlamış. Bu metodun işinin bir IoC bağlayıcısının adını döndürmek olduğunu hatırlayın.

Bir kullanıcı `Cache` cephesinde herhangi bir statik metoda başvurduğunda, Laravel, IoC konteynerinden `cache` bağlayıcısını çözecek ve istenen metodu (bu örnekte `get`) bu nesneye karşı çalıştıracaktır.

Yani bizim `Cache::get` çağrımız şu şekilde yeniden yazılabilir:

	$value = $app->make('cache')->get('anahtar');

<a name="creating-facades"></a>
## Cephe Oluşturma

Kendi uygulama veya paketiniz için bir cephe oluşturulması kolaydır. Sadece üç şeye ihtiyacınız vardır:

- Bir IoC bağlayıcısı
- Bir cephe sınıfı.
- Bir cephe takma adı yapılandırması.

Bir örnek bakalım. Burada, `OdemeGecidi\Odeme` olarak tanımlanmış bir sınıfımız var.

	namespace OdemeGecidi;

	class Odeme {

		public function process()
		{
			//
		}

	}

This class might live in your `app/models` directory, or any other directory that Composer knows how to auto-load.

Bu sınıfı IoC konteynerinden çözebiliyor olmamız lazım. Öyleyse, bir bağlayıcı ekleyelim:

	App::bind('odeme', function()
	{
		return new \OdemeGecidi\Odeme;
	});

Bu bağlayıcıyı kayda geçirmek için harika bir yer `OdemeServiceProvider` adında yeni bir [hizmet sağlayıcı](/docs/ioc#service-providers) oluşturmak ve bu bağlayıcıyı `register` metoduna eklemek olacaktır. Daha sonra Laravel'i sizin hizmet sağlayıcınızı `app/config/app.php` yapılandırma dosyasından yükleyecek şekilde yapılandırın.

Daha sonra, kendi cephe sınıfımızı oluşturabiliriz:

	use Illuminate\Support\Facades\Facade;

	class Odeme extends Facade {

		protected static function getFacadeAccessor() { return 'odeme'; }

	}

Son olarak, eğer istiyorsak, `app/config/app.php` yapılandırma dosyasındaki `aliases` dizisine kendi cephe'miz için bir takma ad ekleyebiliriz. Artık, `process` metodunu `Odeme` sınıfının bir olgusunda çağırabiliriz.

	Odeme::process();

### Alias (Takma Ad)'ların Otomatik Yüklenmesi Üzerine Bir Bilgi Notu

[PHP, tanımlanmamış tip dayatmalı sınıfları otomatik olarak yüklemeye çalışmayacağı için](https://bugs.php.net/bug.php?id=39003) `app\config\app.php` dosyasının `aliases` dizisindeki sınıflar bazı durumlarda kullanılamamaktadır. `\ServiceWrapper\ApiTimeoutException` sınıfı eğer `ApiTimeoutException` olarak aliaslanmış ise, namespace `\ServiceWrapper` aduzayının dışındaki bir `catch(ApiTimeoutException $e)` ifadesi bu istisnayı hiçbir zaman, hatta istisna çıkması durumunda bile yakalamayacaktır. Benzer bir durum, aliaslanmış sınıflara tip dayatması olan Models'de de bulunmuştur. Tek geçici çözüm, aliaslamaktan vazgeçmek ve tip dayatması yapmak istediğiniz sınıfları bunları gerektiren her dosyanın en başında `use` etmektir.

<a name="mocking-facades"></a>
## Cepheleri Taklit Etme

Ünite testi cephelerin nasıl çalıştıkları konusunda önemli bir husustur. Gerçekten, cephelerin varlıkları için bile ilk neden test edilebilirliktir. Daha fazla bilgi için, belgelerdeki [Cepheleri Taklit Etme](/docs/testing#mocking-facades) kesimine bakın.

<a name="facade-class-reference"></a>
## Facade Class Reference

Below you will find every facade and its underlying class. This is a useful tool for quickly digging into the API documentation for a given facade root. The [IoC binding](/docs/ioc) key is also included where applicable.

Facade  |  Class  |  IoC Binding
------------- | ------------- | -------------
App  |  [Illuminate\Foundation\Application](http://laravel.com/api/4.1/Illuminate/Foundation/Application.html)  | `app`
Artisan  |  [Illuminate\Console\Application](http://laravel.com/api/4.1/Illuminate/Console/Application.html)  |  `artisan`
Auth  |  [Illuminate\Auth\AuthManager](http://laravel.com/api/4.1/Illuminate/Auth/AuthManager.html)  |  `auth`
Auth (Instance)  |  [Illuminate\Auth\Guard](http://laravel.com/api/4.1/Illuminate/Auth/Guard.html)  |
Blade  |  [Illuminate\View\Compilers\BladeCompiler](http://laravel.com/api/4.1/Illuminate/View/Compilers/BladeCompiler.html)  |  `blade.compiler`
Cache  |  [Illuminate\Cache\Repository](http://laravel.com/api/4.1/Illuminate/Cache/Repository.html)  |  `cache`
Config  |  [Illuminate\Config\Repository](http://laravel.com/api/4.1/Illuminate/Config/Repository.html)  |  `config`
Cookie  |  [Illuminate\Cookie\CookieJar](http://laravel.com/api/4.1/Illuminate/Cookie/CookieJar.html)  |  `cookie`
Crypt  |  [Illuminate\Encryption\Encrypter](http://laravel.com/api/4.1/Illuminate/Encryption/Encrypter.html)  |  `encrypter`
DB  |  [Illuminate\Database\DatabaseManager](http://laravel.com/api/4.1/Illuminate/Database/DatabaseManager.html)  |  `db`
DB (Instance)  |  [Illuminate\Database\Connection](http://laravel.com/api/4.1/Illuminate/Database/Connection.html)  |
Event  |  [Illuminate\Events\Dispatcher](http://laravel.com/api/4.1/Illuminate/Events/Dispatcher.html)  |  `events`
File  |  [Illuminate\Filesystem\Filesystem](http://laravel.com/api/4.1/Illuminate/Filesystem/Filesystem.html)  |  `files`
Form  |  [Illuminate\Html\FormBuilder](http://laravel.com/api/4.1/Illuminate/Html/FormBuilder.html)  |  `form`
Hash  |  [Illuminate\Hashing\HasherInterface](http://laravel.com/api/4.1/Illuminate/Hashing/HasherInterface.html)  |  `hash`
HTML  |  [Illuminate\Html\HtmlBuilder](http://laravel.com/api/4.1/Illuminate/Html/HtmlBuilder.html)  |  `html`
Input  |  [Illuminate\Http\Request](http://laravel.com/api/4.1/Illuminate/Http/Request.html)  |  `request`
Lang  |  [Illuminate\Translation\Translator](http://laravel.com/api/4.1/Illuminate/Translation/Translator.html)  |  `translator`
Log  |  [Illuminate\Log\Writer](http://laravel.com/api/4.1/Illuminate/Log/Writer.html)  |  `log`
Mail  |  [Illuminate\Mail\Mailer](http://laravel.com/api/4.1/Illuminate/Mail/Mailer.html)  |  `mailer`
Paginator  |  [Illuminate\Pagination\Factory](http://laravel.com/api/4.1/Illuminate/Pagination/Factory.html)  |  `paginator`
Paginator (Instance)  |  [Illuminate\Pagination\Paginator](http://laravel.com/api/4.1/Illuminate/Pagination/Paginator.html)  |
Password  |  [Illuminate\Auth\Reminders\PasswordBroker](http://laravel.com/api/4.1/Illuminate/Auth/Reminders/PasswordBroker.html)  |  `auth.reminder`
Queue  |  [Illuminate\Queue\QueueManager](http://laravel.com/api/4.1/Illuminate/Queue/QueueManager.html)  |  `queue`
Queue (Instance) |  [Illuminate\Queue\QueueInterface](http://laravel.com/api/4.1/Illuminate/Queue/QueueInterface.html)  |
Queue (Base Class) |  [Illuminate\Queue\Queue](http://laravel.com/api/4.1/Illuminate/Queue/Queue.html)  |
Redirect  |  [Illuminate\Routing\Redirector](http://laravel.com/api/4.1/Illuminate/Routing/Redirector.html)  |  `redirect`
Redis  |  [Illuminate\Redis\Database](http://laravel.com/api/4.1/Illuminate/Redis/Database.html)  |  `redis`
Request  |  [Illuminate\Http\Request](http://laravel.com/api/4.1/Illuminate/Http/Request.html)  |  `request`
Response  |  [Illuminate\Support\Facades\Response](http://laravel.com/api/4.1/Illuminate/Support/Facades/Response.html)  |
Route  |  [Illuminate\Routing\Router](http://laravel.com/api/4.1/Illuminate/Routing/Router.html)  |  `router`
Schema  |  [Illuminate\Database\Schema\Blueprint](http://laravel.com/api/4.1/Illuminate/Database/Schema/Blueprint.html)  |
Session  |  [Illuminate\Session\SessionManager](http://laravel.com/api/4.1/Illuminate/Session/SessionManager.html)  |  `session`
Session (Instance)  |  [Illuminate\Session\Store](http://laravel.com/api/4.1/Illuminate/Session/Store.html)  |
SSH  |  [Illuminate\Remote\RemoteManager](http://laravel.com/api/4.1/Illuminate/Remote/RemoteManager.html)  |  `remote`
SSH (Instance)  |  [Illuminate\Remote\Connection](http://laravel.com/api/4.1/Illuminate/Remote/Connection.html)  |
URL  |  [Illuminate\Routing\UrlGenerator](http://laravel.com/api/4.1/Illuminate/Routing/UrlGenerator.html)  |  `url`
Validator  |  [Illuminate\Validation\Factory](http://laravel.com/api/4.1/Illuminate/Validation/Factory.html)  |  `validator`
Validator (Instance)  |  [Illuminate\Validation\Validator](http://laravel.com/api/4.1/Illuminate/Validation/Validator.html)
View  |  [Illuminate\View\Factory](http://laravel.com/api/4.1/Illuminate/View/Factory.html)  |  `view`
View (Instance)  |  [Illuminate\View\View](http://laravel.com/api/4.1/Illuminate/View/View.html)  |