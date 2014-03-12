# Denetçiler (Controllers)

- [Temel Denetçiler](#basic-controllers)
- [Denetçi Filtreleri](#controller-filters)
- [TEDA-uyumlu (TEmsili Durum Aktarma uyumlu, RESTful) Denetçiler](#restful-controllers)
- [Kaynak (Resource) Denetçileri](#resource-controllers)
- [Eksik Olan Metodların Yönetilmesi](#handling-missing-methods)

<a name="basic-controllers"></a>
## Temel Denetçiler

Bütün rotalandırma mantığını, tüm rotaları tek tek `routes.php` dosyasında tanımlamak yerine, bu davranışlarını Denetçiler (Controllers) sınıflarını kullanarak organize edebilirsiniz. Denetçiler, ilişkin oldukları rotaların mantığını bir sınıfta gruplar. Aynı zamanda, daha ileri çerçeve (framework) özelliklerini kullanma avantajına sahiptirler, örneğin otomatik [dependency injection](/docs/ioc) (bağımlılık enjeksiyonu) gibi.

Denetçiler genelde `app/controllers` dizininde konumlandırılır ve `composer.json` dosyanızın sınıf haritası `classmap` seçeneğinde, varsayılan olarak bu dizin belirlenmiştir.

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

Eğer bir denetçinizi, dizin içerisinde yuvalandırarak (nest) veya PHP isim-alanları (namespaces) kullanarak organize etmek isterseniz, bu durumda rotayı tanımlarken, tam nitelendirilmiş (fully qualified) sınıf adını kullanınız:

	Route::get('falanca', 'Namespace\FalancaController@yontemAdi');

> **Note:** Since we're using [Composer](http://getcomposer.org) to auto-load our PHP classes, controllers may live anywhere on the file system, as long as composer knows how to load them. The controller directory does not enforce any folder structure for your application. Routing to controllers is entirely de-coupled from the file system.

Denetçi rotalarına isimler de verebilirsiniz:

	Route::get('falanca', array('uses' => 'FalancaController@yontemAdi',
		'as' => 'rotaAdi'));

Herhangi bir denetçi eylemine ait bir URL üretmek için, `URL::action` metodunu kullanabilirsiniz:

	$url = URL::action('FalancaController@yontemAdi');

	$url = action('FalancaController@yontemAdi');

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
		 * Yeni bir KullaniciController sureti (instance) oluştur. (new KullaniciController)
		 */
		public function __construct()
		{
			$this->beforeFilter('auth', array('except' => 'getGiris'));

			$this->beforeFilter('csrf', array('on' => 'post'));

			$this->afterFilter('log', array('only' =>
				array('falancaYontem', 'filancaYontem')));
		}

	}

Controller filtrelerini bir Closure kullanarak da belirtebilirsiniz:

	class KullaniciController extends BaseController {

		/**
		 * Yeni bir KullaniciController sureti (instance) oluştur. (new KullaniciController)
		 */
		public function __construct()
		{
			$this->beforeFilter(function()
			{
				//
			});
		}

	}

If you would like to use another method on the controller as a filter, you may use `@` syntax to define the filter:

	class UserController extends BaseController {

		/**
		 * Instantiate a new UserController instance.
		 */
		public function __construct()
		{
			$this->beforeFilter('@filterRequests');
		}

		/**
		 * Filter the incoming requests.
		 */
		public function filterRequests($route, $request)
		{
			//
		}

	}

<a name="restful-controllers"></a>
## TEDA-uyumlu (Temsili Durum Aktarma uyumlu, RESTful) Denetçiler

Laravel size, basit TEDA (REST) isimlendirme tüzüklerini (naming conventions) kullanarak, belirleyeceğiniz tek bir rota ile, denetçilerinizin içindeki her eylemi kullanabilme imkanını tanır. İlk olarak, (rota : denetçi) `Route::controller` metodu ile bu rotayı tanımlayınız:

**TEDA-uyumlu Bir Denetçi Oluşturulması**

	Route::controller('kullanicilar', 'KullaniciController');

`controller` (denetçi) metodu iki argüman alır. Birincisi denetçinin yöneteceği baz URI olup, ikincisi denetçinin sınıf ismidir. Akabinde sadece, isimlerine HTTP eyleminin ön ek olarak ekleneceği ve bunlara cevap verecek olan metodlarınızı denetçinize ilave ediniz:

	class KullaniciController extends BaseController {

		public function getIndex()
		{
			//
		}

		public function postProfile()
		{
			//
		}

	}

`index` (fihrist) metodları, denetçi tarafından yönetilmekte olan kök URI 'a cevap verir. Örneğimizde bu, `kullanicilar` dır.

Denetçinizdeki bir eylem metodunun ismi birden fazla kelimeden oluşuyorsa, bu eylem metoduna, URI da kelime aralarına tire işareti "-" eklenmiş şekilde yazarak erişebilirsiniz. Örneğin, 'KullaniciController' denetçimizdeki aşağıdaki şekilde isimlendirilmiş olan metod, `kullanici/yonetici-profili` URI 'na cevap verecektir.

	public function getYoneticiProfili() {}

<a name="resource-controllers"></a>
## Kaynak (Resource) Denetçileri

Kaynak denetçileri, kaynaklar etrafında TEDA-uyumlu denetçiler oluşturulmasını kolaylaştırır. Artisan KSA'daki (Artisan Komut Satırı Arayüzü) `controller:make` komutunu ve de `Route::resource` metodunu kullanmak sureti ile böyle bir denetçiyi çabucak oluşturabiliriz.

Denetçiyi komut satırını kullanarak oluşturmak için şu komutu kullanınız:

	php artisan controller:make FotoController

Bu denetçinin TEDA-uyumlu rotasını (routes.php) dosyasında kayıt ettiriniz:

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

Ve, rotasında da eylemlerin sadece bazılarını yönetmesini belirleyebilirsiniz:

	Route::resource('foto', 'FotoController',
			array('only' => array('index', 'show')));

	Route::resource('photo', 'PhotoController',
			array('except' => array('create', 'store', 'update', 'destroy')));

By default, all resource controller actions have a route name; however, you can override these names by passing a `names` array with your options:

	Route::resource('photo', 'PhotoController',
					array('names' => array('create' => 'photo.build'));

<a name="handling-missing-methods"></a>
## Eksik Olan Metodların Yönetilmesi

Denetçide tanımlanmamış olan metodlara gelecek olan çağrıları yönetmek için bir "hepsini-yakala" metodu tanımlanabilir. Bu metodun isminin `missingMethod` (eksik metod) olması gerekir ve tek argümanı olarak gelen isteğin (request) parametrelerini  alır:

**Bir Hepsini-Yakala Metodunun Tanımlanması**

	public function missingMethod($parameters)
	{
		//
	}