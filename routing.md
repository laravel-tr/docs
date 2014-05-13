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

#### Bir Rotanın Birden Çok HTTP Fiili İçin Kayda Geçirilmesi

	Route::match(array('GET', 'POST'), '/', function()
	{
		return 'Merhaba Laravel!';
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

#### Bir Where'ler Dizisi Geçilmesi

Tabii ki kuralları bir dizi hâlinde tanımlayabilirsiniz:

	Route::get('kullanici/{id}/{isim}', function($id, $isim)
	{
		//
	})
	->where(array('id' => '[0-9]+', 'isim' => '[a-z]+'))

#### Global Desenler Tanımlanması

Bir rota parametresinin her zaman için verilen bir düzenli ifadeyle sınırlandırılmasını istiyorsanız, `pattern` metodunu kullanabilirsiniz:

	Route::pattern('id', '[0-9]+');

	Route::get('kullanici/{id}', function($id)
	{
		// Sadece {id} sayısal ise çağrılır.
	});

#### Bir Rota Parametre Değerine Erişmek

Bir rotanın dışında bir rota parametre değerine erişmeniz gerekirse `Route::input` metodunu kullanabilirsiniz:

	Route::filter('falan', function()
	{
		if (Route::input('id') == 1)
		{
			//
		}
	});

<a name="route-filters"></a>
## Rota Filtreleri

Rota filtreleri, sitenizin yetkilendirme gereken alanlarına erişimi kısıtlamak için uygun bir yoldur. Laravel frameworkte aralarında bir `auth` filtresi, bir `auth.basic` filtresi, bir `guest`filtresi ve bir `csrf` filtresinden oluşan birkaç filtre dahil edilmiştir. Bunlar `app/filters.php` dosyasında bulunmaktadır.

#### Rota Filtresi Tanımlama

	Route::filter('yas', function()
	{
		if (Input::get('yas') < 200)
		{
			return Redirect::to('anasayfa');
		}
	});

Eğer filtreden bir yanıt (`Redirect::to` gibi) döndürülürse, bu cevap cevap olarak kabul edilecek ve rota çalıştırılmayacaktır. Rotada olabilecek `after` filtreleri de iptal edilecektir.

#### Bir Rotaya Bir Filtre Bağlanması

	Route::get('kullanici', array('before' => 'yas', function()
	{
		return '200 yaş üzerisin!';
	}));

#### Bir Controller Eylemine Bir Filtre Bağlanması

	Route::get('kullanici', array('before' => 'yas', 'uses' => 'UserController@showProfile'));

#### Rotaya Birden Çok Filtre Bağlanması

	Route::get('kullanici', array('before' => 'auth|yas', function()
	{
		return '200 yaşın üzerisin ve giriş yetkin var!';
	}));

#### Birden Çok Filtrenin Dizi Yoluyla Bağlanması

	Route::get('kullanici', array('before' => array('auth', 'yas'), function()
	{
		return '200 yaşın üzerisin ve giriş yetkin var!';
	}));

#### Filtre Parametrelerinin Belirtilmesi

	Route::filter('yas', function($route, $request, $value)
	{
		//
	});

	Route::get('kullanici', array('before' => 'yas:200', function()
	{
		return 'Merhaba Laravel!';
	}));

After filtreleri filtreye üçüncü parametre olarak geçilen bir `$response` parametresi alırlar:

	Route::filter('log', function($route, $request, $response)
	{
		//
	});

#### Desen Temelli Filtreler

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

Daha gelişmiş filtreler için Closure yerine sınıfları kullanmak isteyebilirsiniz. Filtre sınıfları uygulama [IoC Konteyneri](/docs/ioc)nde çözümlendikleri için, daha fazla test edilebilirlik için bu filtrelerde bağımlılık enjeksiyonu kullanmanız mümkün olabilecektir.

#### Sınıf Temelli Filtrenin Kayda Geçirilmesi

	Route::filter('falan', 'FalanFiltresi');

Ön tanımlı olarak, `FalanFiltresi` sınıfındaki `filter` metodu çağrılacaktır:

	class FalanFiltresi {

		public function filter()
		{
			// Filtre işlemleri...
		}

	}

Bu `filter` metodunu kullanmak istemiyorsanız, başka bir metod belirtebilirsiniz:

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

Ayrıca, grup içindeki tüm controllerlerin verilen bir aduzayında olacağını belirtmek için `group` array'iniz içerisinde `namespace` parametresini kullanabilirsiniz:

	Route::group(array('namespace' => 'Admin'), function()
	{
		//
	});

<a name="sub-domain-routing"></a>
## Alt Alanadı (Subdomain) Rotalaması

Laravel rotaları joker alt alanadlarını da işleyebilirler ve joker parmetreleri alanadına geçebilirsiniz.

#### Alt Alanadı Rotası Tanımlama

	Route::group(array('domain' => '{hesapadi}.uygulamam.com'), function()
	{

		Route::get('kullanici/{id}', function($hesapadi, $id)
		{
			//
		});

	});

<a name="route-prefixing"></a>
## Rotalarda Ön-ek

Bir grubun nitelikler dizisinde `prefix` seçeneğini kullanarak gruptaki rotalara ön-ek ekleyebilirsiniz:

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

Eğer eşleşmeme durumunda yapılacak işlemi kendiniz belirlemek istiyorsanız, `model` metoduna 3. argüman olarak bir Closure ekleyebilirsiniz:

	Route::model('kullanici', 'Kullanici', function()
	{
		throw new NotFoundHttpException;
	});

Bazen, rota parametreleri için kendi çüzümleyicinizi kullanmak isteyebilirsiniz. Bunun için `Route::bind` metodu kullanılır:

	Route::bind('kullanici', function($deger, $route)
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

Laravel sadece Closure'lara değil, aynı zamanda denetçi sınıflarına rotalamaya da imkan verir ve hatta [kaynak denetçileri](/docs/controllers#resource-controllers) oluşturulması da mümkündür.

Daha fazla bilgi için [Denetçiler](/docs/controllers) konusunu inceleyin.