# Unit Testing

- [Giriş](#introduction)
- [Testleri Tanımlamak ve Çalıştırmak](#defining-and-running-tests)
- [Test Ortamı](#test-environment)
- [Testlerin İçerisinden Rotaları Çağırmak](#calling-routes-from-tests)
- [Facade'ları Taklit Etmek](#mocking-facades)
- [Çatının "Assert" Metodları](#framework-assertions)
- [Yardımcı Metodlar](#helper-methods)
- [Application'ın Tazelenmesi](#refreshing-the-application)

<a name="introduction"></a>
## Giriş

Laravel hazırlanırken birim testler ile hazırlandı. Açıkçası, PHPUnit ile test desteği halihazırda var ve uygulamanız için hazırlanmış `phpunit.xml` dosyası da Laravel ile birlikte geliyor. PHPUnit'in haricinde, Laravel ayrıca Symfony HttpKernel, DomCrawler ve BrowserKit bileşenlerinden de yararlanıyor ki, kullanıcılar testler sırasında görünümleri inceleyebilsin ve müdahale edebilsin. Bu bileşenler ile temsili bir tarayıcıya sahip oluyorsunuz.

Örnek bir test dosyası `app/tests` dizininde bulunmaktadır. Yeni bir Laravel uygulaması kurulumundan sonra, komut satırında `phpunit` komutuyla testlerinizi çalıştırabilirsiniz.

<a name="defining-and-running-tests"></a>
## Testleri Tanımlamak ve Çalıştırmak

Yeni bir test durumu oluşturmak için, `app/test` dizini içerisinde yeni bir test dosyası oluşturmanız yeterli. Test sınıflarınız `TestCase` sınıfını extend ediyor olmalıdır. Bu şekilde normalde PHPUnit ile hazırladığınız test metodlarını aynı şekilde oluşturabilirsiniz.

#### Örnek Bir Test Sınıfı

	class FooTest extends TestCase {

		public function testSomethingIsTrue()
		{
			$this->assertTrue(true);
		}

	}

Daha sonra komut satırında `phpunit` ile uygulamanızın tüm testlerini çalıştırabilirsiniz.

> **Not:** Eğer kendi `setUp` methodunuzu tanımlarsanız, `parent::setUp` kodunu çalıştırdığınızdan emin olun.

<a name="test-environment"></a>
## Test Ortamı

Testleri çalıştırırken, Laravel otomatik olarak ortam yapılandırmasını `testing`'e alacaktır. Ayrıca, Laravel'de test ortamında `önbellekleme` ve `oturum` için özel ayar dosyaları bulunmaktadır. İki sürücü de bir `dizi` olacak şekilde ayarlanmış olup, test yaparkenki oturum ve önbellek verilerinin kalıcı olmaması sağlanmıştır. Test ortamı için gerektiğinde başka ayarlar yapmakta özgürsünüz.

<a name="calling-routes-from-tests"></a>
## Testlerin İçerisinden Rotaları Çağırmak

#### Test Dosyasından Bir Rota Çağırmak

Testleriniz içerisinde `call` metodu ile rahatlıkla rotaları çağırabilirsiniz:

	$response = $this->call('GET', 'user/profile');

	$response = $this->call($method, $uri, $parameters, $files, $server, $content);

Daha sonra `Illuminate\Http\Response` nesnesini inceleyebilirsiniz:

	$this->assertEquals('Hello World', $response->getContent());

#### Test Dosyasından Bir Denetçi Çağırmak

Ayrıca bir test dosyasından denetçileri de çağırabilirsiniz:

	$response = $this->action('GET', 'HomeController@index');

	$response = $this->action('GET', 'UserController@profile', array('user' => 1));

`getContent` metodu cevap olarak değerlendirilmiş string içeriğini döndürecektir. Eğer rotanız bir `Görünüm` döndürüyorsa, `original` özelliği ile buna ulaşabilirsiniz:

	$view = $response->original;

	$this->assertEquals('Tuana Şeyma', $view['name']);

Bir HTTPS rotayı çağırmak için, `callSecure` metodunu kullanabilirsiniz.

	$response = $this->callSecure('GET', 'falan/filan');

> **Not:** Testing ortamında olduğunuzda rota filtreleri devre dışı bırakılır. Bunları etkinleştirmek için testinize `Route::enableFilters()` ekleyin.

### DOM Böceği

Ayrıca bir rota çağırıp, bir DOM Böceği nesnesi alarak içeriği inceleyebilirsiniz:

	$crawler = $this->client->request('GET', '/');

	$this->assertTrue($this->client->getResponse()->isOk());

	$this->assertCount(1, $crawler->filter('h1:contains("Hello World!")'));

Böceği nasıl kullanacağınız hakkında detaylı bilgi için [kendi dökümantasyonunu](http://symfony.com/doc/master/components/dom_crawler.html) okuyabilirsiniz.

<a name="mocking-facades"></a>
## Facade'ları Taklit Etmek

Test yaparken, sabit Laravel facadelarını taklit etmeniz gerekecektir. Örneğin, şu denetçi aksiyonunu varsayalım:

	public function getIndex()
	{
		Event::fire('falan', array('name' => 'Sergin Arı'));

		return 'Herşey yolunda!';
	}

`Event` sınıfına yapılan çağrıyı taklit edebilmek için Facade üzerinde `shouldReceive` metodunu kullanabilirsiniz, bu metod bir [Mockery](https://github.com/padraic/mockery) olgusu döndürecek.

#### Bir Facade'ı Taklit Etmek

	public function testGetIndex()
	{
		Event::shouldReceive('fire')->once()->with(array('name' => 'Sergin Arı'));

		$this->call('GET', '/');
	}

> **Not:** `Request` metodunu taklit etmemelisiniz. Bunun yerine, testlerinizi çalıştırırken istediğiniz girdileri `call` metodunda belirtin.

<a name="framework-assertions"></a>
## Çatının `Assert` Metodları

Laravel test yapımını kolaylaştırmak için halihazırda bazı `assert` metodlarıyla gelir:

#### Yanıtın Başarıyla Geldiği Ispatlamak

	public function testMethod()
	{
		$this->call('GET', '/');

		$this->assertResponseOk();
	}

#### Yanıt Kodlarını Ispatlamak

	$this->assertResponseStatus(403);

#### Yanıtın Bir Yönlendirme Olduğunu Ispatlamak

	$this->assertRedirectedTo('falan');

	$this->assertRedirectedToRoute('route.name');

	$this->assertRedirectedToAction('Controller@method');

#### Bir Görünümde Veri Olduğunu Ispatlamak

	public function testMethod()
	{
		$this->call('GET', '/');

		$this->assertViewHas('name');
		$this->assertViewHas('age', $value);
	}

#### Oturumda Bir Verinin Kayıtlı Olduğunu Ispatlamak

	public function testMethod()
	{
		$this->call('GET', '/');

		$this->assertSessionHas('name');
		$this->assertSessionHas('age', $value);
	}

#### Session'da Hatalar Olup Olmadığını Ispatlama

	public function testMethod()
	{
		$this->call('GET', '/');

		$this->assertSessionHasErrors();

		// Verilen bir anahtar için sessionda hata olup olmadığına bakmak...
		$this->assertSessionHasErrors('name');

		// Birkaç anahtar için sessionda hata olup olmadığına bakmak...
		$this->assertSessionHasErrors(array('name', 'age'));
	}

#### Eski Girdide Veri Olduğunu Ispatlamak

	public function testMethod()
	{
		$this->call('GET', '/');

		$this->assertHasOldInput();
	}

<a name="helper-methods"></a>
## Yardımcı Metodlar

Test yapımını kolaylaştırmak için `TestCase` sınıfı bazı yardımcı metodlarla birlikte gelir.

#### Test İçerisinden Oturum Tanımlamak ve Silmek

	$this->session(['falan' => 'filan']);

	$this->flushSession();

#### Oturum Açmış Kullanıcıyı Belirleme

Mevcut oturum açmış kullanıcıyı `be` metodu ile belirleyebilirsiniz.

	$user = new User(array('name' => 'Tuana Şeyma'));

	$this->be($user);

Bir test içerisinden `seed` metoduyla veritabanınıza yeniden veri ekebilirsiniz:

#### Test İçerisinden Veritabanına Yeniden Veri Ekmek

	$this->seed();

	$this->seed($connection);

Veri Ekmeyle ilgili daha fazla bilgiyi dökümantasyonun [migrasyon ve veri ekme](/docs/migrations#database-seeding) bölümünde bulabilirsiniz.

<a name="refreshing-the-application"></a>
## Application'ın Tazelenmesi

Belki de zaten bildiğiniz gibi, Laravel `Application` / IoC Konteynerinize herhangi bir test metodundan `$this->app` aracılığıyla erişebilirsiniz. Bu Application olgusu her test sınıfı için tazelenir. Şayet siz Application'ı verilen bir metod için elle tazelenmeye zorlamak istiyorsanız, test metodunuzdan `refreshApplication` metodunu kullanabilirsiniz. Bu, test durumu çalıştırılmaya başlandıktan itibaren IoC konteynerine konmuş olan herhangi bir ekstra bağlamayı, örneğin mocks (taklit)'ları resetleyecektir.