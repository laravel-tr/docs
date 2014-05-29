# Denetçiler (Controllers)

- [Temel Denetçiler](#basic-controllers)
- [Denetçi Filtreleri](#controller-filters)
- [TEDA-uyumlu (TEmsili Durum Aktarma uyumlu, RESTful) Denetçiler](#restful-controllers)
- [Kaynak (Resource) Denetçileri](#resource-controllers)
- [Eksik Olan Metodların Yönetilmesi](#handling-missing-methods)

<a name="basic-controllers"></a>
## Temel Denetçiler

Bütün rotalandırma mantığını, tüm rotaları tek tek `routes.php` dosyasında tanımlamak yerine, bu davranışlarını Denetçiler (Controllers) sınıflarını kullanarak organize edebilirsiniz. Denetçiler, ilişkin oldukları rotaların mantığını bir sınıfta gruplar. Aynı zamanda, daha ileri çerçeve (framework) özelliklerini kullanma avantajına sahiptirler, örneğin otomatik [dependency injection](/docs/ioc) (bağımlılık enjeksiyonu) gibi.

Denetçiler genelde `app/controllers` dizininde konumlandırılır ve varsayılan olarak bu dizin `composer.json` dosyanızın `classmap` seçeneğinde kayda geçirilmiştir.

Basit bir denetçi (controller) sınıfı örneği şöyledir:

	class KullaniciController extends BaseController {

		/**
		 * Verilen kullanıcının profilini göster.
		 */
		public function showProfile($id)
		{
			$kullanici = Kullanici::find($id);

			return View::make('kullanici.profil', array('kullanici' => $kullanici));
		}

	}

Bütün denetçilerin `BaseController` sınıfından türetilmiş olması gerekir. `BaseController` ın kendisi de `app/controllers` dizininde bulunur ve bütün denetçiler için geçerli olacak ortak mantığın içine yerleştirilmesinde kullanılabilir. `BaseController` sınıfı, framework'ün `Controller` sınıfının uzantısıdır. Bu durumda, oluşturmuş olduğumuz denetçi fonksiyonuna rotalandırmayı şu şekilde yapabiliriz:

	Route::get('kullanici/{id}', 'KullaniciController@showProfile');

Eğer bir denetçinizi, dizin içerisinde yuvalandırarak (nest) veya PHP aduzayları (namespaces) kullanarak organize etmek isterseniz, bu durumda rotayı tanımlarken, tam nitelendirilmiş (yani, aduzayıyla birlikte) sınıf adını kullanınız:

	Route::get('falanca', 'Namespace\FalancaController@metodAdi');

> **Not:** PHP sınıflarımızı otomatik yüklemek için [Composer](http://getcomposer.org) kullandığımız için, composer onların nasıl yükleneceğini bildiği sürece controllerler dosya sistemindeki herhangi bir yerde bulunabilir. Controller dizini uygulamanız için herhangi bir klasör yapısını zorlamaz. Controller'lara rotalama dosya sisteminden tamamen ayrık tutulmuştur.

Denetçi rotalarına isimler de verebilirsiniz:

	Route::get('falanca', array('uses' => 'FalancaController@metodAdi',
		'as' => 'rotaAdi'));

Herhangi bir denetçi eylemine ait bir URL üretmek için, `URL::action` metodunu veya `action` helper metodunu kullanabilirsiniz:

	$url = URL::action('FalancaController@metodAdi');

	$url = action('FalancaController@metodAdi');

Çalıştırılmakta olan bir denetçi eyleminin ismine `currentRouteAction` metodu ile erişebilirsiniz:

	$action = Route::currentRouteAction();

<a name="controller-filters"></a>
## Denetçi Filtreleri

Denetçi rotalarına, diğer rotalarda olduğuna benzer şekilde, filtreler [Filters](/docs/routing#route-filters) belirlenebilir:

	Route::get('profile', array('before' => 'auth',
				'uses' => 'KullaniciController@showProfile'));

Filtreleri, denetçinizin içerisinden de belirtebilirsiniz:

	class KullaniciController extends BaseController {

		/**
		 * Yeni bir KullaniciController olgusu başlat.
		 */
		public function __construct()
		{
			$this->beforeFilter('auth', array('except' => 'getGiris'));

			$this->beforeFilter('csrf', array('on' => 'post'));

			$this->afterFilter('log', array('only' =>
				array('falancaMetod', 'filancaMetod')));
		}

	}

Denetçi filtrelerini bir Closure kullanarak da belirtebilirsiniz:

	class KullaniciController extends BaseController {

		/**
		 * Yeni bir KullaniciController olgusu başlat.
		 */
		public function __construct()
		{
			$this->beforeFilter(function()
			{
				//
			});
		}

	}

Eğer filtre olarak denetçi sınıfın bir metodunu kullanmak isterseniz, filtre isminin önüne `@` koymalısınız.

	class UserController extends BaseController {

		/**
		 * Yeni bir KullaniciController olgusu başlat.
		 */
		public function __construct()
		{
			$this->beforeFilter('@filterRequests');
		}

		/**
		 * Gelen istekleri filtrele.
		 */
		public function filterRequests($route, $request)
		{
			//
		}

	}

<a name="restful-controllers"></a>
## TEDA-uyumlu (Temsili Durum Aktarma uyumlu, RESTful) Denetçiler

Laravel size, basit TEDA (REST) isimlendirme gelenekleri kullanarak, belirleyeceğiniz tek bir rota ile, denetçilerinizin içindeki her eylemi kullanabilme imkanını tanır. İlk olarak, `Route::controller` metodu ile bu rotayı tanımlayınız:

	Route::controller('kullanicilar', 'KullaniciController');

`controller` metodu iki argüman alır. Birincisi denetçinin yöneteceği taban URI olup, ikincisi denetçinin sınıf ismidir. Akabinde sadece, isimlerine HTTP eyleminin ön ek olarak ekleneceği ve bunlara cevap verecek olan metodlarınızı denetçinize ilave ediniz:

	class KullaniciController extends BaseController {

		public function getIndex()
		{
			//
		}

		public function postProfile()
		{
			//
		}

		public function anyLogin()
		{
			//
		}

	}

`index` metodları, denetçi tarafından yönetilmekte olan kök URI'a cevap verir. Örneğimizde bu, `kullanicilar` dır.

Denetçinizdeki bir eylem metodunun ismi birden fazla kelimeden oluşuyorsa, bu eylem metoduna, URI da kelime aralarına tire işareti "-" eklenmiş şekilde yazarak erişebilirsiniz. Örneğin, 'KullaniciController' denetçimizdeki aşağıdaki şekilde isimlendirilmiş olan metod, `kullanici/yonetici-profili` URI'na cevap verecektir.

	public function getYoneticiProfili() {}

<a name="resource-controllers"></a>
## Kaynak (Resource) Denetçileri

Kaynak denetçileri, kaynaklar etrafında TEDA-uyumlu denetçiler oluşturulmasını kolaylaştırır. Artisan KSA'daki (Artisan Komut Satırı Arayüzü) `controller:make` komutunu ve de `Route::resource` metodunu kullanmak sureti ile böyle bir denetçiyi çabucak oluşturabiliriz.

Denetçiyi komut satırını kullanarak oluşturmak için şu komutu kullanınız:

	php artisan controller:make FotoController

Şimdi bu controller için resourceful bir rotayı kayda geçirebiliriz (routes.php dosyasında):

	Route::resource('foto', 'FotoController');

Bu tek bir rota deklarasyonu, foto kaynağınız üzerinde çalıştıracağınız çeşitli TEDA-uyumlu eylem metodlarına erişeceğiniz rotalar oluşturur. Aynı zamanda, oluşturulmuş olan denetçide, bu eylemlerin her biri için metodları hazır olarak oluşturulmuş ve hangi URI'ı ve eylemi yönettikleri yanlarına not olarak yazılmış olacaktır.

#### Kaynak Denetçisinin Yöneteceği Eylemler

HTTP Fiili | Patika                | Eylem            | Rota İsmi
-----------|-----------------------|------------------|---------------
GET        | /kaynak               | index            | kaynak.index
GET        | /kaynak/create        | create (oluştur) | kaynak.create
POST       | /kaynak               | store (kaydet)   | kaynak.store
GET        | /kaynak/{id}          | show (göster)    | kaynak.show
GET        | /kaynak/{id}/edit     | edit (düzenle)   | kaynak.edit
PUT/PATCH  | /kaynak/{id}          | update (güncelle)| kaynak.update
DELETE     | /kaynak/{id}          | destroy (imha et)| kaynak.destroy

Bazen bu eylemlerin sadece bazılarına ihtiyaç duyabilirsiniz:

	php artisan controller:make FotoController --only=index,show   //sadece belirtilenleri

	php artisan controller:make FotoController --except=index     //belirtilenler hariç

Ve, aynı zamanda rotasında da eylemlerin sadece bazılarını yönetmesini belirleyebilirsiniz:

	Route::resource('foto', 'FotoController',
			array('only' => array('index', 'show')));

	Route::resource('foto', 'FotoController',
			array('except' => array('create', 'store', 'update', 'destroy')));

Varsayılan olarak tüm kaynak denetçilerinin bir rota ismi bulunur; ancak bu isimleri üçüncü parametrede gireceğiniz dizi ile kendiniz belirleyebilirsiniz.

	Route::resource('foto', 'FotoController',
					array('names' => array('create' => 'foto.build')));

#### Resource Controller'lere Başka Rotalar Eklenmesi

Bir bir resource controller'e ön tanımlı resource rotaları dışında başka rotalar eklemeniz gerekli bir hale gelirse, bu rotaları `Route::resource` metodunu çağırmadan önce tanımlamalısınız:

	Route::get('foto/populer');
	Route::resource('foto', 'FotoController');

<a name="handling-missing-methods"></a>
## Eksik Olan Metodların İşlenmesi

Denetçide tanımlanmamış olan metodlara gelecek olan çağrıları işlemek için bir "hepsini yakala" metodu tanımlanabilir. Bu metodun isminin `missingMethod` olması gerekir ve istek için metod ve parametre dizisi alır:

#### Bir Hepsini Yakala Metodunun Tanımlanması

	public function missingMethod($parameters = array())
	{
		//
	}
