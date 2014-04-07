# Yükseltme Rehberi

- [4.1'den 4.2'ye Yükseltme](#upgrade-4.2)
- [4.0'dan 4.1'e Yükseltme](#upgrade-4.1)

<a name="upgrade-4.2"></a>
## 4.1'den 4.2'ye Yükseltme

### PHP 5.4+

Laravel 4.2, PHP 5.4.0 veya daha üstünü gerektirir.

### Modellerdeki Soft Silmeler Artýk Trait Kullanýyorlar

Modellerde soft silmeler kullanýyorsanýz, `softDeletes` propertisi çýkartýlmýþtýr. Artýk aþaðýdakine benzer þekilde `SoftDeletingTrait` kullanmalýsýnýz:

	use Illuminate\Database\Eloquent\SoftDeletingTrait;

	class User extends Eloquent {
		use SoftDeletingTrait;
	}

Ayrýca, `dates` propertisine `deleted_at` sütununu elle eklemeniz gerekir:

	class User extends Eloquent {
		use SoftDeletingTrait;

		protected $dates = ['deleted_at'];
	}

Tüm soft silme iþlemlerinin API'si ayný kalmýþtýr.

### View / Pagination Environment Sýnýflarýnýn Adý Deðiþti

Þayet `Illuminate\View\Environment` sýnýfýný veya `Illuminate\Pagination\Environment` sýnýfýný doðrudan referans ediyorsanýz, kodunuzu bunlar yerine `Illuminate\View\Factory` ve `Illuminate\Pagination\Factory` sýnýflarýný referans verecek þekilde güncellemelisiniz. Bu iki sýnýfýn isimleri, iþlevlerini daha iyi yansýtmasý için deðiþtirilmiþtir.

<a name="upgrade-4.1"></a>
## 4.0'dan 4.1'e Yükseltme

### Composer Baðýmlýlýðýnýn Yükseltilmesi

Uygulamanýzý Laravel 4.1'e yükseltmek için, `composer.json` dosyanýzdaki `laravel/framework` sürümünü `4.1.*` olarak deðiþtirin.

### Dosyalarýn Deðiþtirilmesi

Uygulamanýzdaki `public/index.php` dosyasýný [ambardaki bu yeni kopya](https://github.com/laravel/laravel/blob/master/public/index.php) ile deðiþtirin.

Uygulamanýzdaki `artisan` dosyasýný [ambardaki bu yeni kopya](https://github.com/laravel/laravel/blob/master/artisan) ile deðiþtirin.

### Yapýlandýrma Dosya ve Seçeneklerinin Eklenmesi

Uygulamanýzdaki `app/config/app.php` yapýlandýrma dosyanýzdaki `aliases` ve `providers` dizilerini güncelleyin. Bu dizilerin güncellenmiþ deðerleri [bu dosyada](https://github.com/laravel/laravel/blob/master/app/config/app.php) bulunabilir. Kendi özel ve paket servis saðlayýcýlarýný / aliaslarý tekrar eklemeyi unutmayýn.

[Ambardaki](https://github.com/laravel/laravel/blob/master/app/config/remote.php) yeni `app/config/remote.php` dosyasýný ekleyin.

Uygulamanýzdaki `app/config/session.php` dosyanýza yeni `expire_on_close` yapýlandýrma seçeneðini ekleyin. Ön tanýmlý deðer `false` olmalýdýr.

Uygulamanýzdaki `app/config/queue.php` dosyanýza yeni `failed` yapýlandýrma kesimini ekleyin. Bu kesimin default deðerleri þöyledir:

	'failed' => array(
		'database' => 'mysql', 'table' => 'failed_jobs',
	),

**(Ýsteðe Baðlý)** Uygulamanýzdaki `app/config/view.php` dosyanýzdaki `pagination` yapýlandýrma seçeneðini `pagination::slider-3` olarak güncelleyin.

### Controller Güncellemeleri

Eðer `app/controllers/BaseController.php` dosyasýnda en üstte bir `use` cümlesi varsa, buradaki `use Illuminate\Routing\Controllers\Controller;` olan yeri `use Illuminate\Routing\Controller;` olarak güncelleyin.

### Password Reminders Güncellemeleri

Þifre hatýrlatýcýlarý daha büyük esneklik olmasý için elden geçirilmiþtir. Artisan `php artisan auth:reminders-controller` komutunu çalýþtýrmak suretiyle yeni iskelet controlleri inceleyebilirsiniz. Ayrýca [güncellenmiþ dokümantasyonu](/docs/security#password-reminders-and-reset) da okuyabilir ve uygulamanýzý ona göre güncelleyebilirsiniz.

Uygulamanýzdaki `app/lang/en/reminders.php` dil dosyasýný [güncellenen bu dosyaya](https://github.com/laravel/laravel/blob/master/app/lang/en/reminders.php) uyacak þekilde güncelleyin.

### Ortam Saptama Güncellemeleri

Güvenlik sebepleri nedeniyle, uygulama ortamýnýzý tespit etmek için URL domainleri artýk kullanýlmayabilir. Bu deðerler kolaylýkla kafeslenebilir ve saldýrganlarýn bir istek için ortamý modifiye etmesine imkan verebilir. Ortam tespitinizi makine host adlarý (Mac & Ubuntu üzerinde `hostname` komutu) kullanacak þekilde deðiþtirmelisiniz.

### Daha Sade ve Basit Günlük Dosyalarý

Laravel artýk tek bir log dosyasý üretir: `app/storage/logs/laravel.log`. Bununla birlikte, bu davranýþý yine de `app/start/global.php` dosyanýzda yapýlandýrabilirsiniz.

### En Sonda Bölü Varsa Yeniden Yönlendirin Çýkartýlmasý

Uygulamanýzýn `bootstrap/start.php` dosyasýndan `$app->redirectIfTrailingSlash()` çaðrýsýný çýkartýn. Bu iþlevsellik þimdi frameworkle gelen `.htaccess` dosyasý tarafýndan halledildiði için bu metod artýk gerekli deðildir.

Sonra da, sizin Apache `.htaccess` dosyanýzýn yerine, sondaki bölüleri halleden [bu yenisini](https://github.com/laravel/laravel/blob/master/public/.htaccess) koyun.

### Güncel Rotaya Eriþim

Güncel rotaya `Route::getCurrentRoute()` yerine þimdi `Route::current()` ile eriþilmektedir.

### Composer Güncellemesi

Yukarýdaki deðiþiklikleri tamamladýktan sonra, çekirdek application dosyalarýný güncellemek için `composer update` fonksiyonunu çalýþtýrabilirsiniz! Eðer sýnýf yükleme (class load) hatalarý alýrsanýz, `update` komutunu þu þekilde etkinleþtirilmiþ `--no-scripts` seçeneði ile kullanmayý deneyin: `composer update --no-scripts`.

### Joker Olay Dinleyiciler

Joker Olay Dinleyiciler artýk handler fonksiyon parametrelerinize event'i eklemez. Þayet ateþlenen olayý bulmanýz gerekiyorsa, `Event::firing()` kullanmalýsýnýz.