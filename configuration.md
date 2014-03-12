# Yapılandırma

- [Giriş](#introduction)
- [Ortam Yapılandırması](#environment-configuration)
- [Provider Configuration](#provider-configuration)
- [Protecting Sensitive Configuration](#protecting-sensitive-configuration)
- [Bakım Modu](#maintenance-mode)

<a name="introduction"></a>
## Giriş

Laravel'in tüm yapılandırma dosyaları `app/config` dizini içindedir. Tüm dosyalardaki yapılandırma seçenekleri açıklanmıştır, dosyalara göz gezdirip size sunulan seçeneklere göz atabilirsiniz.

Bazen yapılandırma değerlerine run-time (çalışma anı) esnasında erişmeniz gerekir. Bunu `Config` sınıfını kullanarak yapabilirsiniz:

**Bir Yapılandırma Değerine Erişmek**

	Config::get('app.timezone');

Eğer yapılandırma değeri bulunamazsa dönecek değeri ise ikinci bir parametreyle belirleyebilirsiniz:

	$timezone = Config::get('app.timezone', 'UTC');

Lütfen dikkat edin, "nokta" şeklindeki kullanım biçimi tüm yapılandırma dosyalarına erişmenizi sağlar. Dilerseniz yapılandırma değerlerini run-time (çalışma anı) esnasında da ayarlayabilirsiniz:

**Bir Yapılandırma Değeri Ayarlamak**

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

Şimdi yapmamız gereken Laravel'e hangi ortamda çalıştığını belirtmek. Öntanımlı ortam daima `production` ortamıdır. Ancak ana dizindeki `bootstrap/start.php` dosya içerisine eklemeler yaparak farklı ortamlar oluşturmak mümkündür. Bu dosya içerisinde `$app->detectEnvironment` adında bir tanım bulacaksınız. Bu methoda eklenen bir parametre ile Laravel'e hangi ortamda çalıştığını belirtebilirsiniz. Hatta ihtiyacınız olursa, diğer ortam ve makine isimlerini de dizi olarak ekleyebilirsiniz:

    <?php

    $env = $app->detectEnvironment(array(

        'local' => array('bilgisayarınızın-ismi'),

    ));

In this example, 'local' is the name of the environment and 'bilgisayarınızın-ismi' is the hostname of your server. On Linux and Mac, you may determine your hostname using the `hostname` terminal command.

Dilerseniz, `detectEnvironment` methoduna `Closure` ekleyip ortam algılama özelliğini kendiniz de yazabilirsiniz:

	$env = $app->detectEnvironment(function()
	{
		return $_SERVER['MY_LARAVEL_ENV'];
	});

Şuanki uygulama ortamına `environment` methoduyla erişebilirsiniz:

#### Şuanki Uygulama Ortamına Erişmek

	$environment = App::environment();

You may also pass arguments to the `environment` method to check if the environment matches a given value:

	if (App::environment('local'))
	{
		// The environment is local
	}

	if (App::environment('local', 'staging'))
	{
		// The environment is either local OR staging...
	}

<a name="provider-configuration"></a>
### Provider Configuration

When using environment configuration, you may want to "append" environment [service providers](/docs/ioc#service-providers) to your primary `app` configuration file. However, if you try this, you will notice the environment `app` providers are overriding the providers in your primary `app` configuration file. To force the providers to be appended, use the `append_config` helper method in your environment `app` configuration file:

	'providers' => append_config(array(
		'LocalOnlyServiceProvider',
	))

<a name="protecting-sensitive-configuration"></a>
## Protecting Sensitive Configuration

For "real" applications, it is advisable to keep all of your sensitive configuration out of your configuration files. Things such as database passwords, Stripe API keys, and encryption keys should be kept out of your configuration files whenever possible. So, where should we place them? Thankfully, Laravel provides a very simple solution to protecting these types of configuration items using "dot" files.

First, [configure your application](/docs/configuration#environment-configuration) to recognize your machine as being in the `local` environment. Next, create a `.env.local.php` file within the root of your project, which is usually the same directory that contains your `composer.json` file. The `.env.local.php` should return an array of key-value pairs, much like a typical Laravel configuration file:

	<?php

	return array(

		'TEST_STRIPE_KEY' => 'super-secret-sauce',

	);

All of the key-value pairs returned by this file will automatically be available via the `$_ENV` and `$_SERVER` PHP "superglobals". You may now reference these globals from within your configuration files:

	'key' => $_ENV['TEST_STRIPE_KEY']

Be sure to add the `.env.local.php` file to your `.gitignore` file. This will allow other developers on your team to create their own local environment configuration, as well as hide your sensitive configuration items from source control.

Now, On your production server, create a `.env.php` file in your project root that contains the corresponding values for your production environment. Like the `.env.local.php` file, the production `.env.php` file should never be included in source control.

> **Note:** You may create a file for each environment supported by your application. For example, the `development` environment will load the `.env.development.php` file if it exists.

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

If the Closure passed to the `down` method returns `NULL`, maintenace mode will be ignored for that request.

### Maintenance Mode & Queues

While your application is in maintenance mode, no [queue jobs](/docs/queues) will be handled. The jobs will continue to be handled as normal once the application is out of maintenance mode.