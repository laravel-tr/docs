# HTTP Controllers

- [Giriş](#introduction)
- [Temel Denetçiler (Controllers)](#basic-controllers)
- [Denetçi Ara Katmanlar (Middleware)](#controller-middleware)
- [RESTful Kaynak Denetçiler](#restful-resource-controllers)
- [Enjeksiyon ve Denetçi Bağımlılığı](#dependency-injection-and-controllers)
- [Rota Önbellekleme (Route Caching)](#route-caching)

<a name="introduction"></a>
## Giriş

Tüm istek işleme mantığı sadece 'routes.php' dosyasında tanımlamak yerine, bu işlemi Denetçi(Controller) sınıfları kullanarak düzenlemek isteyebilirsiniz. Denetçileri(Controllers) ilgili HTTP istek işleme mantığını bir sınıfta gruplandırabilirsiniz. Denetçiler(Controllers) genellikle `app/Http/Controllers` dizininde depolanır.

<a name="basic-controllers"></a>
## Temel Denetçiler (Controllers)

 Burada bir temel Denetçi sınıfı örneği verilmiştir.

	<?php namespace App\Http\Controllers;

	use App\Http\Controllers\Controller;

	class UserController extends Controller {

		/**
		 * Verilen kullanıcı profilini göster.
		 *
		 * @param  int  $id
		 * @return Response
		 */
		public function showProfile($id)
		{
			return view('user.profile', ['user' => User::findOrFail($id)]);
		}

	}

 Denetçi hareketini aşağıdaki gibi yönlendirebiliriz.

	Route::get('user/{id}', 'UserController@showProfile');

> **Not:** Tüm Denetçiler ana denetçi sınıfından genişletilmelidir.

#### Denetçiler(Controllers) & Ad Alanları(Namespaces)

Unutlmaması gereken çok önemli nokta tam denetçi ad alanını belirtmeye ihtiyaç duymadığımız, sadece sonra gelen sınıf adının kısmı `App\Http\Controllers`dan sonra ad alanı "root(kök)" gelmesi. Varsayılan olarak, 'RouteServiceProvider' 'routes.php' dosyasının kök denetçisi ad alanı içeren bir rota grubu içinde yüklenecektir.

İç içe veya denetçilerinizi PHP ad alanları daha tanımlı 'App\Http\Controllers' içinde kullanarak düzenlemek isterseniz, sadece 'App\Http\Controllers' kök ad alanına göre sınıf adı kullanın. Yani,`App\Http\Controllers\Photos\AdminController` tam denetçi sınıfınız ise rota kaydı aşağıdaki gibi olur.

	Route::get('foo', 'Photos\AdminController@method');

#### Adlandırma Denetçisi Rotaları

 Kapatma rotaları gibi, denetçi yolları üzerinde adları belirtebilirsiniz:

	Route::get('foo', ['uses' => 'FooController@method', 'as' => 'name']);

#### Denetçi eylemleri için URL'ler

Bir denetçi eylemi için bir URL oluşturmak için 'action' yardımcı yöntemini(method) kullanın:

	$url = action('App\Http\Controllers\FooController@method');

Denetçi ad sınıf adı alanınıza göre sadece bir bölümünü kullanırken bir denetçi eylemi için bir URL oluşturmak istiyorsanız, URL oluşturu ile kök(root) denetçisi ad alanları kaydı:

	URL::setRootControllerNamespace('App\Http\Controllers');

	$url = action('FooController@method');

Denetçi eylem adına erişmek için 'CurrentRouteAction' yöntemini kullanarak çalıştırabilirsiniz:

	$action = Route::currentRouteAction();

<a name="controller-middleware"></a>
## Denetçi Ara Katmanlar (Middleware)

[Middleware](/docs/master/middleware) denetçi rotaları üzerinde aşağıdaki gibi belirtilebilir:

	Route::get('profile', [
		'middleware' => 'auth',
		'uses' => 'UserController@showProfile'
	]);

Ayrıca, denetçinizin(Controller) yapıcısının(constructor) içindeki katmanı belirtebilirsiniz:

	class UserController extends Controller {

		/**
		 *  Yeni bir UserController örneği örneğini oluşturma
		 */
		public function __construct()
		{
			$this->middleware('auth');

			$this->middleware('log', ['only' => ['fooAction', 'barAction']]);

			$this->middleware('subscribed', ['except' => ['fooAction', 'barAction']]);
		}

	}

<a name="restful-resource-controllers"></a>
## RESTful Kaynak Denetçiler

Kaynak denetçileri kaynaklar etrafında RESTful denetçilerini hatasız(sancısız) oluşturmak için yapılır. Örneğin, uygulamanız tarafından saklanan(depolanan) "fotoğrafları(photos)" HTTP isteklerinin işleme denetçisini oluşturmak isteyebilirsiniz. `make:controller` Artisan komutu kullanarak, hızlı bir şekilde bu tür bir denetçi oluşturabiliriz:

	php artisan make:controller PhotoController

Ardından, yetenekli bir rota denetçisi kayıt ediyoruz:

	Route::resource('photo', 'PhotoController');

Bu tek rota ifadesi fotoğraf kaynaktağında RESTful çeşitli eylemler işlemek için birden fazla rota oluşturur. Aynı şekilde, oluşturulan denetçi zaten bu eylemlerin her biri için temizleyici yöntemler olacaktır, URI'lar ve fiiller hakkında bilgi veren notlar da dahil edilir.

#### Kaynak Denetçisi Tarafından İşlenen Eylemler

Verb      | Path                        | Action       | Route Name
----------|-----------------------------|--------------|---------------------
GET       | /resource                   | index        | resource.index
GET       | /resource/create            | create       | resource.create
POST      | /resource                   | store        | resource.store
GET       | /resource/{resource}        | show         | resource.show
GET       | /resource/{resource}/edit   | edit         | resource.edit
PUT/PATCH | /resource/{resource}        | update       | resource.update
DELETE    | /resource/{resource}        | destroy      | resource.destroy

#### Kaynak Rotaları Özelleştirme

Ayrıca, sadece bir alt kümesini rota üzerinde işlemek için belirtebilirsiniz.

	Route::resource('photo', 'PhotoController',
					['only' => ['index', 'show']]);

	Route::resource('photo', 'PhotoController',
					['except' => ['create', 'store', 'update', 'destroy']]);

Varsayılan olarak, tüm kaynak denetçi eylemlerinin bir rota adı var; ancak, seçeneklerinizi 'names' dizisiyle geçirerek bu adları geçersiz kılabilirsiniz.

	Route::resource('photo', 'PhotoController',
					['names' => ['create' => 'photo.build']]);

#### İç içe Kaynak İşleme Denetçileri

"nest" kaynak denetçileri için, rotanızın ifadesi için "dot" gösterimini kullanın:

	Route::resource('photos.comments', 'PhotoCommentController');

Bu rota erişilebilir bir "nested(iç içe)" kaynak URL ile aşağıdaki gibi kaydeder:
`photos/{photos}/comments/{comments}`.

	class PhotoCommentController extends Controller {

		/**
		 * Belirtilen fotoğraf yorumu göster.
		 *
		 * @param  int  $photoId
		 * @param  int  $commentId
		 * @return Response
		 */
		public function show($photoId, $commentId)
		{
			//
		}

	}

#### Kaynak Denetçiler için Ek Rotalar Ekleme

Bir kaynak denetçisi varsayılan kaynak rotaları haricinde ek rotalar eklemek gerekli olursa, `Route::resource` 'ı çağırmadan önce bu rotaları tanımlamanız gerekir:

	Route::get('photos/popular');

	Route::resource('photos', 'PhotoController');

<a name="dependency-injection-and-controllers"></a>
## Enjeksiyon ve Denetçi Bağımlılığı

#### Oluşturucu Enjeksiyon

Laravel [service container](/docs/master/container) tüm Laravel denetçilerini çözümlemek için kullanılır. Sonuç olarak, tip-ipucu denetçinizin herhangi bir bağımlılığı için kurucu içinde gerekebilir:

	<?php namespace App\Http\Controllers;

	use Illuminate\Routing\Controller;
	use App\Repositories\UserRepository;

	class UserController extends Controller {

		/**
		 * Kullanıcı deposu örneği.
		 */
		protected $users;

		/**
		 * Yeni bir denetçi örneği oluşturun.
		 *
		 * @param  UserRepository  $users
		 * @return void
		 */
		public function __construct(UserRepository $users)
		{
			$this->users = $users;
		}

	}

Tabii ki, aynı zamanda herhangi bir [Laravel contract](/docs/master/contracts) olabilir. Eğer kapsayıcı(container) çözümleyebilirse, bu tip-ipucu olabilir.

#### Metod Injection(Enjeksiyon)

 Oluşturucu enjeksiyon ek olarak, denetçinizin metodlarının tip-ipucu ayrıca bağımlılıkları olabilir. Örneğin, hadi  metodlarımızdan birini `Request` örneğini tip-ipucu alalım:

	<?php namespace App\Http\Controllers;

	use Illuminate\Http\Request;
	use Illuminate\Routing\Controller;

	class UserController extends Controller {

		/**
		 * Yeni bir kullanıcı deposu.
		 *
		 * @param  Request  $request
		 * @return Response
		 */
		public function store(Request $request)
		{
			$name = $request->input('name');

			//
		}

	}

Eğer denetçi metodunuzla da bir rota parametresi girdi bekliyor ise diğer bağımlılıklar sonra, basitçe rota argümanlar listesi:

	<?php namespace App\Http\Controllers;

	use Illuminate\Http\Request;
	use Illuminate\Routing\Controller;

	class UserController extends Controller {

		/**
		 * Yeni bir kullanıcı deposu.
		 *
		 * @param  Request  $request
		 * @param  int  $id
		 * @return Response
		 */
		public function update(Request $request, $id)
		{
			//
		}

	}

> **Not:** Metod enjeksiyonu ile [model binding](/docs/master/routing#route-model-binding) tamamen uyumludur. Kapsayıcı bağlı argüman modellerini ve enjekte edilecek argümanları belirleyebilir.

<a name="route-caching"></a>
## Rota Önbellekleme (Route Caching)

 Uygulamanız sadece denetçi rotalar kullanıyorsa, Laravel'ın rota önbelleğinden yararlanmak avantajlı olabilir. Rota önbelleğini kullanarak uygulamanızın rota kayıt süresi büyük ölçüde azalacaktır. Bazı durumlarda, rotanızın kaydı bile 100x daha hızlı olabilir! Bir rota belleği oluşturmak için, sadece `route:cache` Artisan komutunu çalıştırın:

	php artisan route:cache

İşte hepsi bu kadar! Önbelleğe alınmış rotalar artık `app/Http/routes.php` dosyası yerine kullanılacak. Unutmayın, eğer herhangi yeni rota eklerseniz, yeni bir rota önbeleği oluşturmanız gerekir. Bu nedenle, projenizin dağıtımı sırasında sadece `route:cache` komutunu çalıştırabilirsiniz.

Yeni bir önbellek oluşturmadan önce önbelleğe alınan rotaları kaldırmak için, `route:clear` komutunu kullanın:

	php artisan route:clear
