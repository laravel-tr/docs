# Laravel Hızlı Başlangıç

- [Kurulum](#installation)
- [Rotalandırma (Routing)](#routing)
- [Bir View Oluşturma](#creating-a-view)
- [Bir Migrasyon Oluşturma](#creating-a-migration)
- [Eloquent ORM](#eloquent-orm)
- [Veri Gösterme](#displaying-data)

<a name="installation"></a>
## Kurulum

### Via Laravel Installer

First, download the [Laravel installer PHAR archive](http://laravel.com/laravel.phar). For convenience, rename the file to `laravel` and move it to `/usr/local/bin`. Once installed, the simple `laravel new` command will create a fresh Laravel installation in the directory you specify. For instance, `laravel new blog` would create a directory named `blog` containing a fresh Laravel installation with all dependencies installed. This method of installation is much faster than installing via Composer.

### Via Composer

The Laravel framework utilizes [Composer](http://getcomposer.org) for installation and dependency management. If you haven't already, start by [installing Composer](http://getcomposer.org/doc/00-intro.md).

Now you can install Laravel by issuing the following command from your terminal:

	composer create-project laravel/laravel your-project-name --prefer-dist

This command will download and install a fresh copy of Laravel in a new `your-project-name` folder within your current directory.

If you prefer, you can alternatively download a copy of the [Laravel repository from Github](https://github.com/laravel/laravel/archive/master.zip) manually. Next run the `composer install` command in the root of your manually created project directory. This command will download and install the framework's dependencies.

### Permissions

After installing Laravel, you may need to grant the web server write permissions to the `app/storage` directories. See the [Installation](/docs/installation) documentation for more details on configuration.

### Serving Laravel

Typically, you may use a web server such as Apache or Nginx to serve your Laravel applications. If you are on PHP 5.4+ and would like to use PHP's built-in development server, you may use the `serve` Artisan command:

	php artisan serve

<a name="directories"></a>
### Directory Structure

After installing the framework, take a glance around the project to familiarize yourself with the directory structure. The `app` directory contains folders such as `views`, `controllers`, and `models`. Most of your application's code will reside somewhere in this directory. You may also wish to explore the `app/config` directory and the configuration options that are available to you.

<a name="routing"></a>
## Rotalandırma (Routing)

Başlangıç olarak Laravel'de ilk Route'umuzu yazalım. Laravel'de rota oluşturmak için en basit yol bir closure (anonim fonksiyon) kullanmaktır. `app/routes.php` dosyasını açın ve aşağıdaki kod parçacığını sayfanın en altına yapıştırın:

	Route::get('kullanicilar', function()
	{
		return 'Kullanıcılar!';
	});

Şimdi, eğer web tarayıcınızda `/kullanicilar` adresine girerseniz, ekranda `Kullanıcılar!` yazısını görmüş olmanız gerekir. Eğer gördüyseniz çok iyi! İlk rotanızı başarıyla oluşturdunuz.

Route'lar ayrıca controller sınıflarına da bağlanabilir. Örneğin:

	Route::get('kullanicilar', 'KullaniciController@getIndex');

Bu Route Laravel'e şunu belirtiyor: `/kullanicilar`  rotasına yapılan bir istek `KullaniciController` sınıfındaki `getIndex` metodunu çağırmalıdır. Controller Routing hakkında daha fazla bilgi almak için [Controller Dökümantasyonu'na](/docs/controllers) bir göz atın.

<a name="creating-a-view"></a>
## Bir View Oluşturma

Şimdi basit bir view dosyası oluşturup, kullanıcı bilgilerini ekrana view üzerinden yazdıracağız. View dosyaları `app/views` dizini içerisinde bulunmakta olup projenizin HTML dosyalarını barındırır. Şimdi bu dizin içerisine 2 tane dosya oluşturacağız: `layout.blade.php` ve `kullanicilar.blade.php`. Önce `layout.blade.php` dosyamızı oluşturalım:

	<html>
		<body>
			<h1>Laravel Hızlı Başlangıç</h1>

			@yield('content')
		</body>
	</html>

Şimdiki adımda ise `kullanicilar.blade.php` view dosyasını oluşturalım:

	@extends('layout')

	@section('content')
		Kullanıcılar!
	@stop

Bu syntax size ilk etapta biraz yabancı gelebilir. Bunun sebebi Laravel'in güçlü templating sisteminin (Blade) kullanılmasıdır. Blade son derece hızlı çalışır çünkü sadece birkaç tane regex kodları kullanıp Blade syntaxını PHP scriptlerine dönüştürür. Blade kullanıcılarına çok büyük fonksiyonellik sağlar. Şablon kalıtımı (Template inheritance) ve PHP'nin `if` ve `for` gibi temel kontrol yapılarını Blade üzerinden kullanabilirsiniz. Daha fazla bilgi için [Blade Dökümantasyonu'na](/docs/templates) bakınız.

Şimdi gerekli view dosyalarımızı oluşturduğumuza göre, oluşturduğumuz viewi `/kullanicilar` isteğine bir cevap olarak döndürelim. `Kullanıcılar!` stringini döndürmek yerine, bu kez oluşturduğumuz view dosyalarını döndüreceğiz:

	Route::get('kullanicilar', function()
	{
		return View::make('kullanicilar');
	});

Harika! Bir layoutu genişleten bir view oluşturdunuz. Bir sonraki bölümümümüzde Veritabanı Katmanı (Database Layer) üzerinde duracağız.

<a name="creating-a-migration"></a>
## Bir Migrasyon Oluşturma

Bir veritabanı tablosu oluşturmak için Laravel'in (migrasyon) migration özelliğini kullanacağız. Migrationlar çok kolay bir şekilde veritabanında değişiklikler yapmayı ve bunları takım arkadaşlarınızla paylaşmanızı sağlar.

Öncelikle bir veritabanı konfigürasyonu ayarlayalım. Tüm veritabanı konfigürasyonlarınızı  `app/config/database.php` dosyası içerisinde değiştirebilirsiniz. Laravel öntanımlı olarak MySQL kullanmaya ayarlanmıştır, veritabanı konfigürasyonlarınızı  `app/config/database.php` dosyası içerisine tanımlamanız gerekecektir. Dilerseniz `driver` değerini `sqlite` yapıp `app/database` dizininde bulunan SQLite veritabanını kullanabilirsiniz.

Sonra, bir migration oluşturmak için [Artisan CLI](/docs/artisan) kullanacağız. Projenizin ana dizinine gelerek, aşağıdaki kodu terminal üzerinde yazın:

	php artisan migrate:make create_users_table

Şimdi, oluşturulan migration dosyasını `app/database/migrations` dizininde bulun. Bu dosya 2 methoddan oluşmaktadır: `up` ve `down`. `up` methodunda, tablonuzdaki değişiklikleri yapmalısınız. `down` methodunda ise yaptığınız değişiklikleri geri almalısınız.

Şuna benzeyen bir migration oluşturalım:

	public function up()
	{
		Schema::create('users', function($table)
		{
			$table->increments('id');
			$table->string('email')->unique();
			$table->string('name');
			$table->timestamps();
		});
	}

	public function down()
	{
		Schema::drop('users');
	}

Şimdi bu migrationu Artisan CLI üzerinde `migrate` komutu kullanarak çalıştıralım. Projenizin ana dizinine gelip aşağıdaki kodu çalıştırın:

	php artisan migrate

Eğer bir migrationu geri almak isterseniz `migrate:rollback` komutunu çalıştırmanız yeterli olacaktır. Şimdi bir veritabanı tablosu oluşturduğumza göre, tablomuzdan veri çekmeyi öğrenerek devam edelim!

<a name="eloquent-orm"></a>
## Eloquent ORM

Laravel mükemmel bir ORM aracıyla beraber gelmektedir: Eloquent. Eğer daha önce Ruby on Rails frameworkü üzerinde çalıştıysanız Eloquent size çok tanıdık gelecektir, çünkü veritabanı işlemleri için ActiveRecord stilini kullanır.

Öncelikle, modeli tanımlayalım. Bir Eloquent modeli ilgili veritabanı tablosunu sorgulamak için kullanılabilir, aynı zamanda bu tablo içindeki belirli bir satırı temsil eder. Merak etmenize gerek yok, örnekleri görünce ne kadar kolay olduğunu anlayacaksınız! Model dosyaları `app/models` dizininde bulunmaktadır. Şimdi o dizinde bir `User.php` modeli oluşturalım:

	class User extends Eloquent {}

Lütfen dikkat edin, herhangi bir veritabanı tablosu belirtmedik. Eloquent'in içerisinde birçok hüküm vardır, bunlardan birisi model adının çoğul yapısını veritabanı tablosu olarak kullandırmaktır. Kullanışlı, değil mi?

Tercih ettiğiniz veritabanı yönetim aracını kullanarak, `users` tablosuna birkaç satır ekleyin. Ondan sonra Eloquent'i kullanarak o tablodan bazı verileri çekip view dosyamıza göndereceğiz.

Şimdi `/users` routemizi editleyelim, ve şuna benzer bir hale getirelim:

	Route::get('users', function()
	{
		$users = User::all(); //Users tablosundaki tüm verileri $users değişkenine atar

		return View::make('users')->with('users', $users);
	});

Şimdi bu scripti biraz inceleyelim. Öncelikle, `User` modelindeki `all` methodu `users` tablosundaki tüm verileri çekecektir. Daha sonra bu veriler `with` methodu kullanılarak view dosyasına gönderilir. `with` methodu bir anahtar ve bir değer almaktadır, böylece gönderilen veriyi view dosyası tanıyabilir.

Harika. Artık kullanıcıları view dosyamızda göstermeye hazırız!

<a name="displaying-data"></a>
## Veri Gösterme

Artık `users` objesini view dosyamıza yönlendirdiğimiz için ekrana bastırabiliriz:

	@extends('layout')

	@section('content')
		@foreach($users as $user)
			<p>{{ $user->name }}</p>
		@endforeach
	@stop

`echo` ifadesinin nerede olduğunu merak ediyor olabilirsiniz. Blade kullanırken, küme parantezi arasına yazılan değişkenler aynı `echo` ifadesindeki gibi ekrana bastırılır. Şimdi `/users` adresine girip veritabanınızda kayıtlı olan tüm kullanıcıların listesinin ekrana bastırıldığını görebilirsiniz.

Bu sadece bir başlangıç. Bu derste Laravel'in en temel konularını gördünüz, ancak daha göreceğiniz birçok heyecan verici özellikler var! Dökümantasyonu okumaya devam edin ve Laravel içerisinde gelen birçok farklı özellik hakkında daha fazla bilgiye sahip olun. Örneğin [Eloquent](/docs/eloquent) ve [Blade](/docs/templates). Belkide sizin ilginizi [Queues](/docs/queues) ve [Unit Testing](/docs/testing) çekiyordur?. Yada [IoC Container](/docs/ioc) kullanarak uygulamanızın mimarisini güçlendirmek istiyorsunuzdur? Seçim sizin!