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

### Via Laravel Installer

İlk olarak, [Laravel installer PHAR arşivini indirin](http://laravel.com/laravel.phar). Kolaylık açısından ismini `laravel` olarak değiştirin ve `/usr/local/bin` yoluna taşıyın. Bir kere kurduktan sonra, `laravel new` komutu, istediğiniz klasöre pürüzsüz bir laravel kurulumunu yapacaktır. Örneğin, `laravel new blog` komutu, içinde tüm bağımlılıkları yüklenmiş bir laravel kurulumu barındıran `blog` klasörünü oluşturacaktır. Bu yolla kurulum yapmak Composer ile yapmaktan katbekat kolaydır.

### Composer'ın Create-Project Komutuyla

Terminalinizde Composer create-project komutunu yayınlayarak Laravel'i yükleyebilirsiniz:

`composer create-project laravel/laravel --prefer-dist`

### Elle İndirerek

Composer yüklendikten sonra, Laravel framework'ün [son sürümü](https://github.com/laravel/laravel/archive/master.zip)nü indirip, içeriğini sunucunuzdaki bir dizine çıkarınız. Ardından, Laravel uygulamanızın ana dizininde, Laravel gereksinimlerini yüklemek için, `php composer.phar install` (veya `composer install`) komutunu çalıştırınız. Bu işlemin başarıyla tamamlanabilmesi için sunucunuzda [Git](http://git-scm.com/downloads) yüklü olması gerekmektedir.

Laravel'i güncellemek isterseniz `php composer.phar update` komutunu yayınlayabilirsiniz.

<a name="server-requirements"></a>
## Sunucu Gereksinimleri

Laravel framework'un birkaç sistem gereksinimi bulunmaktadır:

- PHP >= 5.3.7
- MCrypt PHP Eklentisi

As of PHP 5.5, some OS distributions may require you to manually install the PHP JSON extension. When using Ubuntu, this can be done via `apt-get install php5-json`.

<a name="configuration"></a>
## Yapılandırma

Laravel'in çalışabilmesi için neredeyse hiç yapılandırma ayarı gerekmez. Geliştirmeye başlamak için serbestsiniz! Ancak `app/config/app.php` dosyasını ve dokümantasyonunu gözden geçirebilirsiniz. Buradaki `timezone` (saat dilimi) ve `locale` (lisan) gibi değerleri uygulamanızın ihtiyaçlarına göre düzenleyebilirsiniz.

<a name="permissions"></a>
### İzinler
Laravel `app/storage` dizin içeriğinin web sunucu tarafından yazılabilir olmasını gerektirmektedir.

<a name="paths"></a>
### Dosya Yolları

Framework dizin yollarının birkaçı yapılandırılabilirdir. Bu dizin yollarını değiştirebilmek için `bootstrap/paths.php` dosyasını gözden geçiriniz.

<a name="pretty-urls"></a>
## Zarif URL'ler

### Apache

Framework ile beraber gelen `public/.htaccess` dosyası URL'lerin `index.php` olmadan kullanımına olanak sağlamaktadır. Laravel uygulamanızın sunumu için Apache kullanıyorsanız `mod_rewrite` modülünün etkin olduğundan emin olunuz.

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
