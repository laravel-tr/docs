# HTTP Middleware

- [Giriş](#introduction)
- [Middleware Tanımlama](#defining-middleware)
- [Middleware Kaydetme](#registering-middleware)

<a name="introduction"></a>
## Giriş

HTTP middleware uygulamalarınıza gelen HTTP isteklerini filtreleyebilmeniz için uygun bir mekanizma sağlar. Örneğin, Laravel içinde dahili olarak bulunan bir middleware kullanıcının giriş yapıp yapmadığını kontrol eder. Eğer kullanıcı giriş yapmamışsa, middleware kullanıcıyı giriş ekranına yönlendirecektir. Ancak, Eğer kullanıcı giriş yapmışsa, uygulamanızın içinde devam edebilmesi için middleware isteğe izin verecektir.

Tabi ki, kimlik kontrolü dışında çeşitli biçimlerdede middleware yazılabilir. CORS middleware uygulamanızın verdiği tüm cevaplara doğru HTTP Başlıklarını eklemekten sorumlu olabilir. Logging middleware uygulamanıza gelen tüm istekleri loglayabilir.

Laravel frameworkü içinde çeşitli middlewareler vardır, bunlardan bazıları bakım, kimlik kontrolü, CSRF koruması vb. Tüm bu middlewareler `app/Http/Middleware` klasörü altında bulunmaktadır.

<a name="defining-middleware"></a>
## Middleware Tanımlama

Yeni bir route filtresi yaratmak için, `make:middleware` Artisan komutunu kullanın:

	php artisan make:middleware YasliMiddleware

Bu komut `YasliMiddleware` sıfınıfı `app/Http/Middleware` klasörü altına yerleştirecektir. Bu middleware ile sadece `age` parametresi 200'den küçük olduğunda route'a erişebilir durumda olacağız. Ötetürlü, kullanıcıyı "home" URI'sine geri yönlendireceğiz.

	<?php namespace App\Http\Middleware;

	class EskiMiddleware {

		/**
		 * Run the request filter.
		 *
		 * @param  \Illuminate\Http\Request  $request
		 * @param  \Closure  $next
		 * @return mixed
		 */
		public function handle($request, Closure $next)
		{
			if ($request->input('age') < 200)
			{
				return redirect('home');
			}

			return $next($request);
		}

	}

Gördüğünüz gibi, Eğer `age` parametremiz küçükse `200` den, the middleware will return an HTTP redirect to the client; otherwise, the request will be passed further into the application. To pass the request deeper into the application (allowing the middleware to "pass"), simply call the `$next` callback with the `$request`.

En iyisi middleware'leri gözünüzde HTTP isteklerinin uygulamanıza ulaşmadan önce içinden geçmek zorunda olduğu bir dizi "katman" olarak düşünebilirsiniz.
Her bir katman gelen isteği sınayabilir hatta tamamen rededebilir.

<a name="registering-middleware"></a>
## Middleware Kaydetme

### Evrensel Middleware

Eğer middleware uygulamanıza gelen her bir HTTP isteği için çalışmasını istiyorsanız, basitçe `app/Http/Kernel.php` dosyasındaki `$middleware` özelliğine middleware sınıfınızı ekleyin.

### Middlewareleri Routelara Atama

Eğer middleware'inizi belirli bir route'a atamak isterseniz, `app/Http/Kernel.php` dosyasında bulunan middleware'inize bir anahtar kelime tanımlamalısınız. Standart olarak, bu sınıftaki `$middleware` özelliği Laravel içinde dahili gelen Middlewareleride barındırır. Kendinizinkini eklemek için, basitçe listenin sonuna ekleyin ve bir anahtar kelime tanımlayın.

Once the middleware has been defined in the HTTP kernel, you may use the `middleware` key in the route options array:

	Route::get('admin/profile', ['middleware' => 'auth', function()
	{
		//
	}]);
