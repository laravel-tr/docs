# Laravel Hızlı Başlangıç

- [Kurulum](#installation)
- [Local Geliştirme Ortamı](#local-development-environment)
- [Rotalandırma (Routing)](#routing)
- [Bir View Oluşturma](#creating-a-view)
- [Bir Migrasyon Oluşturma](#creating-a-migration)
- [Eloquent ORM](#eloquent-orm)
- [Veri Gösterme](#displaying-data)
- [Uygulamanızın Yayımlanması](#deploying-your-application)

<a name="installation"></a>
## Kurulum

### Laravel Installer Aracılığıyla

İlk olarak, [Laravel installer PHAR arşivini indirin](http://laravel.com/laravel.phar). Kolaylık açısından ismini `laravel` olarak değiştirin ve `/usr/local/bin` yoluna taşıyın. Bir kere kurduktan sonra, `laravel new` komutu, istediğiniz klasöre yeni bir laravel kurulumunu yapacaktır. Örneğin, `laravel new blog` komutu, içinde tüm bağımlılıkları yüklenmiş yeni bir laravel kurulumu barındıran `blog` klasörünü oluşturacaktır. Bu yolla kurulum yapmak Composer ile yapmaktan çok daha hızlıdır.

### Composer Aracılığıyla

Laravel framework kurulumu ve bağımlılık yönetimi için [Composer](http://getcomposer.org) kullanır. Şayet sizde yoksa [Composer yüklemesi](http://getcomposer.org/doc/00-intro.md) ile başlayın.

Artık terminalinizden aşağıdaki komutu vermek suretiyle Laravel yükleyebilirsiniz:

	composer create-project laravel/laravel sizin-projenizin-ismi --prefer-dist

Bu komut sizin geçerli dizininiz içerisindeki yeni bir `sizin-projenizin-ismi` klasörüne Laravel'in yepyeni bir kopyasını indirecek ve yükleyecektir.

Eğer isterseniz, alternatif olarak [Github'daki Laravel ambarının](https://github.com/laravel/laravel/archive/master.zip) bir kopyasını elle indirebilirsiniz. Sonra da elle oluşturduğunuz proje dizininizin kökünde `composer install` komutunu çalıştırın. Bu komut, frameworkün bağımlılıklarını indirecek ve yükleyecektir.

### İzinler

Laravel yüklenmesinden sonra, `app/storage` dizinlerine web sunucu yazma izinleri hakları tanımanız gerekebilir. Yapılandırma konusunda daha fazla ayrıntılar için [Kurulum](/docs/installation) dokümantasyonuna bakınız.

### Laravel'in Hizmete Sokulması

Tipik olarak, Laravel uygulamalarınızı sunmak için Apache veya Nginx gibi bir web sunucusu kullanabilirsiniz. Eğer sizde PHP 5.4+ var ve PHP'nin yerleşik geliştirme sunucusunu kullanmak isterseniz, `serve` Artisan komutunu kullanabilirsiniz:

	php artisan serve

<a name="directories"></a>
### Dizin Yapısı

Frameworkün yüklenmesinden sonra, dizin yapısıyla aşina olmak için projenize bir göz atın. Projenizdeki `app` dizini `views`, `controllers` ve `models` gibi klasörler içerir. Uygulamanızın kodlarının çoğu bu dizin içindeki bir yerlerde ikamet eder. Ayrıca, `app/config` dizinini de inceleyip sizin için sunulmuş yapılandırma seçeneklerini keşfetmek isteyebilirsiniz.

<a name="local-development-environment"></a>
## Local Geliştirme Ortamı

Geçmişte, makineniz üzerinde lokal bir PHP geliştirme ortamı yapılandırılması bir başağrısıydı. PHP, gerekli uzantılar ve gerekli diğer bileşenlerin doğru sürümlerinin yüklenmesi zaman harcayıcı ve kafa karıştırıcıdır. Bunun yerine [Laravel Homestead](/docs/homestead) kullanmayı düşünün. Homestead Laravel ve [Vagrant](http://vagrantup.com) için tasarlanmış basit bir sanal makinedir. Homestead Vagrant kutusu güçlü ve sağlam PHP uygulamaları inşa etmeniz için gerekli yazılımların hepsiyle birlikte önceden paketlendiği için, sanallaştırılmış, izole bir geliştirme ortamını saniyeler içerisinde oluşturabilirsiniz. İşte Homestead'a dahil edilen araçlardan bazılarından oluşan bir liste:

- Nginx
- PHP 5.5
- MySQL
- Redis
- Memcached
- Beanstalk

"Sanallaştırılmış" biraz karışık geliyorsa da merak etmeyin, sancısızdır. Homestead'ın bağımlılıkları olan VirtualBox ve Vagrant'ın her ikisi de popüler tüm işletim sistemleri için grafiksel program yükleyicileri içermektedir. Başlamak için [Homestead dokümantasyonunu](/docs/homestead) kontrol ediniz.

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

Bu sözdizimi size ilk etapta biraz yabancı gelebilir. Bunun sebebi Laravel'in güçlü şablonlama sisteminin (Blade) kullanılmasıdır. Blade son derece hızlı çalışır çünkü sadece birkaç tane regex kodları kullanıp Blade sözdizimini PHP skriptlerine dönüştürür. Blade kullanıcılarına çok büyük fonksiyonellik sağlar. Şablon kalıtımı (Template inheritance) ve PHP'nin `if` ve `for` gibi temel kontrol yapılarını Blade üzerinden kullanabilirsiniz. Daha fazla bilgi için [Blade Dökümantasyonu'na](/docs/templates) bakınız.

Şimdi gerekli view dosyalarımızı oluşturduğumuza göre, oluşturduğumuz viewi `/kullanicilar` isteğine bir cevap olarak döndürelim. `Kullanıcılar!` stringini döndürmek yerine, bu kez oluşturduğumuz view dosyalarını döndüreceğiz:

	Route::get('kullanicilar', function()
	{
		return View::make('kullanicilar');
	});

Harika! Bir layoutu genişleten bir view oluşturdunuz. Bir sonraki bölümümümüzde Veritabanı Katmanı (Database Layer) üzerinde duracağız.

<a name="creating-a-migration"></a>
## Bir Migrasyon Oluşturma

Bir veritabanı tablosu oluşturmak için Laravel'in migrasyon (migration) özelliğini kullanacağız. Migrationlar çok kolay bir şekilde veritabanında değişiklikler yapmayı ve bunları takım arkadaşlarınızla paylaşmanızı sağlar.

Öncelikle bir veritabanı konfigürasyonu ayarlayalım. Tüm veritabanı konfigürasyonlarınızı `app/config/database.php` dosyası içerisinde değiştirebilirsiniz. Laravel öntanımlı olarak MySQL kullanmaya ayarlanmıştır, veritabanı konfigürasyonlarınızı  `app/config/database.php` dosyası içerisinde tanımlamanız gerekecektir. Dilerseniz `driver` değerini `sqlite` yapıp `app/database` dizininde bulunan SQLite veritabanını kullanabilirsiniz.

Sonra, bir migration oluşturmak için [Artisan CLI](/docs/artisan) kullanacağız. Projenizin ana dizinine gelerek, aşağıdaki kodu terminal üzerinde yazın:

	php artisan migrate:make create_users_table

Şimdi, oluşturulan migration dosyasını `app/database/migrations` dizininde bulun. Bu dosya 2 metoddan oluşmaktadır: `up` ve `down`. `up` metodunda, tablonuzdaki değişiklikleri yapmalısınız. `down` metodunda ise yaptığınız değişiklikleri geri almalısınız.

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

Eğer bir migrationu geri almak isterseniz `migrate:rollback` komutunu çalıştırmanız yeterli olacaktır. Şimdi bir veritabanı tablosu oluşturduğumuza göre, tablomuzdan veri çekmeyi öğrenerek devam edelim!

<a name="eloquent-orm"></a>
## Eloquent ORM

Laravel mükemmel bir ORM aracıyla beraber gelmektedir: Eloquent. Eğer daha önce Ruby on Rails frameworkü üzerinde çalıştıysanız Eloquent size çok tanıdık gelecektir, çünkü veritabanı işlemleri için ActiveRecord stilini kullanır.

Öncelikle, modeli tanımlayalım. Bir Eloquent modeli ilgili veritabanı tablosunu sorgulamak için kullanılabilir, aynı zamanda bu tablo içindeki belirli bir satırı temsil eder. Merak etmenize gerek yok, örnekleri görünce ne kadar kolay olduğunu anlayacaksınız! Model dosyaları `app/models` dizininde bulunmaktadır. Şimdi o dizinde bir `User.php` modeli oluşturalım:

	class User extends Eloquent {}

Lütfen dikkat edin, herhangi bir veritabanı tablosu belirtmedik. Eloquent'in içerisinde çeşitli gelenekler vardır, bunlardan birisi modelin veritabanı tablosu olarak model adının çoğul halini kullanmaktır. Kullanışlı, değil mi?

Tercih ettiğiniz veritabanı yönetim aracını kullanarak, `users` tablosuna birkaç satır ekleyin. Ondan sonra Eloquent'i kullanarak o tablodan bazı verileri çekip view dosyamıza göndereceğiz.

Şimdi `/kullanicilar` rotamızda değişiklik yapalım ve şuna benzer bir hale getirelim:

	Route::get('kullanicilar', function()
	{
		$users = User::all(); //Users tablosundaki tüm verileri $users değişkenine atar

		return View::make('kullanicilar')->with('users', $users);
	});

Şimdi bu scripti biraz inceleyelim. Öncelikle, `User` modelindeki `all` metodu `users` tablosundaki tüm verileri çekecektir. Daha sonra bu veriler `with` metodu kullanılarak view dosyasına gönderilir. `with` metodu bir anahtar ve bir değer almaktadır, böylece gönderilen veriyi view dosyası tanıyabilir.

Harika. Artık kullanıcıları view dosyamızda göstermeye hazırız!

<a name="displaying-data"></a>
## Veri Gösterme

Şimdi view'imizde `users` değişkenini kullanılabilir yaptığımıza göre, onu şuna benzer bir şekilde gösterebiliriz:

	@extends('layout')

	@section('content')
		@foreach($users as $user)
			<p>{{ $user->name }}</p>
		@endforeach
	@stop

`echo` ifadesinin nerede olduğunu merak ediyor olabilirsiniz. Blade kullanırken, küme parantezi arasına yazılan değişkenler aynı `echo` ifadesindeki gibi ekrana bastırılır. Şimdi `/kullanicilar` adresine girip veritabanınızda kayıtlı olan tüm kullanıcıların listesinin ekrana bastırıldığını görebilirsiniz.

Bu sadece bir başlangıç. Bu derste Laravel'in en temel konularını gördünüz, ancak daha göreceğiniz birçok heyecan verici özellikler var! Dökümantasyonu okumaya devam edin ve Laravel içerisinde gelen birçok farklı özellik hakkında daha fazla bilgiye sahip olun. Örneğin [Eloquent](/docs/eloquent) ve [Blade](/docs/templates). Belki de sizin ilginizi [Queues](/docs/queues) ve [Unit Testing](/docs/testing) çekiyordur? Ya da [IoC Container](/docs/ioc) kullanarak uygulamanızın mimarisini güçlendirmek istiyorsunuzdur? Seçim sizin!

<a name="deploying-your-application"></a>
## Uygulamanızın Yayımlanması

Laravel'in amaçlarından biri de PHP uygulama geliştirmeyi indirmekten yayımlamaya kadar keyifli bir hale getirmektir ve [Laravel Forge](https://forge.laravel.com) Laravel uygulamalarınızı süper hızlı sunucular üzerinde yayımlamak için basit bir yol sağlar. Forge DigitalOcean, Linode, Rackspace ve Amazon EC2 üzerinde sunucuları yapılandırabilir ve karşılayabilir. Tıpkı Homestead gibi, gerekli en son araçlar dahil edilmiştir: Nginx, PHP 5.5, MySQL, Postgres, Redis, Memcached ve başkaları. Hatta, Forge "Quick Deploy" özelliğiyle değişikliklerinizi Github veya Bitbucket'e push ettiğiniz her seferinde kodunuzu yayımlamış olursunuz!

Forge bunlar yanında kuyruk işçileri, SSL, Cron işleri, sub-domainler ve daha birçok şeyi yapılandırmanıza yardım edebilir. Daha fazla bilgi için [Forge websitesini](https://forge.laravel.com) ziyaret ediniz.
