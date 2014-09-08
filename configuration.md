# Yapılandırma

- [Giriş](#introduction)
- [Ortam Yapılandırması](#environment-configuration)
- [Sağlayıcı Yapılandırması](#provider-configuration)
- [Hassas Yapılandırmaları Korumak](#protecting-sensitive-configuration)
- [Bakım Modu](#maintenance-mode)

<a name="introduction"></a>
## Giriş

Laravel'in tüm yapılandırma dosyaları `app/config` dizini içindedir. Tüm dosyalardaki yapılandırma seçenekleri açıklanmıştır, dosyalara göz gezdirip size sunulan seçeneklere göz atabilirsiniz.

Bazen yapılandırma değerlerine run-time (çalışma anı) esnasında erişmeniz gerekir. Bunu `Config` sınıfını kullanarak yapabilirsiniz:

**Bir Yapılandırma Değerine Erişmek**

	Config::get('app.timezone');

Eğer yapılandırma değeri bulunamazsa dönecek değeri ise, ikinci bir parametreyle belirleyebilirsiniz:

	$timezone = Config::get('app.timezone', 'UTC');

**Bir Yapılandırma Değeri Ayarlamak**

Lütfen dikkat edin, "nokta" şeklindeki kullanım biçimi tüm yapılandırma dosyalarına erişmenizi sağlar. Dilerseniz yapılandırma değerlerini run-time (çalışma anı) esnasında da ayarlayabilirsiniz:

	Config::set('database.default', 'sqlite');

Çalışma zamanında ayarlanan yapılandırma değerleri sadece güncel istek süresince ayarlanırlar ve sonraki isteklere aktarılmayacaklardır.

<a name="environment-configuration"></a>
## Ortam Yapılandırması

Uygulamanın çalışma ortamına göre farklı yapılandırma değerlerine sahip olmak çoğu zaman iyidir. Örneğin, kişisel bilgisayarınızda, sunucudan farklı bir önbellekleme uygulaması kullanmak isteyebilirsiniz. Bunu ortam tabanlı yapılandırmalar oluşturarak sağlayabilirsiniz.

Bunu yapmak çok basit! `config` dizini içerisinde, ortam isminizi kullandığınız (örneğin `local`) bir dizin daha oluşturun. Şimdi, belirttiğiniz ortam için üzerine yazmak istediğiniz yapılandırma dosyalarınızı ve seçeneklerinizi geçirin. Örneğin, önbellekleme yapılandırmasının üzerine yazmak için, `app/config/local` dizini içerisinde `cache.php` dosyası oluşturmanız gerekir. Oluşturduğunuz dosyanın içerisine şunları yazın:

	<?php

	return array(

		'driver' => 'file',

	);

> **Not:** 'testing' adını ortam ismi olarak kullanmayın. Bu isim Unit Testing amacıyla rezerve edilmiştir.

Dikkat ederseniz, bu dosyada _bütün_ değerleri yazmanıza gerek yok. Sadece üzerine yazmak istediklerinizi eklemeniz yeterli. Geri kalan değerler, öntanımlı yapılandırma değerlerinden alınacaktır.

Şimdi yapmamız gereken Laravel'e hangi ortamda çalıştığını belirtmek. Öntanımlı ortam daima `production` ortamıdır. Ancak ana dizindeki `bootstrap/start.php` dosyası içerisine eklemeler yaparak farklı ortamlar oluşturmak mümkündür. Bu dosya içerisinde `$app->detectEnvironment` adında bir tanım bulacaksınız. Bu metoda eklenen bir parametre ile Laravel'e hangi ortamda çalıştığını belirtebilirsiniz. Hatta ihtiyacınız olursa, diğer ortam ve makine isimlerini de dizi olarak ekleyebilirsiniz:

    <?php

    $env = $app->detectEnvironment(array(

        'local' => array('bilgisayarınızın-ismi'),

    ));

Bu örnekte, 'local' ortamın ismi ve 'bilgisayarınızın-ismi' sunucunuzun makine ismidir. Linux ve Mac işletim sistemlerinde, terminalde `hostname` komutunu çalıştırarak sunucunuzun makine ismini öğrenebilirsiniz.

Dilerseniz, `detectEnvironment` methoduna `Closure` ekleyip ortam algılama özelliğini kendiniz de yazabilirsiniz:

	$env = $app->detectEnvironment(function()
	{
		return $_SERVER['MY_LARAVEL_ENV'];
	});

#### Şu anki Uygulama Ortamına Erişmek

Şu anki uygulama ortamına `environment` metoduyla erişebilirsiniz:

	$environment = App::environment();

Ayrıca `environment` metoduna bir veya daha fazla parametre girerek, ortamın girilen parametrelerden biriyle eşleşip eşleşmediğini kontrol edebilirsiniz:

	if (App::environment('local'))
	{
		// Ortam 'local'
	}

	if (App::environment('local', 'staging'))
	{
		// Ortam 'local' veya 'staging'
	}

<a name="provider-configuration"></a>
### Sağlayıcı Yapılandırması

Ortam yapılandırması kullanırken ana `app` yapılandırma dosyanıza ortam [hizmet sağlayıcıları](/docs/ioc#service-providers) eklemek isteyebilirsiniz. Denediğinizde, ortama ait sağlayıcıların, ana `app` yapılandırmasındaki sağlayıcıları geçersiz kıldığını fark edeceksiniz. Ortama ait sağlayıcıların, diğerlerini geçersiz kılmak yerine onlara eklenmesini sağlamak için ortam yapılandırma dosyalarınızda `append_config` yardımcı fonksiyonunu kullanmanız gerekir:

	'providers' => append_config(array(
		'LocalOnlyServiceProvider',
	))

<a name="protecting-sensitive-configuration"></a>
## Hassas Yapılandırmaları Korumak

"Gerçek" uygulamalarda, hassas yapılandırmaları yapılandırma dosyalarında tutmamanız önerilir. Veritabanı şifreleri, Stripe API anahtarları ve kriptolama anahtarları mümkün olduğunca yapılandırma dosyalarının dışında tutulmalıdır. O zaman nerede tutacağız bu bilgileri? Neyse ki, Laravel bu tip bilgilerin korunabilmesi için "nokta" yapılandırma dosyaları adında oldukça basit bir çözüm sağlıyor.

Öncelikle uygulamanızı 'local' ortamınızı tanıyacak şekilde [yapılandır](/docs/configuration#environment-configuration)malısınız. Sonra projenizin kök dizininde, yani composer.json dosyanızın bulunduğu dizinde `.env.local.php` dosyanızı oluşturmalısınız. Bu dosya tıpkı diğer Laravel yapılandırma dosyaları gibi anahtar-değer çiftlerine sahip bir dizi döndürmelidir.

	<?php

	return array(

		'TEST_STRIPE_KEY' => 'super-secret-sauce',

	);

Bu dosyadaki tüm anahtar-değer çiftleri PHP'nin `$_ENV` ve `$_SERVER` "süperküresel" değişkenlerinde erişilebilir olacaktır. Artık yapılandırma dosyalarınızda bu değişkenlere erişebilirsiniz:

	'key' => $_ENV['TEST_STRIPE_KEY']

`.env.local.php` dosyasını `.gitignore` dosyasına eklemeyi unutmayın. Bu, dosyanın kaynak kontrol sistemine (Git) girmesini ve ortamınızın kişisel bilgilerine erişilmesini engeller.

Şimdi bir de projenizi yayınladığınız sunucuda `.env.php` dosyası oluşturup gerekli yapılandırmaları aynı formatta girin. Aynı `.env.local.php` dosyası gibi, bu `.env.php` üretim ortamı dosyası da hiçbir zaman kaynak kontrolde bulunmamalıdır.

> **Not:** Her bir ortam için gerekli yapılandırma dosyasını oluşturabilirsiniz. Örneğin, `development` ortamında çalışan proje, eğer varsa `.env.development.php` dosyasını sisteme dahil edecektir. Bununla birlikte, `production` ortamı her zaman için `.env.php` dosyasını kullanır.

<a name="maintenance-mode"></a>
## Bakım Modu

Uygulamanız bakım modundayken, her istek için standart bir view gösterilir. Böylece uygulamanız güncellenirken, bir süreliğine uygulamayı "çalışmaz hale" getirebilirsiniz. Halihazırda `App::down` methoduna yapılan bir istek `app/start/global.php` dosyasında bulunmaktadır. Uygulamanız bakım modunda olduğunda, kullanıcılara bu metoddan dönen yanıt gönderilecektir.

Bakım modunu açmak için `down` komutunu Artisan üzerinde çalıştırın:

	php artisan down

Bakım modunu kapatmak içinse, `up` komutunu çalıştırabilirsiniz:

	php artisan up

Uygulamanız bakım modundayken kullanıcılara özel bir view göstermek için `app/start/global.php` dosyası içerisindeki `down` methodunu dilediğiniz gibi değiştirebilirsiniz:

	App::down(function()
	{
		return Response::view('bakim_sayfasi', array(), 503);
	})

Eğer `down` metoduna girilen anonim fonksiyon (Closure) `NULL` değeri döndürürse, bakım modu o istek için görmezden gelinecektir.

### Bakım Modu ve Kuyruklar

Uygulamanız bakım modunda iken, hiçbir [kuyruk işlemi](/docs/queues) uygulanmaz. Tüm işlemler, uygulama bakım modundan çıktığında normal bir şekilde devam eder.
