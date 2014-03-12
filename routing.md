# Rotalar

- [Temel Rotalama](#basic-routing)
- [Rota Parametreleri](#route-parameters)
- [Rota Filtreleri](#route-filters)
- [İsimli Rotalar](#named-routes)
- [Rota Grupları](#route-groups)
- [Alt Alanadı (Subdomain) Rotalaması](#sub-domain-routing)
- [Rotalarda Ön-ek](#route-prefixing)
- [Rotalara Model Bağlama](#route-model-binding)
- [404 Hatası Fırlatma](#throwing-404-errors)
- [Denetçilere (Controller) Rotalama](#routing-to-controllers)

<a name="basic-routing"></a>
## Temel Rotalama

Uygulamanızdaki rotaların çoğu `app/routes.php` dosyasında tanımlanır. En temel Laravel rotası "URL deseni" ve `closure (anonim fonksiyon)`'dan oluşur.

#### Temel GET Rotası

	Route::get('/', function()
	{
		return 'Merhaba Laravel!';
	});

#### Temel POST Rotası

	Route::post('bir/sey', function()
	{
		return 'Merhaba Laravel!';
	});

#### Registering A Route For Multiple Verbs

	Route::match(array('GET', 'POST'), '/', function()
	{
		return 'Hello World';
	});

#### Tüm HTTP Metodları (GET, POST gibi) İçin Rota Yazımı

	Route::any('falan', function()
	{
		return 'Merhaba Laravel!';
	});

#### Rotanın Zorunlu Olarak HTTPS Üzerinden Kullanılmasını Sağlamak

	Route::get('falan', array('https', function()
	{
		return 'HTTPS üzerinde olmalı!';
	}));

Çoğu zaman, rotalarınız için URL'ler üretmeniz gerekecek. Bunu `URL::to` metoduyla yapabilirsiniz:

	$url = URL::to('falan');

<a name="route-parameters"></a>
## Rota Parametreleri

	Route::get('kullanici/{id}', function($id)
	{
		return 'Kullanıcı NO: '.$id;
	});

#### İsteğe Bağlı Rota Parametreleri

	Route::get('kullanici/{isim?}', function($isim = null)
	{
		return $isim;
	});

#### Öntanımlı Değerli İsteğe Bağlı Rota Parametreleri

	Route::get('kullanici/{isim?}', function($isim = 'Ali')
	{
		return $isim;
	});

#### Rotalarda Düzenli İfade Kontrolü

	Route::get('kullanici/{isim}', function($isim)
	{
		//
	})
	->where('isim', '[A-Za-z]+');

	Route::get('kullanici/{id}', function($id)
	{
		//
	})
	->where('id', '[0-9]+');

#### Passing An Array Of Wheres

Tabii ki kuralları bir dizi hâlinde tanımlayabilirsiniz:

	Route::get('kullanici/{id}/{isim}', function($id, $isim)
	{
		//
	})
	->where(array('id' => '[0-9]+', 'isim' => '[a-z]+'))

#### Defining Global Patterns

If you would like a route parameter to always be constrained by a given regular expression, you may use the `pattern` method:

	Route::pattern('id', '[0-9]+');

	Route::get('user/{id}', function($id)
	{
		// Only called if {id} is numeric.
	});

#### Accessing A Route Parameter Value

If you need to access a route parameter value outside of a route, you may use the `Route::input` method:

	Route::filter('foo', function()
	{
		if (Route::input('id') == 1)
		{
			//
		}
	});

<a name="route-filters"></a>
## Rota Filtreleri

Rota filtreleri, sitenizin yetkilendirme gereken alanlarına erişimi kısıtlamak için uygun bir yoldur. Laravel'de `auth`, `auth.basic`, `guest`, `csrf` gibi `app/filters.php` dosyasında tanımlı filtreler vardır.

#### Rota Filtresi Tanımlama

	Route::filter('yas', function()
	{
		if (Input::get('yas') < 200)
		{
			return Redirect::to('anasayfa');
		}
	});

Eğer filtreden bir yanıt (`Redirect::to` gibi) döndürülürse, bu cevap olarak kabul edilecek ve rota çalıştırılmayacaktır, rotada olabilecek `after` filtreleri de iptal edilecektir.

#### Rotaya Filtre Ekleme

	Route::get('kullanici', array('before' => 'yas', function()
	{
		return '200 yaş üzerisin!';
	}));

#### Attaching A Filter To A Controller Action

	Route::get('user', array('before' => 'old', 'uses' => 'UserController@showProfile'));

#### Rotaya Birden Çok Filtre Ekleme

	Route::get('user', array('before' => 'auth|yas', function()
	{
		return '200 yaşın üzerisin ve giriş yetkin var!';
	}));

#### Filtre Parametrelerini Belirtme

	Route::filter('yas', function($rota, $istek, $deger)
	{
		//
	});

#### Specifying Filter Parameters

	Route::filter('age', function($route, $request, $value)
	{
		//
	});

	Route::get('kullanici', array('before' => 'yas:200', function()
	{
		return 'Merhaba Laravel!';
	}));

After filtreleri filtreye üçüncü parametre olarak geçilen bir `$yanit` parametresi alırlar:

	Route::filter('log', function($rota, $istek, $yanit, $deger)
	{
		//
	});

#### Desenli Filtreler

URL desenine göre de rotalara filtre ataması yapabilirsiniz.

	Route::filter('admin', function()
	{
		//
	});

	Route::when('admin/*', 'admin');

Yukarıdaki örnekte, `admin` filtresi `admin/` ile başlayan tüm rotalara uygulanacaktır. `*` karakteri tüm karakterleri yakalamak için kullanılır.

Filtreleri HTTP metodlarına (GET, POST gibi) göre uygulayabilirsiniz.

	Route::when('admin/*', 'admin', array('post'));

#### Filtre Sınıfları

Daha gelişmiş filtreler için closure yerine sınıfları kullanmak isteyebilirsiniz. Filtre sınıfları uygulama [IoC Konteyneri](/docs/ioc)nde çözümlendikleri için, daha fazla test edilebilirlik için bu filtrelerde bağımlılık enjeksiyonu kullanmanız mümkün olabilecektir.

#### Sınıf Temelli Filtre Oluşturma

	Route::filter('foo', 'FalanFiltresi');

By default, the `filter` method on the `FalanFiltresi` class will be called:

	class FalanFiltresi {

		public function filter()
		{
			// Filtre işlemleri...
		}

	}

If you do not wish to use the `filter` method, just specify another method:

	Route::filter('falan', 'FalanFiltresi');

<a name="named-routes"></a>
## İsimli Rotalar

İsimli rotalar link veya yönlendirme oluştururken kolaylık sağlar. Bir rotayı şöyle isimlendirebilirsiniz:

	Route::get('kullanici/profil', array('as' => 'profil', function()
	{
		//
	}));

Denetçi metodları için de rota isimleri belirleyebilirsiniz:

	Route::get('kullanici/profil', array('as' => 'profil', 'uses' => 'KullaniciController@profilGoster'));

Şimdi, rota isimlerini link veya yönlendirme oluştururken kullanabilirsiniz:

	$url = URL::route('profil');

	$yonlendirme = Redirect::route('profil');

Çalışan rotanın ismine `currentRouteName` metoduyla ulaşabilirsiniz:

	$isim = Route::currentRouteName();

<a name="route-groups"></a>
## Rota Grupları

Bazen bir grup rotaya filtre atamanız gerekebilir. Her birine ayrı filtre atamaktansa, rota gruplarını kullanabilirsiniz:

	Route::group(array('before' => 'auth'), function()
	{
		Route::get('/', function()
		{
			// Yetki gerekir. ("auth" filtresi)
		});

		Route::get('user/profile', function()
		{
			// Yetki gerekir. ("auth" filtresi)
		});
	});

You may also use the `namespace` parameter within your `group` array to specify all controllers within that group as being in a given namespace:

	Route::group(array('namespace' => 'Admin'), function()
	{
		//
	});

<a name="sub-domain-routing"></a>
## Alt Alanadı (Subdomain) Rotalaması

Laravel rotaları ile alt-alanadlarını yakalayabilir ve parametre olarak kullanabilirsiniz.

#### Alt-alanadı Rotası Tanımlama

	Route::group(array('domain' => '{hesapadi}.uygulamam.com'), function()
	{

		Route::get('kullanici/{id}', function($hesapadi, $id)
		{
			//
		});

	});

<a name="route-prefixing"></a>
## Rotalarda Ön-ek

`prefix` seçeneğini kullanarak gruptaki rotalara ön-ek ekleyebilirsiniz:

#### Gruplanmış Rotalara Ön-ek Ekleme

	Route::group(array('prefix' => 'admin'), function()
	{

		Route::get('kullanici', function()
		{
			//
		});

	});

<a name="route-model-binding"></a>
## Rotalara Model Bağlama

Model bağlama model sınıflarının rotalarda kullanılması için kolaylık sağlar. Mesela, bir kullanıcının ID'sinin aktarılması yerine, ID ile eşleşen Kullanici modelini aktarabilirsiniz. İlk olarak, girilen parametreler için kullanılacak modelleri `Route::model` metoduyla belirleyin:

#### Parametrelere Model Bağlama

	Route::model('kullanici', 'Kullanici');

Daha sonra, `{kullanici}` parametresini içeren bir rota belirleyin:

	Route::get('profil/{kullanici}', function(Kullanici $kullanici)
	{
		//
	});

`{kullanici}` parametresi ile `Kullanici` modelini eşleştirdiğimizden, bir `Kullanici` nesnesi rotaya aktarılacaktır. Yani, `profil/1` şeklindeki istek, ID'si 1 olan `Kullanici` nesnesini aktaracaktır.

> **Not:**  Eğer model için veritabanında eşleşme yapılamazsa, 404 hatası fırlatılır.

Eğer eşleşmeme durumunda yapılacak işlemi kendiniz belirlemek istiyorsanız, `model` metoduna 3. argüman olarak bir closure ekleyebilirsiniz:

	Route::model('kullanici', 'Kullanici', function()
	{
		throw new NotFoundException;
	});

Modeller yerine kendi tanımlayıcınızı kullanmak isteyebilirsiniz. Bunun için `Route::bind` metodu kullanılır:

	Route::bind('kullanici', function($deger, $rota)
	{
		return Kullanici::where('isim', $deger)->first();
	});

<a name="throwing-404-errors"></a>
## 404 Hatası Fırlatma

Rotadan 404 hatasını fırlatmanın iki yolu vardır. İlk olarak, `App::abort` metodunu kullanabilirsiniz.

	App::abort(404);

İkinci, `Symfony\Component\HttpKernel\Exception\NotFoundHttpException` nesnesi fırlatmaktır.

404 hatalarının yakalanması ve özel yanıtla oluşturulması hakkında daha fazla bilgiye dokümantasyonun [hatalar](/docs/errors#handling-404-errors) bölümünden ulaşabilirsiniz.

<a name="routing-to-controllers"></a>
## Denetçilere (Controller) Rotalama

Laravel sadece closure'lara değil, aynı zamanda denetçi sınıflarına rotalamaya da imkan verir ve hatta [kaynak denetçileri](/docs/controllers#resource-controllers) oluşturulması da mümkündür.

Daha fazla bilgi için [Denetçiler](/docs/controllers) konusunu inceleyin.