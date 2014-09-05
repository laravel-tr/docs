# Kurulum

- [Composer Kurulumu](#install-composer)
- [Laravel Yükleme](#install-laravel)
- [Sunucu Gereksinimleri](#server-requirements)
- [Yapılandırma](#configuration)
- [Zarif URL'ler](#pretty-urls)

<a name="install-composer"></a>
## Composer Kurulumu

Laravel bağımlılıklarını yönetmek için [Composer](http://getcomposer.org) kullanır. Öncelikle `composer.phar` dosyasını indiriniz. PHAR arşivini yerel proje dosyanızda tutabileceğiniz gibi `usr/local/bin` içerisine taşıyarak sisteminizde evrensel olarak da kullanabilirsiniz. Windows'ta Composer [Windows kurulumu](https://getcomposer.org/Composer-Setup.exe)nu kullanabilirsiniz. Setup Composer'i PATH değişkeni olarak kaydedecektir, böylece terminal üzerinde `composer` yazdığınızda Composer'i direkt olarak kullanabilirsiniz.

<a name="install-laravel"></a>
## Laravel Yükleme

### Laravel Installer Aracılığıyla

Öncelikle, Composer kullanarak Laravel yükleyicisini indiriniz.

	composer global require "laravel/installer=~1.1"

Terminalinizde `laravel` komutunu çalıştırdığınızda `laravel` çalıştırıcısının bulunabilmesi için PATH'inizde `~/.composer/vendor/bin` dizininin bulunduğundan emin olun.

Bunu bir kere kurduktan sonra, basit `laravel new` komutu sizin belirttiğiniz dizine yeni bir Laravel yüklemesi oluşturucaktır. Örneğin, `laravel new blog` komutu, içinde tüm bağımlılıkları yüklenmiş yeni bir laravel kurulumu barındıran `blog` klasörünü oluşturacaktır. Bu yolla kurulum yapmak Composer aracılığıyla yüklemekten çok daha hızlıdır.

### Composer'ın Create-Project Komutuyla

Terminalinizde Composer create-project komutunu vererek Laravel'i yükleyebilirsiniz:

	composer create-project laravel/laravel --prefer-dist

### Elle İndirerek

Composer yüklendikten sonra, Laravel framework'ün [son sürümünü](https://github.com/laravel/laravel/archive/master.zip) indirip, içeriğini sunucunuzdaki bir dizine çıkarınız. Ardından, Laravel uygulamanızın ana dizininde, Laravel gereksinimlerini yüklemek için, `php composer.phar install` (veya `composer install`) komutunu çalıştırınız. Bu işlemin başarıyla tamamlanabilmesi için sunucunuzda [Git](http://git-scm.com/downloads) yüklü olması gerekmektedir.

Laravel'i güncellemek isterseniz `php composer.phar update` komutunu verebilirsiniz.

<a name="server-requirements"></a>
## Sunucu Gereksinimleri

Laravel framework'un birkaç sistem gereksinimi bulunmaktadır:

- PHP >= 5.4
- MCrypt PHP Eklentisi

PHP 5.5 için, bazı OS yayımlamaları PHP JSON eklentisinin elle yüklenmesini gerektirebilir. Ubuntu kullanırken, bu `apt-get install php5-json` aracılığı ile yapılabilir.

<a name="configuration"></a>
## Yapılandırma

Laravel'in çalışabilmesi için neredeyse hiç yapılandırma ayarı gerekmez. Geliştirmeye hemen başlayabilirsiniz! Ancak `app/config/app.php` dosyasını ve dokümantasyonunu gözden geçirebilirsiniz. Buradaki `timezone` (saat dilimi) ve `locale` gibi değerleri uygulamanızın ihtiyaçlarına göre düzenleyebilirsiniz.

Laravel yüklendikten sonra, [local ortamınızı yapılandırmanız](/docs/configuration#environment-configuration) da gerekmektedir. Bu size local makinenizde geliştirme yaparken ayrıntılı hata mesajları alma imkanı verecektir. Ön tanımlı olarak, üretim yapılandırma dosyanızdaki ayrıntılı hata bildirimi devre dışı durumdadır.

> **Not:** Bir üretim ortamında `app.debug` değerini asla `true` ayarlamamalısınız. Bunu hiçbir zaman yapmayın.

<a name="permissions"></a>
### İzinler
Laravel `app/storage` dizin içeriğinin web sunucu tarafından yazılabilir olmasını gerektirmektedir.

<a name="paths"></a>
### Dosya Yolları

Framework dizin yollarının birkaçı yapılandırılabilirdir. Bu dizin yollarını değiştirebilmek için `bootstrap/paths.php` dosyasını gözden geçiriniz.

<a name="pretty-urls"></a>
## Zarif URL'ler

### Apache

Laravel framework, URL'lerin `index.php` olmadan kullanımına imkan vermek için kullanılan bir `public/.htaccess` dosyası ile birlikte gelmektedir. Laravel uygulamanızın sunumu için Apache kullanıyorsanız `mod_rewrite` modülünün etkin olduğundan emin olunuz.

Eğer Laravel ile birlikte gelen `.htaccess` dosyası Apache kurulumunuz ile işlev göstermezse, bunu deneyiniz:

	Options +FollowSymLinks
	RewriteEngine On

	RewriteCond %{REQUEST_FILENAME} !-d
	RewriteCond %{REQUEST_FILENAME} !-f
	RewriteRule ^ index.php [L]

### Nginx

Nginx kullanıyorsanız, ekteki ayar "zarif url"lerin çalışmasını sağlamaya yeterlidir:

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }
