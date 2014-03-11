# Frameworkün Genişletilmesi

- [Giriş](#introduction)
- [Manager'lar & Factory'ler](#managers-and-factories)
- [Cache](#cache)
- [Session](#session)
- [Authentication](#authentication)
- [IoC Temelli Genişletme](#ioc-based-extension)
- [Request Genişletmesi](#request-extension)

<a name="introduction"></a>
## Giriş

Laravel, frameworkün çekirdek bileşenlerinin davranışlarını isteğinize göre özelleştirebilmeniz, hatta tümden değiştirebilmeniz için size birçok genişletme noktası sağlar. Örneğin, hash yapma araçları bir `HasherInterface` sözleşmesi ile sağlanmış olup, kendi uygulamanızın gereksinimlerine dayalı olarak implemente edebilirsiniz. Ayrıca, sizin kendi uygun "helper" metodlarınızı eklemenize imkan vermek üzere `Request` nesnesini de genişletebilirsiniz. Hatta tamamen yeni kimlik doğrulama, cache ve session sürücüleri bile ekleyebilirsiniz!

Laravel bileşenleri genel olarak iki şekilde genişletilir: IoC konteynerinde yeni implementasyonlar bağlayarak veya bir ekstensiyonu "Factory" tasarım deseninin implementasyonları olan bir `Manager` sınıfı ile register ederek. Bu bölümde, frameworkün genişletilmesi için çeşitli yöntemleri keşfedeceğiz ve gerekli kodları inceleyeceğiz.

> **Not:** Aklınızda tutun, Laravel bileşenleri tipik olarak iki yoldan biriyle genişletilir: IoC bağlamaları ve `Manager` sınıfları. Manager sınıfları "factory" tasarım deseninin bir implementasyonu olarak hizmet eder ve cache ve session gibi sürücü temelli araçların başlatılmasından sorumludur.

<a name="managers-and-factories"></a>
## Manager'lar & Factory'ler

Laravel sürücü temelli bileşenlerin oluşturulmasını yöneten birkaç `Manager` sınıfıyla gelir. Bunlar cache, session, authentication ve queue bileşenleridir. Manager sınıfı, uygulamanın yapılandırmasına dayalı olarak belirli bir sürücü implementasyonunun oluşturulmasından sorumludur. Örneğin, `CacheManager` sınıfı APC, Memcached, Native ve diğer cache sürücü implementasyonlarını oluşturabilir.

Bu managerlerin her birisinde, managere kolaylıkla yeni sürücü çözünürlük işlevselliği enjekte edilmesi için kullanılabilen bir `extend` metodu bulunmaktadır. Aşağıda, bu managerlerin her birini, onların her birine özel bir sürücü desteğinin nasıl enjekte edildiğinin örnekleriyle birlikte göreceğiz.

> **Not:** Laravel'le gelen `CacheManager` ve `SessionManager` gibi çeşitli `Manager` sınıflarını keşfetmek için biraz zaman ayırın. Bu sınıfların baştan sona okunması Laravel'in örtü altında nasıl çalıştığı konusunda size daha kapsamlı bir anlayış verecektir. Tüm manager sınıfları `Illuminate\Support\Manager` taban sınıfını genişletir, bu taban sınıf her manager için yararlı, ortak bazı işlevsellik sağlar.

<a name="cache"></a>
## Cache

Laravel cache aracını genişletmek için, `CacheManager`deki managera özel bir sürücü çözümleyicisi bağlamak için kullanılan ve tüm manager sınıfları çapında ortak olan `extend` metodunu kullanacağız. Örneğin, "mongo" adında yeni bir cache sürücüsü register etmek için, şöyle yapacağız:

	Cache::extend('mongo', function($app)
	{
		// Illuminate\Cache\Repository olgusu döndür...
	});

`extend` metoduna geçilen ilk parametre sürücünün adıdır. Bu, sizin `app/config/cache.php` yapılandırma dosyanızdaki `driver` opsiyonuna tekabül edecektir. İkinci parametre bir `Illuminate\Cache\Repository` olgusu döndürmesi gereken bir Closure'dur. Bu Closure'a `Illuminate\Foundation\Application`in bir olgusu ve bir IoC konteyneri olan bir `$app` olgusu geçilecektir.

Özel cache sürücümüzü oluşturmak için, öncelikle `Illuminate\Cache\StoreInterface` sözleşmesini implemente etmemiz gerekiyor. Yani, bizim MongoDB cache implementasyonumuz şöyle bir şey olacaktır:

	class MongoStore implements Illuminate\Cache\StoreInterface {

		public function get($key) {}
		public function put($key, $value, $minutes) {}
		public function increment($key, $value = 1) {}
		public function decrement($key, $value = 1) {}
		public function forever($key, $value) {}
		public function forget($key) {}
		public function flush() {}

	}

Sadece bir MongoDB bağlantısı kullanarak bu metodların her birini implemente etmemiz gerekiyor. Implementasyonumuzu tamamladıktan sonra, özel sürücümüzün kaydını bitirebiliriz:

	use Illuminate\Cache\Repository;

	Cache::extend('mongo', function($app)
	{
		return new Repository(new MongoStore);
	});

Yukarıdaki örnekte görebileceğiniz gibi, özel cache sürücüleri oluştururken taban `Illuminate\Cache\Repository` sınıfını kullanabilirsiniz. Tipik olarak kendi repository sınıfınızı oluşturma zorunluğu söz konusu değildir.

Özel cache sürücü kodunuzu nereye koyacağınızı merak ediyorsanız, onu Packagist'te bulundurmayı düşünün! Veya, uygulamanızın birincil klasörü içinde bir `Extensions` aduzayı oluşturabilirsiniz. Örneğin, uygulama `Snappy` adındaysa, cache ekstensiyonunu `app/Snappy/Extensions/MongoStore.php` içine koyabilirsiniz. Bununla birlikte, Laravel'in katı bir uygulama yapısına sahip olmadığını ve uygulamanızı sizin tercihlerinize göre organize etmekte özgür olduğunuzu aklınızda tutun.

> **Not:** Şayet kod parçalarınızı nereye koyacağınızı merak ediyorsanız, her zaman bir hizmet sağlayıcı düşünün. Daha önce tartıştığımız gibi, framework ekstensiyonlarını organize etmek için bir hizmet sağlayıcı kullanmak kodunuzu organize etmek için harika bir yoldur.

<a name="session"></a>
## Session

Laravel'i özel bir session sürücüsü ile genişletmek, tıpkı cache sisteminin genişletilmesi kadar kolaydır. Aynı şekilde, özel kodumuzu register etmek için `extend` metodunu kullanacağız:

	Session::extend('mongo', function($app)
	{
		// SessionHandlerInterface'in implementasyonunu döndür
	});

Dikkat ederseniz bizim özel session sürücümüz `SessionHandlerInterface`i implemente edecektir. Bu interface PHP 5.4+ çekirdeğine dahil edilmiştir. Eğer siz PHP 5.3 kullanıyorsanız, ileriye yönelik uyumluluğa sahip olmanız için bu interface Laravel tarafından sizin için tanımlanmış olacaktır. Bu interface, implemente etmemiz gereken sadece birkaç basit metod içermektedir. Bir MongoDB implementation kalıbı şöyle bir şeydir:

	class MongoHandler implements SessionHandlerInterface {

		public function open($savePath, $sessionName) {}
		public function close() {}
		public function read($sessionId) {}
		public function write($sessionId, $data) {}
		public function destroy($sessionId) {}
		public function gc($lifetime) {}

	}

Bu metodlar cache `StoreInterface` kadar kolay anlaşılabilir olmadıklarından, en iyisi bu metodların her birinin yaptıklarını kısaca keşfedelim:

- `open` metodu tipik olarak dosya tabanlı oturum depolama sistemlerinde kullanılacaktır. Laravel oturumlar için PHP'nin natif dosya depolamasını kullanan `native` bir session sürücüsü ile geldiğinden, bu metoda neredeyse hiçbir şey koymanız gerekmeyecektir. Onu boş bir kalıp olarak bırakabilirsiniz. PHP'nin bizden bu metodu implemente etmemizi istemesi, gerçekte sadece kötü bir interface tasarımıdır (bunu ileride tartışacağız).
- `close` metodu, `open` metoduna benzer şekilde genellikle gözardı edilebilir. Çoğu sürücü için, bu metod gerekli değildir.
- `read` metodu verilen bir `$sessionId` ile eşlik eden oturum verisinin string versiyonunu döndürecektir. Serileştirmeyi Laravel sizin yerinize yapacağı için, sürücünüzde oturum verisini elde ederken veya depolarken herhangi bir serileştirme veya başka kodlamalar yapmanıza gerek yoktur.
- `write` metodu `$sessionId` ile eşlik eden belirli bir `$data` stringini MongoDB, Dynamo vb gibi kalıcı depo sistemlerine yazacaktır.
- `destroy` metodu `$sessionId` ile eşlik eden veriyi kalıcı depodan kaldıracaktır.
- `gc` metodu bir UNIX timestamp türünde verilen bir `$lifetime` süresinden daha eski tüm oturum verisini imha edecektir. Memcached ve Redis gibi süresi kendiliğinden dolan sistemler için bu metod boş bırakılabilir.

`SessionHandlerInterface` implemente edildikten sonra, onu Session manager ile register etmeye hazırız:

	Session::extend('mongo', function($app)
	{
		return new MongoHandler;
	});

Session sürücüsü kayda geçirildikten sonra `app/config/session.php` yapılandırma dosyamızda `mongo` sürücüsünü kullanabiliriz.

> **Not:** Unutmayın, özel bir session işleyici yazarsanız, onu Packagist'te paylaşın!

<a name="authentication"></a>
## Authentication

Authentication (kimlik doğrulama) da cache ve session araçlarıyla aynı yolla genişletilebilir. Burada da yine aşina olduğumuz `extend` metodunu kullanacağız:

	Auth::extend('riak', function($app)
	{
		// Illuminate\Auth\UserProviderInterface'un implementasyonunu döndür
	});

Bu `UserProviderInterface` implementasyonlarının tek sorumluluğu MySQL, Riak vb gibi kalıcı bir depolama sisteminin bir `UserInterface` implementasyonunu getirmektir. Bu iki interface Laravel authentication mekanizmalarının kullanıcı verisinin nerede depolandığına veya onu temsil etmek için kullanılan sınıf tipine bakılmaksızın fonksiyon görmeye devam etmesini sağlar.

`UserProviderInterface`e bir göz atalım:

	interface UserProviderInterface {

		public function retrieveById($identifier);
		public function retrieveByCredentials(array $credentials);
		public function validateCredentials(UserInterface $user, array $credentials);

	}

`retrieveById` fonksiyonu tipik olarak kullanıcıyı temsil eden sayısal bir anahtar alır (örneğin bir MySQL veritabanındaki otomatik artan ID gibi). Metod tarafından, bu ID'e uyan `UserInterface` implementasyonu getirilecek ve döndürülecektir.

`retrieveByCredentials` metodu bir uygulamaya giriş yapma girişiminde bulunulduğu zaman `Auth::attempt` metoduna geçilen kimlik bilgilerinden oluşan diziyi alır. Bu metod daha sonra bu kimlik bilgilerine uyan kullanıcıyı altta yatan kalıcı depolama sisteminden "sorgulamalıdır". Tipik olarak, bu metod `$credentails['username']` üzerine bir "where" koşulu olan bir sorgu çalıştıracaktır. **Bu metod herhangi bir şifre doğrulaması veya authentication yapmaya kalkışmamalıdır.**

`validateCredentials` metodu kullanıcı kimliğini doğrulamak için verilen bir `$user` ile `$credentials`i karşılaştırır. Örneğin, bu metod `$user->getAuthPassword()` stringini `$credentials['password']`in bir `Hash::make` hali ile karşılaştırabilir.

Artık `UserProviderInterface` metodlarının her birini keşfettiğimize göre, bir de `UserInterface`e göz atalım. Hatırlayınız, providerin `retrieveById` ve `retrieveByCredentials` metodları bu interface'in implementasyonlarını döndürecektir:

	interface UserInterface {

		public function getAuthIdentifier();
		public function getAuthPassword();

	}

Bu interface basittir. `getAuthIdentifier` metodu kullanıcının "birincil anahtarını" döndürmelidir. Bir MySQL back-endinde, bu yine otomatik artan birincil anahtar olacaktır. `getAuthPassword` kullanıcının hash'lenmiş şifresini döndürmelidir. Bu interface, sizin kullandığınız ORM veya depolama soyutlama katmanı ne olursa olsun,   authentication sisteminin herhangi bir User sınıfı ile çalışmasına imkan verir. Ön tanımlı olarak Laravel `app/models` klasörü içinde bu interface'i implemente eden bir `User` sınıfı bulundurur, bu yüzden bir implementasyon örneğini görmek için bu sınıfa başvurabilirsiniz.

Son olarak, `UserProviderInterface` implemente edildikten sonra, genişletmemizi `Auth` facade'ı ile kayda geçirmeye hazırız:

	Auth::extend('riak', function($app)
	{
		return new RiakUserProvider($app['riak.connection']);
	});

Sürücüyü `extend` metodu ile register ettikten sonra, `app/config/auth.php` yapılandırma dosyanızda yeni sürücüyü belirtin.

<a name="ioc-based-extension"></a>
## IoC Temelli Genişletme

Laravel frameworke dahil edilen hemen her hizmet sağlayıcı IoC konteynerine nesneler bağlar. Uygulamanızın hizmet sağlayıcılarının bir listesini `app/config/app.php` yapılandırma dosyasında bulabilirsiniz. Vaktiniz oldukça bu sağlayıcıların her birinin kaynak koduna baştan sona göz gezdiriniz. Bunu yapmakla, her bir sağlayıcının frameworke neler eklediğini çok daha iyi anlayacaksınız, bunun yanı sıra IoC konteynerine çeşitli hizmetleri bağlamak için hangi anahtarların kullanıldığını da öğreneceksiniz.

Örneğin, `PaginationServiceProvider` IoC konteynerine bir `paginator` anahtarı bağlamaktadır ve bu anahtar bir `Illuminate\Pagination\Environment` olgusuna çözümlenmektedir. Bu IoC bağlamasını override etmek suretiyle kendi uygulamanızda bu sınıfı kolaylıkla genişletebilir ve override edebilirsiniz. Örneğin, taban `Environment` sınıfını genişleten bir sınıf oluşturabilirsiniz:

	namespace Snappy\Extensions\Pagination;

	class Environment extends \Illuminate\Pagination\Environment {

		//

	}

Sınıf genişletmenizi oluşturduktan sonra, yeni bir `SnappyPaginationProvider` hizmet sağlayıcı sınıfı oluşturarak bunun `boot` metodunda paginator'u override edebilirsiniz:

	class SnappyPaginationProvider extends PaginationServiceProvider {

		public function boot()
		{
			App::bind('paginator', function()
			{
				return new Snappy\Extensions\Pagination\Environment;
			});

			parent::boot();
		}

	}

Bu sınıfın ön tanımlı `ServiceProvider` taban sınıfını değil `PaginationServiceProvider` sınıfını genişlettiğine dikkat ediniz. Service providerinizi genişlettikten sonra, `app/config/app.php` yapılandırma dosyanızdaki `PaginationServiceProvider`ı sizin genişletilmiş providerin ismi ile takas edin.

Konteynerde bağlanan herhangi bir çekirdek sınıfın genişletilmesi için genel yöntem budur. Esasında, her çekirdek sınıf konteynerde bu tarzda bağlanır ve override edilebilir. Tekrar ifade edeyim, frameworkte yer alan hizmet sağlayıcılarının baştan sona okunması çeşitli sınıfların konteynerde nerede bağlandığı ve onu bağlamak için hangi anahtarın kullanıldığı konusunda sizi bilgilendirecektir. Laravelin nasıl biraraya getirildiğini daha çok öğrenmek için harika bir yoldur.

<a name="request-extension"></a>
## Request Genişletmesi

Request frameworkün çok temel bir parçası olduğu ve istek döngüsünde çok erken başlatıldığı için, `Request` sınıfının genişletilmesi önceki örneklerden biraz farklı yapılır.

İlk olarak, sınıfı normaldeki gibi genişletin:

	<?php namespace QuickBill\Extensions;

	class Request extends \Illuminate\Http\Request {

		// Burada özel, yararlı metodlar olacak...

	}

Sınıfı genişlettikten sonra, `bootstrap/start.php` dosyasını açın. Bu dosya, uygulamanıza yapılan her istekte en başta dahil edilen dosyalardan biridir. Dikkat ederseniz, bu dosyada yapılan ilk eylem Laravel'in `$app` olgusunun oluşturulmasıdır:

	$app = new \Illuminate\Foundation\Application;

Yeni bir application olgusu oluşturulduğu zaman, yeni bir `Illuminate\Http\Request` olgusu oluşturacak ve `request` anahtarını kullanarak onu IoC konteynerine bağlayacaktır. Bu yüzden, "default" istek tipi olarak kullanılması gereken özel bir sınıfı belirten bir yola ihtiyacımız var, değil mi? Ve, ne mutlu ki, application olgusundaki `requestClass` metodu tam bunu yapar! Yani, `bootstrap/start.php` dosyamızın en üstüne şu satırı ekleyebiliriz:

	use Illuminate\Foundation\Application;

	Application::requestClass('QuickBill\Extensions\Request');

Özel istek sınıfı belirtildikten sonra, Laravel bir `Request` olgusu oluşturduğu her zaman bu sınıfı kullanacaktır, böylece sizin özel request sınıfı olgunuzun, unit testlerde bile, her zaman kullanılabilir olmasına imkan verecektir!