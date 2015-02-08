# Artisan CLI

- [Giriş](#introduction)
- [Kullanım](#usage)
- [Artisan Komutlarının CLI Dışında Kullanımı](#calling-commands-outside-of-cli)
- [Zaman Ayarlı Artisan Komutları](#scheduling-artisan-commands)

<a name="introduction"></a>
## Giriş

Artisan, Laravel içerisinde gelen CLI (Komut Satırı Arayüzü)'ın adıdır. Artisan size uygulamanızı geliştirirken birçok yardımcı komut sağlar. Artisan, güçlü Symfony Console bileşeni üzerinden geliştirilmiştir.

<a name="usage"></a>
## Kullanım

#### Tüm Artisan Komutlarının Listelenmesi

Tüm Artisan komutlarının listesini görmek için, `list` komutunu kullanabilirsiniz:

	php artisan list

#### Bir Komut için Yardım Ekranının Görüntülenmesi

Her komut ayrıca bir yardım ekranı içerir. Bu ekran komutların erişilebir parametlerini ve seçeneklerini görüntüler. Yardım ekranını görüntülemek için `help` ile birlikte komutun adını girin:

	php artisan help migrate

#### Yapılandırma Ortamının Belirtilmesi

Bir komut çalışırken `--env` ile kullanılacak yapılandırma ortamını belirtebilirsiniz:

	php artisan migrate --env=local

#### Güncel Laravel Sürümünüzün Gösterilmesi

Ayrıca Laravel yüklemenizin güncel sürümünü de `--version` seçeneğini kullanarak görebilirsiniz:

	php artisan --version

<a name="calling-commands-outside-of-cli"></a>
## Komutların CLI'in Dışından Çağrılması

Bazen Artisan komutlarını CLI dışından çalıştırmaya ihtiyacınız olabilir. Örneğin, Bir HTTP rotasındayken bir Artisan komutunu ateşleyerek çalıştırabiliriz. Bunu yapmak için sadece `Artisan` facade'ını kullanmanız yeterlidir:

	Route::get('/foo', function()
	{
		$exitCode = Artisan::call('command:name', ['--option' => 'foo']);

		//
	});

Artisan komutlarını kuyruğa ekleyerek arkaplanda [queue workers](/docs/master/queues) tarafından işlenmesini sağlayabilirsiniz:

	Route::get('/foo', function()
	{
		Artisan::queue('command:name', ['--option' => 'foo']);

		//
	});

<a name="scheduling-artisan-commands"></a>
## Artisan Komutlarının Zamanlanması

Geçmişte, geliştiriciler zaman ayarlı olarak çalıştırmak istedikleri her bir komut için bir Cron tanımalaması yapmak zorundaydılar. Ancak, bu bir baş ağrısıdır. Sizin zaman ayarlı komutlarınız kaynak kod kontrol sistemi içerisinde olmadığından dolayı, yeni bir Cron tanımlaması yapmak için SSH ile sunucunuza bağlanmanız gerekmekteydi. Hadi hayatımızı kolaylaştıralım. Laravel komut planlayıcısı sizin zaman ayarlı komutlarınızı tanımlamanızı sağlar ve sizin sunucunuzda sadece bir tane Cron tanımalamasına ihtiyaç duyar.

Sizin zaman ayarlı komutlarınız `app/Console/Kernel.php` dosyasında tutulur. Bu sınıfın içinde `schedule` isimli bir metod göreceksiniz. Başlamanıza yardımcı olmak için, bu metoda basit bir örnek dahil edilmiştir. `Schedule` nesnesine bir çok zaman ayarlı komutu ekleme konusunda özgürsünüz. Sadece aşağıdaki şekilde bir Cron tanımlaması sunucunuza yapmalısınız:

	* * * * * php /path/to/artisan schedule:run 1>> /dev/null 2>&1

Bu Cron her saniye Laravel komut planlayıcısını çağıracak. Sonra, Laravel sizin zaman ayarkı işlerinizi değerlendirir ve and ve bağlantılı olan işleri çalıştırır. Daha kolay olamazdı!

### Daha Fazla Zaman Ayarlı Örnekler

Daha fazla zaman ayarlı öğrneğe bakalım:

#### Zaman Ayarlı Closure'lar

	$schedule->call(function()
	{
		// Do some task...

	})->hourly();

#### Zaman Ayarlı Terminal Komutları

	$schedule->exec('composer self-update')->daily();

#### Manuel Cron İfadesi

	$schedule->command('foo')->cron('* * * * *');

#### Belirli Aralıklı Görevler

	$schedule->command('foo')->everyFiveMinutes();

	$schedule->command('foo')->everyTenMinutes();

	$schedule->command('foo')->everyThirtyMinutes();

#### Günlük Görevler

	$schedule->command('foo')->daily();

#### Belirli Bir Zamandaki Görevler (24 Saat)

	$schedule->command('foo')->dailyAt('15:00');

#### Günde 2 Kere Çalışan Görevler

	$schedule->command('foo')->twiceDaily();

#### Haftaiçi Hergün Çalışan Görevler

	$schedule->command('foo')->weekdays();

#### Haftalık Görevler

	$schedule->command('foo')->weekly();

	// Haftanın belirli bir gün(0-6) ve saatinde çalışan zaman ayarlı görev...
	$schedule->command('foo')->weeklyOn(1, '8:00');

#### Aylık Görevler

	$schedule->command('foo')->monthly();

#### Belirli bir ordamda(Environment) çalışacak şekilde sınırlandırma

	$schedule->command('foo')->monthly()->environments('production');

#### Uygulama bakım modunda olsa bile görevlerin çalıştırılmasını sağlar

	$schedule->command('foo')->monthly()->evenInMaintenanceMode();

#### Sadece Callback True Olduğu Zaman Çalışacak Görev

	$schedule->command('foo')->monthly()->when(function()
	{
		return true;
	});
