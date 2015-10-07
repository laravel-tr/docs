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

Resource controllers make it painless to build RESTful controllers around resources. For example, you may wish to create a controller that handles HTTP requests regarding "photos" stored by your application. Using the `make:controller` Artisan command, we can quickly create such a controller:

	php artisan make:controller PhotoController

Next, we register a resourceful route to the controller:

	Route::resource('photo', 'PhotoController');

This single route declaration creates multiple routes to handle a variety of RESTful actions on the photo resource. Likewise, the generated controller will already have methods stubbed for each of these actions, including notes informing you which URIs and verbs they handle.

#### Actions Handled By Resource Controller

Verb      | Path                        | Action       | Route Name
----------|-----------------------------|--------------|---------------------
GET       | /resource                   | index        | resource.index
GET       | /resource/create            | create       | resource.create
POST      | /resource                   | store        | resource.store
GET       | /resource/{resource}        | show         | resource.show
GET       | /resource/{resource}/edit   | edit         | resource.edit
PUT/PATCH | /resource/{resource}        | update       | resource.update
DELETE    | /resource/{resource}        | destroy      | resource.destroy

#### Customizing Resource Routes

Additionally, you may specify only a subset of actions to handle on the route:

	Route::resource('photo', 'PhotoController',
					['only' => ['index', 'show']]);

	Route::resource('photo', 'PhotoController',
					['except' => ['create', 'store', 'update', 'destroy']]);

By default, all resource controller actions have a route name; however, you can override these names by passing a `names` array with your options:

	Route::resource('photo', 'PhotoController',
					['names' => ['create' => 'photo.build']]);

#### Handling Nested Resource Controllers

To "nest" resource controllers, use "dot" notation in your route declaration:

	Route::resource('photos.comments', 'PhotoCommentController');

This route will register a "nested" resource that may be accessed with URLs like the following: `photos/{photos}/comments/{comments}`.

	class PhotoCommentController extends Controller {

		/**
		 * Show the specified photo comment.
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

#### Adding Additional Routes To Resource Controllers

If it becomes necessary to add additional routes to a resource controller beyond the default resource routes, you should define those routes before your call to `Route::resource`:

	Route::get('photos/popular');

	Route::resource('photos', 'PhotoController');

<a name="dependency-injection-and-controllers"></a>
## Enjeksiyon ve Denetçi Bağımlılığı

#### Oluşturucu Enjeksiyon

The Laravel [service container](/docs/master/container) is used to resolve all Laravel controllers. As a result, you are able to type-hint any dependencies your controller may need in its constructor:

	<?php namespace App\Http\Controllers;

	use Illuminate\Routing\Controller;
	use App\Repositories\UserRepository;

	class UserController extends Controller {

		/**
		 * The user repository instance.
		 */
		protected $users;

		/**
		 * Create a new controller instance.
		 *
		 * @param  UserRepository  $users
		 * @return void
		 */
		public function __construct(UserRepository $users)
		{
			$this->users = $users;
		}

	}

Of course, you may also type-hint any [Laravel contract](/docs/master/contracts). If the container can resolve it, you can type-hint it.

#### Method Injection

In addition to constructor injection, you may also type-hint dependencies on your controller's methods. For example, let's type-hint the `Request` instance on one of our methods:

	<?php namespace App\Http\Controllers;

	use Illuminate\Http\Request;
	use Illuminate\Routing\Controller;

	class UserController extends Controller {

		/**
		 * Store a new user.
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

If your controller method is also expecting input from a route parameter, simply list your route arguments after your other dependencies:

	<?php namespace App\Http\Controllers;

	use Illuminate\Http\Request;
	use Illuminate\Routing\Controller;

	class UserController extends Controller {

		/**
		 * Store a new user.
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

> **Note:** Method injection is fully compatible with [model binding](/docs/master/routing#route-model-binding). The container will intelligently determine which arguments are model bound and which arguments should be injected.

<a name="route-caching"></a>
## Rota Önbellekleme (Route Caching)

If your application is exclusively using controller routes, you may take advantage of Laravel's route cache. Using the route cache will drastically decrease the amount of time it take to register all of your application's routes. In some cases, your route registration may even be up to 100x faster! To generate a route cache, just execute the `route:cache` Artisan command:

	php artisan route:cache

That's all there is to it! Your cached routes file will now be used instead of your `app/Http/routes.php` file. Remember, if you add any new routes you will need to generate a fresh route cache. Because of this, you may wish to only run the `route:cache` command during your project's deployment.

To remove the cached routes file without generating a new cache, use the `route:clear` command:

	php artisan route:clear
