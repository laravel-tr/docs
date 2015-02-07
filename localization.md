# Yerelleştirme

- [Önsöz](#introduction)
- [Dil Dosyaları](#language-files)
- [Temel Kullanım](#basic-usage)
- [Çoğullaştırma](#pluralization)
- [Geçerlilik Denetimi Yerelleştirmesi](#validation)
- [Paket Dil Dosyalarının Ezilmesi](#overriding-package-language-files)

<a name="introduction"></a>
## Önsöz

Laravel'in Lang sınıfı farklı dillerdeki yazılara ulaşabileceğiniz bir hizmet verir, bu sayede uygulamanızda rahatlıkla çoklu dil desteği verebilirsiniz.

<a name="language-files"></a>
## Dil Dosyaları

Diller için kayıtlar `resources/lang` dizininin içerisindeki dosyalarda tutulur. Bu dizin içerisinde desteklenen her dil için bir klasör oluşturulmalıdır.

	/resources
		/lang
			/en
				messages.php
			/es
				messages.php

#### Örnek Dil Dosyası

Dil dosyaları basitçe anahtarlı bir şekilde kayıtları barındıran bir dizi döndürür. Örneğin:

	<?php

	return array(
		'welcome' => 'Welcome to our application'
	);

#### Varsayılan Dili Çalışma Esnasında Değiştirmek

The default language for your application is stored in the `config/app.php` configuration file. You may change the active language at any time using the `App::setLocale` method:

	App::setLocale('es');

#### Yedek Dil Ayarı

Etkin dil verilen bir dil satırını içermediğinde kullanılacak olan bir "yedek dil" de yapılandırabilirsiniz. Varsayılan dile benzer şekilde, yedek dil de `config/app.php` yapılandırma dosyasında yapılandırılır:

	'fallback_locale' => 'en',

<a name="basic-usage"></a>
## Temel Kullanım

#### Bir Dil Dosyasından Satırları Almak

	echo Lang::get('messages.welcome');

`get` metoduna verilen parametrenin ilk kısmı dil dosyasının adını, ikinci kısım ise alınmak istenen satırın anahtarını içerir.

> **Not:** Eğer istenen dil satırı bulunmuyorsa, `get` metodu anahtarı döndürecektir.

`Lang::get` ile aynı parametreleri kullanan ve bunun kısaltması olan `trans` yardımcı metodunu kullanabilirsiniz:

	echo trans('messages.welcome');

#### Satırlarda Değişiklik Yapmak

Ayrıca dil satırlarınızda yer tutucular tanımlayabilirsiniz:

	'welcome' => 'Welcome, :name',

Daha sonra, `Lang::get` metoduna ikinci bir parametreyle yapılacak değişiklikleri belirtin:

	echo Lang::get('messages.welcome', array('name' => 'Dayle'));

#### Bir Dil Dosyasının İstenen Satıra Sahip Olup Olmadığını Kontrol Etmek

	if (Lang::has('messages.welcome'))
	{
		//
	}

<a name="pluralization"></a>
## Çoğullaştırma

Çoğullaştırma karmaşık bir problemdir, çünkü her dilin farklı ve karmaşık çoğullaştırma kuralları vardır. Dil dosyalarınızda bunu kolaylıkla yönetebilirsiniz. "dik çubuk" karakteri ile, bir çevirinin tekil ve çoğul hallerini birbirinden ayırabilirsiniz:

	'apples' => 'There is one apple|There are many apples',

Daha sonra `Lang::choice` metoduyla satırı alabilirsiniz:

	echo Lang::choice('messages.apples', 10);

Ayrıca dili belirtmek için bir locale parametresi de verebilirsiniz. Örneğin, eğer Rusca (ru) dili kullanmak istiyorsanız

	echo Lang::choice('товар|товара|товаров', $count, array(), 'ru');

Laravel'in tercümecisi gücünü Symfony'nin tercüme bileşeninden aldığı için, daha belirgin çoğullaştırma kuralları da belirleyebilirsiniz:

	'apples' => '{0} There are none|[1,19] There are some|[20,Inf] There are many',


<a name="validation"></a>
## Geçerlilik Denetimi Yerelleştirmesi

Geçerlilik Denetimi hatalarının ve mesajlarının yerelleştirmesi için dokümantasyonun <a href="/docs/master/validation#localization">Geçerlilik Denetimi</a> bölümüne bakınız.

<a name="overriding-package-language-files"></a>
## Paket Dil Dosyalarının Ezilmesi

Birçok paket kendi dil satırlarıyla gelir. Bu satırları değiştirmek için paketin çekirdek dosyalarıyla oynamak yerine, `resources/lang/packages/{locale}/{package}` dizinine dosyalar koymak suretiyle onları ezebilirsiniz. Dolayısıyla, örneğin, eğer `skyrim/hearthfire` adındaki bir paket için `messages.php`'yi Türkçe dil satırlarıyla ezmeniz gerekiyorsa koyacağınız dil dosyası şudur: `resources/lang/packages/en/hearthfire/messages.php`. Bu dosyada sadece ezmek istediğiniz dil satırlarını tanımlayacaksınız. Ezmediğiniz dil satırları paketin dil dosyalarından yüklenmeye devam edecektir.

