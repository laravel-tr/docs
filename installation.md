# Kurulum

- [Composer Kurulumu](#install-composer)
- [Laravel Yükleme](#install-laravel)
- [Sunucu Gereksinimleri](#server-requirements)

<a name="install-composer"></a>
## Composer Kurulumu

Laravel bağımlılıklarını yönetmek için [Composer](http://getcomposer.org) kullanır. Bu yüzden Laravel'i kullanmadan önce, bilgisayarınızda Composer kurulu olduğundan emin olmanız gerekir.

<a name="install-laravel"></a>
## Laravel Yükleme

### Laravel Installer Aracılığıyla

Öncelikle, Composer kullanarak Laravel yükleyicisini indiriniz..

	composer global require "laravel/installer=~1.1"

Terminalinizde `laravel` komutunu çalıştırdığınızda `laravel` çalıştırıcısının bulunabilmesi için PATH'inizde `~/.composer/vendor/bin` dizininin bulunduğundan emin olun.

Bunu bir kere kurduktan sonra, basit `laravel new` komutu sizin belirttiğiniz dizine yeni bir Laravel yüklemesi oluşturacaktır. Örneğin, `laravel new blog` komutu, içinde tüm bağımlılıkları yüklenmiş yeni bir laravel kurulumu barındıran `blog` klasörünü oluşturacaktır. Bu yolla kurulum yapmak Composer aracılığıyla yüklemekten çok daha hızlıdır:

	laravel new blog

### Composer'ın Create-Project Komutuyla

Terminalinizde Composer `create-project` komutunu vererek Laravel'i yükleyebilirsiniz:

	composer create-project laravel/laravel --prefer-dist

<a name="server-requirements"></a>
## Sunucu Gereksinimleri

Laravel framework'un birkaç sistem gereksinimi bulunmaktadır:

- PHP >= 5.4
- Mcrypt PHP Eklentisi
- OpenSSL PHP Eklentisi
- Mbstring PHP Eklentisi

PHP 5.5 için, bazı OS yayımlamaları PHP JSON eklentisinin elle yüklenmesini gerektirebilir. Ubuntu kullanırken, bu `apt-get install php5-json` aracılığı ile yapılabilir.

<a name="configuration"></a>
## Yapılandırma

Laravel'i kurduktan sonra yapmanız gereken ilk şey; rastgele bir dizeden oluşan uygulama anahtarını girmektir. Laravel'i Composer aracılığı ile kurduysanız, bu anahtar sizin için `key:generate` komutu tarafından tanımlanmıştır.

Örneğin, Bu dize 32 karakter uzunluğunda olmalıdır. Bu anahtar`app.php` yapılandırma dosyasında ayarlanabilir. **Eğer uygulama anahtarı değeri tanımlanmamışsa, sizin kullanıcı oturum bilgileriniz ve diğer şifreli verileriniz güvenli olmayacaktır!**

Laravel'in çalışabilmesi için neredeyse hiç yapılandırma ayarı gerekmez. Geliştirmeye hemen başlayabilirsiniz! Ancak `config/app.php` dosyasını ve dokümantasyonunu gözden geçirebilirsiniz. Buradaki `timezone` (saat dilimi) ve `locale` gibi değerleri uygulamanızın ihtiyaçlarına göre düzenleyebilirsiniz.

Laravel yüklendikten sonra, [local ortamınızı yapılandırmanız](/docs/master/configuration#environment-configuration) da gerekmektedir. 

> **Not:** Bir üretim ortamında `app.debug` değerini asla `true` ayarlamamalısınız. Bunu hiçbir zaman yapmayın.

<a name="permissions"></a>
### İzinler

Laravel `storage` dizin içeriğinin web sunucu tarafından yazılabilir olmasını gerektirmektedir.

<a name="pretty-urls"></a>
## Zarif URL'ler

### Apache

Laravel framework, URL'lerin index.php olmadan kullanımına imkan vermek için kullanılan bir public/.htaccess dosyası ile birlikte gelmektedir. Laravel uygulamanızın sunumu için Apache kullanıyorsanız mod_rewrite modülünün etkin olduğundan emin olunuz.

Eğer Laravel ile birlikte gelen .htaccess dosyası Apache kurulumunuz ile işlev göstermezse, bunu deneyiniz:

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

[Homestead](/docs/master/homestead) kullanırsanız, zarif URL'ler otomatik yapılandırılacaktır.
