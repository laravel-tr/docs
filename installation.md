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

### Composer'ın Create-Project Komutuyla

Terminalinizde Composer create-project komutunu yayınlayarak Laravel'i yükleyebilirsiniz:

`composer create-project laravel/laravel --prefer-dist`

### Elle İndirerek

Composer yüklendikten sonra, Laravel framework'ün [son sürümü](https://github.com/laravel/laravel/archive/master.zip)nü indirip, içeriğini sunucunuzdaki bir dizine çıkarınız. Ardından, Laravel uygulamanızın ana dizininde, Laravel gereksinimlerini yüklemek için, `php composer.phar install` (veya `composer install`) komutunu çalıştırınız. Bu işlemin başarıyla tamamlanabilmesi için sunucunuzda [Git](http://git-scm.com/downloads) yüklü olması gerekmektedir.

Laravel'i güncellemek isterseniz `php composer.phar update` komutunu yayınlayabilirsiniz.

- Composer kurulumu için Türkçe kaynak: [Composer’ı Evrensel Olarak Kuralım](http://www.sinaneldem.com.tr/composeri-evrensel-olarak-kuralim/)
- Laravel 4 kurulumu için Türkçe kaynak: [Laravel Framework Kurulumu](http://www.sinaneldem.com.tr/laravel-framework-kurulumu/)

<a name="server-requirements"></a>
## Sunucu Gereksinimleri

Laravel framework'un birkaç sistem gereksinimi bulunmaktadır:

- PHP >= 5.3.7
- MCrypt PHP Eklentisi

<a name="configuration"></a>
## Yapılandırma

Laravel'in çalışabilmesi için neredeyse hiç yapılandırma ayarı gerekmez. Geliştirmeye başlamak için serbestsiniz! Ancak `app/config/app.php` dosyasını ve dokümantasyonunu gözden geçirebilirsiniz. Buradaki `timezone` (saat dilimi) ve `locale` (lisan) gibi değerleri uygulamanızın ihtiyaçlarına göre düzenleyebilirsiniz.

<a name="permissions"></a>
### İzinler
Laravel `app/storage` dizin içeriğinin web sunucu tarafından yazılabilir olmasını gerektirmektedir.

<a name="paths"></a>
### Dosya Yolları

Framework dizin yollarının birkaçı yapılandırılabilirdir. Bu dizin yollarını değiştirebilmek için `bootstrap/paths.php` dosyasını gözden geçiriniz.

> **Not:** Laravel, public klasörüne sadece public olması zorunlu dosyaları koymak suretiyle, uygulama kodunuzu ve lokal depolamanızı koruyacak şekilde tasarlanmıştır. Ya public klasörünü sitenizin documentRoot (web root olarak da bilinmektedir)'u olarak ayarlamanız, ya da public klasörünün içeriğini sitenizin kök dizinine koymanız ve Laravelin diğer tüm dosyalarını kök dizini dışına koymanız önerilir.

<a name="pretty-urls"></a>
## Zarif URL'ler

Framework ile beraber gelen `public/.htaccess` dosyası URL'lerin `index.php` olmadan kullanımına olanak sağlamaktadır. Laravel uygulamanızın sunumu için Apache kullanıyorsanız `mod_rewrite` modülünün etkin olduğundan emin olunuz.

Eğer Laravel ile birlikte gelen `.htaccess` dosyası Apache kurulumunuz ile işlev göstermezse, bunu deneyiniz:

	Options +FollowSymLinks
	RewriteEngine On

	RewriteCond %{REQUEST_FILENAME} !-d
	RewriteCond %{REQUEST_FILENAME} !-f
	RewriteRule ^ index.php [L]
