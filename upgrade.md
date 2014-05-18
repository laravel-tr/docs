# Yükseltme Rehberi

- [4.1'den 4.2'ye Yükseltme](#upgrade-4.2)
- [4.1.25 ve Öncesinden 4.1.26'ye Yükseltme](#upgrade-4.1.26)
- [4.0'dan 4.1'e Yükseltme](#upgrade-4.1)

<a name="upgrade-4.2"></a>
## 4.1'den 4.2'ye Yükseltme

### PHP 5.4+

Laravel 4.2, PHP 5.4.0 veya daha üstünü gerektirir.

### Kriptolama Varsayılanları

`app/config/app.php` yapılandırma dosyanıza yeni bir `cipher` seçeneği ekleyin. Bu seçeneğin değeri `MCRYPT_RIJNDAEL_256` olmalıdır.

	'cipher' => MCRYPT_RIJNDAEL_256

Bu ayar, Laravel kriptolama araçları tarafından kullanılan varsayılan cipher'i (kriptolama sistemini) kontrol etmek için kullanılabilir.

### Modellerdeki Soft Silmeler Artık Trait Kullanıyorlar

Modellerde soft silmeler kullanıyorsanız, `softDeletes` propertisi çıkartılmıştır. Artık aşağıdakine benzer şekilde `SoftDeletingTrait` kullanmalısınız:

	use Illuminate\Database\Eloquent\SoftDeletingTrait;

	class User extends Eloquent {
		use SoftDeletingTrait;
	}

Ayrıca, `dates` propertisine `deleted_at` sütununu elle eklemeniz gerekir:

	class User extends Eloquent {
		use SoftDeletingTrait;

		protected $dates = ['deleted_at'];
	}

Tüm soft silme işlemlerinin API'si aynı kalmıştır.

### View / Pagination Environment Sınıflarının Adı Değişti

Şayet `Illuminate\View\Environment` sınıfını veya `Illuminate\Pagination\Environment` sınıfını doğrudan referans ediyorsanız, kodunuzu bunlar yerine `Illuminate\View\Factory` ve `Illuminate\Pagination\Factory` sınıflarını referans verecek şekilde güncellemelisiniz. Bu iki sınıfın isimleri, işlevlerini daha iyi yansıtması için değiştirilmiştir.

### Pagination Sunumcusunda Ek Parametre

Eğer `Illuminate\Pagination\Presenter` sınıfını genişletiyorsanız, `getPageLinkWrapper` abstract metodunun kalıbı  `rel` parametresi eklenecek şekilde değiştirilmiştir:

	abstract public function getPageLinkWrapper($url, $page, $rel = null);

<a name="upgrade-4.1.26"></a>
## 4.1.25 ve Öncesinden 4.1.26'ye Yükseltme

Laravel 4.1.26 "remember me" cookie'leri için güvenlik iyileştirmeleri getirdi. Bu güncellemeler öncesinde, eğer bir remember cookie kötü niyetli başka bir kullanıcı tarafından gasp edilmişse ("hijacked"), hesabın gerçek sahibi kendi şifresini yeniledikten, çıkış yaptıktan (logged out) v.b sonra bile ilgili cookie uzun bir zaman süresince geçerli kalırdı.

Bu değişiklik `users` (veya dengi olan) veritabanı tablonuza yeni bir `remember_token` sütunu eklenmesini gerektirmektedir. Bu değişiklikten sonra, uygulamanıza giriş (login) yaptıkları her seferinde kullanıcıya yepyeni bir token atanacaktır. Bu token ayrıca kullanıcı uygulamadan çıkış yaptığı zaman da yenilenecektir. Bu değişikliğin etkileri şunlardır: eğer bir "remember me" cookie gasp edilirse, sadece uygulamadan çıkış yapılması bu cookie'yi geçersiz kılacaktır.

### Yükseltme Adımları

Öncelikle, `users` tablonuza VARCHAR(100), TEXT veya dengi yeni bir nullable `remember_token` sütunu ekleyin.

Daha sonra, eğer Eloquent authentication sürücüsü kullanıyorsanız, `User` sınıfınızı aşağıdaki üç metodla güncelleyin:

	public function getRememberToken()
	{
		return $this->remember_token;
	}

	public function setRememberToken($value)
	{
		$this->remember_token = $value;
	}

	public function getRememberTokenName()
	{
		return 'remember_token';
	}

> **Not:** Bu değişiklikle mevcut tüm "remember me" oturumları geçersiz kılınacaktır, bu nedenle tüm kullanıcılar uygulamanıza yeniden authenticate olmaya (kimliği doğrulanmaya) zorlanacaklardır.

### Paket Sürdürücüleri

`Illuminate\Auth\UserProviderInterface` interface'ine iki yeni metod eklenmiştir. Örnek implementationlar ön tanımlı sürücülerde bulunabilir:

	public function retrieveByToken($identifier, $token);

	public function updateRememberToken(UserInterface $user, $token);

Ayrıca, `Illuminate\Auth\UserInterface` de "Yükseltme Adımları" kesiminde açıklanan üç yeni metodu almıştır.

<a name="upgrade-4.1"></a>
## 4.0'dan 4.1'e Yükseltme

### Composer Bağımlılığının Yükseltilmesi

Uygulamanızı Laravel 4.1'e yükseltmek için, `composer.json` dosyanızdaki `laravel/framework` sürümünü `4.1.*` olarak değiştirin.

### Dosyaların Değiştirilmesi

Uygulamanızdaki `public/index.php` dosyasını [ambardaki bu yeni kopya](https://github.com/laravel/laravel/blob/master/public/index.php) ile değiştirin.

Uygulamanızdaki `artisan` dosyasını [ambardaki bu yeni kopya](https://github.com/laravel/laravel/blob/master/artisan) ile değiştirin.

### Yapılandırma Dosya ve Seçeneklerinin Eklenmesi

Uygulamanızdaki `app/config/app.php` yapılandırma dosyanızdaki `aliases` ve `providers` dizilerini güncelleyin. Bu dizilerin güncellenmiş değerleri [bu dosyada](https://github.com/laravel/laravel/blob/master/app/config/app.php) bulunabilir. Kendi özel ve paket servis sağlayıcılarını / aliasları tekrar eklemeyi unutmayın.

[Ambardaki](https://github.com/laravel/laravel/blob/master/app/config/remote.php) yeni `app/config/remote.php` dosyasını ekleyin.

Uygulamanızdaki `app/config/session.php` dosyanıza yeni `expire_on_close` yapılandırma seçeneğini ekleyin. Ön tanımlı değer `false` olmalıdır.

Uygulamanızdaki `app/config/queue.php` dosyanıza yeni `failed` yapılandırma kesimini ekleyin. Bu kesimin default değerleri şöyledir:

	'failed' => array(
		'database' => 'mysql', 'table' => 'failed_jobs',
	),

**(İsteğe Bağlı)** Uygulamanızdaki `app/config/view.php` dosyanızdaki `pagination` yapılandırma seçeneğini `pagination::slider-3` olarak güncelleyin.

### Controller Güncellemeleri

Eğer `app/controllers/BaseController.php` dosyasında en üstte bir `use` cümlesi varsa, buradaki `use Illuminate\Routing\Controllers\Controller;` olan yeri `use Illuminate\Routing\Controller;` olarak güncelleyin.

### Password Reminders Güncellemeleri

Şifre hatırlatıcıları daha büyük esneklik olması için elden geçirilmiştir. Artisan `php artisan auth:reminders-controller` komutunu çalıştırmak suretiyle yeni iskelet controlleri inceleyebilirsiniz. Ayrıca [güncellenmiş dokümantasyonu](/docs/security#password-reminders-and-reset) da okuyabilir ve uygulamanızı ona göre güncelleyebilirsiniz.

Uygulamanızdaki `app/lang/en/reminders.php` dil dosyasını [güncellenen bu dosyaya](https://github.com/laravel/laravel/blob/master/app/lang/en/reminders.php) uyacak şekilde güncelleyin.

### Ortam Saptama Güncellemeleri

Güvenlik sebepleri nedeniyle, uygulama ortamınızı tespit etmek için URL domainleri artık kullanılmayabilir. Bu değerler kolaylıkla kafeslenebilir ve saldırganların bir istek için ortamı modifiye etmesine imkan verebilir. Ortam tespitinizi makine host adları (Mac, Linux ve Windows üzerinde `hostname` komutu) kullanacak şekilde değiştirmelisiniz.

### Daha Sade ve Basit Günlük Dosyaları

Laravel artık tek bir log dosyası üretir: `app/storage/logs/laravel.log`. Bununla birlikte, bu davranışı yine de `app/start/global.php` dosyanızda yapılandırabilirsiniz.

### En Sonda Bölü Varsa Yeniden Yönlendirin Çıkartılması

Uygulamanızın `bootstrap/start.php` dosyasından `$app->redirectIfTrailingSlash()` çağrısını çıkartın. Bu işlevsellik şimdi frameworkle gelen `.htaccess` dosyası tarafından halledildiği için bu metod artık gerekli değildir.

Sonra da, sizin Apache `.htaccess` dosyanızın yerine, sondaki bölüleri halleden [bu yenisini](https://github.com/laravel/laravel/blob/master/public/.htaccess) koyun.

### Güncel Rotaya Erişim

Güncel rotaya `Route::getCurrentRoute()` yerine şimdi `Route::current()` ile erişilmektedir.

### Composer Güncellemesi

Yukarıdaki değişiklikleri tamamladıktan sonra, çekirdek application dosyalarını güncellemek için `composer update` fonksiyonunu çalıştırabilirsiniz! Eğer sınıf yükleme (class load) hataları alırsanız, `update` komutunu şu şekilde etkinleştirilmiş `--no-scripts` seçeneği ile kullanmayı deneyin: `composer update --no-scripts`.

### Joker Olay Dinleyiciler

Joker Olay Dinleyiciler artık handler fonksiyon parametrelerinize event'i eklemez. Şayet ateşlenen olayı bulmanız gerekiyorsa, `Event::firing()` kullanmalısınız.
