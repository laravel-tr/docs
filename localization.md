# Yerelleştirme

- [Giriş](#introduction)
- [Dil Dosyaları](#language-files)
- [Temel Kullanım](#basic-usage)
- [Çoğullaştırma](#pluralization)
- [Geçerlilik Denetimi Yerelleştirmesi](#validation)
- [Paket Dil Dosyalarının Ezilmesi](#overriding-package-language-files)

<a name="introduction"></a>
## Giriş

Laravel'in `Lang` sınıfı farklı dillerdeki yazılara ulaşabileceğiniz bir hizmet verir, bu sayede uygulamanızda rahatlıkla çoklu dil desteği verebilirsiniz.

<a name="language-files"></a>
## Dil Dosyaları

Diller için kayıtlar `app/lang` dizininin içerisindeki dosyalarda tutulur. Bu dizin içerisinde desteklenen her dil için bir klasör oluşturulmalıdır.

	/app
		/lang
			/en
				mesajlar.php
			/tr
				mesajlar.php

Dil dosyaları basitçe anahtarlı bir şekilde kayıtları barındıran bir dizi döndürür. Örneğin:

#### Örnek Dil Dosyası

	<?php

	return array(
		'hosgeldiniz' => 'Uygulamamıza hoş geldiniz!'
	);

Uygulamanız için varsayılan dil `app/config/app.php` ayar dosyasında tutulmaktadır. Bunun dışında, aktif dili `App::setLocale` metoduyla çalışma esnasında da değiştirebilirsiniz.

#### Varsayılan Dili Çalışma Esnasında Değiştirmek

	App::setLocale('tr');

<a name="basic-usage"></a>
## Temel Kullanım

#### Bir Dil Dosyasından Satırları Almak

	echo Lang::get('mesajlar.hosgeldin');

`get` metoduna verilen parametrenin ilk kısmı dil dosyasının adını, ikinci kısım ise alınmak istenen satırın anahtarını içerir.

> **Not**: Eğer istenen dil satırı bulunmuyorsa, `get` metodu anahtarı döndürecektir.

`Lang::get()` ile aynı parametreleri kullanan ve bunun kısaltması olan `trans()` yardımcı metodunu kullanabilirsiniz:

	echo trans('mesajlar.hosgeldin');

#### Satırlarda Değişiklik Yapmak

Ayrıca dil satırlarınızda yer tutucular tanımlayabilirsiniz:

	'hosgeldin' => 'Hoşgeldin, :isim',

Daha sonra, `Lang::get` metoduna ikinci bir parametreyle yapılacak değişiklikleri belirtin:

	echo Lang::get('mesajlar.hosgeldin', array('isim' => 'Ekrem'));

#### Bir Dil Dosyasının İstenen Satıra Sahip Olup Olmadığını Kontrol Etmek

	if (Lang::has('mesajlar.hosgeldin'))
	{
		//
	}

<a name="pluralization"></a>
## Çoğullaştırma

Çoğullaştırma karmaşık bir problemdir, çünkü her dilin farklı ve karmaşık çoğullaştırma kuralları vardır. Dil dosyalarınızda bunu kolaylıkla yönetebilirsiniz. `dik çubuk` karakteri ile, bir çevirinin tekil ve çoğul hallerini birbirinden ayırabilirsiniz:

	'elmalar' => 'Bir elma var|Bir sürü elma var',

Daha sonra `Lang::choise` metoduyla satırı alabilirsiniz:

	echo Lang::choice('mesajlar.elmalar', 10);

Ayrıca dili belirtmek için bir locale parametresi de verebilirsiniz. Örneğin, eğer Rusca (ru) dili kullanmak istiyorsanız:

	echo Lang::choice('{0} нет товаров|{1} товар', 1, array(), 'ru');

Laravel'in tercümecisi gücünü Symfony'nin tercüme bileşeninden aldığı için, daha belirgin çoğullaştırma kuralları da belirleyebilirsiniz:

	'elmalar' => '{0} Hiç elma yok|[1,19] Bir kaç elma var|[20,Inf] Çok fazla elma var',

<a name="validation"></a>
## Geçerlilik Denetimi Yerelleştirmesi

Geçerlilik Denetimi hatalarının ve mesajlarının yerelleştirmesi için dokümantasyonun <a href="/docs/validation#localization">Geçerlilik Denetimi</a> bölümüne bakınız.

<a name="overriding-package-language-files"></a>
## Paket Dil Dosyalarının Ezilmesi

Birçok paket kendi dil satırlarıyla gelir. Bu satırları değiştirmek için paketin çekirdek dosyalarıyla oynamak yerine, `app/lang/packages/{locale}/{package}` dizinine dosyalar koymak suretiyle onları ezebilirsiniz. Dolayısıyla, örneğin, eğer `skyrim/hearthfire` adındaki bir paket için `messages.php`'yi Türkçe dil satırlarıyla ezmeniz gerekiyorsa koyacağınız dil dosyası şudur: `app/lang/packages/tr/hearthfire/messages.php`. Bu dosyada sadece ezmek istediğiniz dil satırlarını tanımlayacaksınız. Ezmediğiniz dil satırları paketin dil dosyalarından yüklenmeye devam edecektir.