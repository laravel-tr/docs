# SSH

- [Yapýlandýrma](#configuration)
- [Temel Kullaným](#basic-usage)
- [Görevler](#tasks)
- [SFTP Dosya Ýndirmeleri](#sftp-downloads)
- [SFTP Dosya Göndermeleri](#sftp-uploads)
- [Uzak Günlüklerin Ýzlenmesi](#tailing-remote-logs)
- [Envoy Görev Çalýþtýrýcýsý](#envoy-task-runner)

<a name="configuration"></a>
## Yapýlandýrma

Laravel uzak sunuculara SSH (Secure Shell) iletiþimi ve komutlar çalýþtýrmak için basit bir yol içerir ve uzak sunucularda çalýþan Artisan görevlerini kolayca inþa etmenize imkan verir. `SSH` facade'ý uzak sunucularýnýza baðlanmanýz ve komutlar çalýþtýrmanýz için eriþim noktasý saðlar.

Yapýlandýrma dosyasý `app/config/remote.php` konumundadýr ve uzak baðlantýlarýnýzý yapýlandýrmak için gerekli tüm seçenekleri içerir. Bu dosyadaki `connections` dizisi, sürücülerinizin isimlerine göre anahtarlanmýþ bir listesini taþýr. Bu `connections` dizisindeki host, username, password, key gibi eriþim güven bilgilerini (credentials) doldurduktan sonra uzak görevleri çalýþtýrmaya hazýr olacaksýnýz. Unutmayýn, `SSH`, ya bir password ya da bir SSH key kullanarak kimlik doðrulamasý yapabilmektedir.

> **Not:** Uzak sunucunuzda çeþitli görevleri kolayca çalýþtýrma ihtiyacýnýz mý var? [Envoy görev çalýþtýrýcýsýna](#envoy-task-runner) bir bakýn!

<a name="basic-usage"></a>
## Temel Kullaným

#### Komutlarý Default Sunucuda Çalýþtýrmak

Komutlarýnýzý `default` uzak baðlantýnýzda çalýþtýrmak için `SSH::run` metodunu kullanýn:

	SSH::run(array(
		'cd /var/www',
		'git pull origin master',
	));

#### Komutlarý Belirli Bir Baðlantýda Çalýþtýrmak

Alternatif olarak, `into` metodunu kullanmak suretiyle komutlarý belirli bir baðlantý üzerinde çalýþtýrabilirsiniz:

	SSH::into('staging')->run(array(
		'cd /var/www',
		'git pull origin master',
	));

#### Komut Çýktýlarýný Yakalamak

`run` metoduna bir Closure geçmek suretiyle, uzak komutlarýnýzýn "canlý" çýktýsýný yakalayabilirsiniz:

	SSH::run($commands, function($line)
	{
		echo $line.PHP_EOL;
	});

## Görevler
<a name="tasks"></a>

Eðer her zaman birlikte çalýþmasý gereken bir grup komut tanýmlamanýz gerekiyorsa, bir `task` (görev) tanýmlamak için `define` metodunu kullanabilirsiniz:

	SSH::into('staging')->define('deploy', array(
		'cd /var/www',
		'git pull origin master',
		'php artisan migrate',
	));

Bu þekilde bir task tanýmladýktan sonra, onu çalýþtýrmak için `task` metodunu kullanabilirsiniz:

	SSH::into('staging')->task('deploy', function($line)
	{
		echo $line.PHP_EOL;
	});

<a name="sftp-downloads"></a>
## SFTP Dosya Ýndirmeleri

`SSH` sýnýfý `get` ve `getString` metodlarý kullanýlarak, dosyalar indirmek için basit bir yol saðlar:

	SSH::into('staging')->get($remotePath, $localPath);

	$contents = SSH::into('staging')->getString($remotePath);

<a name="sftp-uploads"></a>
## SFTP Dosya Göndermeleri

`SSH` sýnýfý ayný zamanda `put` ve `putString` metodlarý kullanýlarak sunucuya dosyalar, hatta stringler upload etmek için de basit bir yol içerir:

	SSH::into('staging')->put($localFile, $remotePath);

	SSH::into('staging')->putString($remotePath, 'Falan');

<a name="tailing-remote-logs"></a>
## Uzak Günlüklerin Ýzlenmesi

Laravel sizin uzak baðlantýlarýnýzýn herhangi birindeki `laravel.log` dosyalarýnýn izlenmesi için yararlý bir komut içermektedir. Bunun için basitçe `tail` Artisan komutunu kullanýn ve izlemek istediðiniz uzak baðlantýnýn adýný belirtin:

	php artisan tail staging

	php artisan tail staging --path=/path/to/log.file

<a name="envoy-task-runner"></a>
## Envoy Görev Çalýþtýrýcýsý

- [Yükleme](#envoy-installation)
- [Görevlerin Çalýþtýrýlmasý](#envoy-running-tasks)
- [Birden Çok Sunucu](#envoy-multiple-servers)
- [Paralel Çalýþtýrma](#envoy-parallel-execution)
- [Task Makrolarý](#envoy-task-macros)
- [Bildirimler](#envoy-notifications)
- [Envoy'in Güncellenmesi](#envoy-updating-envoy)

Laravel Envoy, uzak sunucularýnýzda ortak görevler tanýmlanmasý için temiz, minimal bir sözdizimi saðlar. [Blade](/docs/templates#blade-templating) tarzý bir sözdizimi kullanarak yayýmlama, Artisan komutlarý ve baþka þeyler için kolayca görevler inþa edebilirsiniz.

> **Not:** Envoy, PHP versiyon 5.4 veya daha üstünü gerektirir ve sadece Mac / Linux iþletim sistemlerinde çalýþýr.

<a name="envoy-installation"></a>
### Yükleme

Ýlk olarak Envoy [Phar arþivini](https://github.com/laravel/envoy/raw/master/envoy.phar) indirin ve eriþim kolaylýðý için onu `envoy` olarak `/usr/local/bin` konumuna koyun. Görevleri çalýþtýrabilmeniz için, bu `envoy` dosyasýna çalýþtýrma izinleri vermeniz gerekebilir.

Sonra da, projenizin kökünde bir `Envoy.blade.php` dosyasý oluþturun. Ýþte baþlayabileceðiniz bir örnek:

	@servers(['web' => '192.168.1.1'])

	@task('falan', ['on' => 'web'])
		ls -la
	@endtask

Görebileceðiniz gibi, dosyanýn en üstünde bir `@servers` dizisi tanýmlanýr. Bu sunucularý, task (görev) deklarasyonlarýnýzýn `on` seçeneðinde refere edebilirsiniz. Bu `@task` deklarasyonlarýnýzýn içerisine, task çalýþtýrýldýðý zaman sunucunuzda çalýþtýrýlacak olan Bash kodunu koyacaksýnýz.

Bir iskelet Envoy dosyasýný kolayca oluþturmak için `init` komutu kullanýlabilir:

	envoy init user@192.168.1.1

<a name="envoy-running-tasks"></a>
### Görevlerin Çalýþtýrýlmasý

Bir görevi çalýþtýrmak için Envoy yüklemenizin `run` komutunu kullanýn:

	envoy run falan

Eðer gerekliyse, komut satýrý seçeneklerini kullanarak Envoy dosyasýna deðiþkenler geçebilirsiniz:

	envoy run deploy --branch=master

Bu seçenekleri, kullandýðýnýz Blade sözdizimi aracýlýðýyla kullanabilirsiniz:

	@servers(['web' => '192.168.1.1'])

	@task('deploy', ['on' => 'web'])
		cd site
		git pull origin {{ $branch }}
		php artisan migrate
	@endtask

#### Bootstrapping

Envoy dosyasýnýn içinde deðiþkenler deklare etmek ve genel PHP iþi yapmak için ```@setup``` direktifini kullanabilirsiniz:

	@setup
		$now = new DateTime();

		$environment = isset($env) ? $env : "testing";
	@endsetup

Ayrýca bir PHP dosyasý include etmek için ```@include``` kullanabilirsiniz:

	@include('vendor/autoload.php');

<a name="envoy-multiple-servers"></a>
### Birden Çok Sunucu

Bir görevi birden çok sunucuda kolaylýkla çalýþtýrabilirsiniz. Sadece task deklarasyonunda sunucularý listeleyin:

	@servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

	@task('deploy', ['on' => ['web-1', 'web-2']])
		cd site
		git pull origin {{ $branch }}
		php artisan migrate
	@endtask

Ön tanýmlý olarak, ilgili görev her bir sunucuda seri olarak çalýþtýrýlacaktýr. Yani, görev bir sonraki sunucuda çalýþmaya baþlamadan önce, önceki çalýþmasýný tamamlayacaktýr.

<a name="envoy-parallel-execution"></a>
### Paralel Çalýþtýrma

Eðer bir görevi birden çok sunucuda paralel olarak çalýþtýrmak istiyorsanýz, yapmanýz gereken tek þey task deklarasyonunuza `parallel` seçeneðini eklemektir:

	@servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

	@task('deploy', ['on' => ['web-1', 'web-2'], 'parallel' => true])
		cd site
		git pull origin {{ $branch }}
		php artisan migrate
	@endtask

<a name="envoy-task-macros"></a>
### Task Makrolarý

Makrolar basit bir komut kullanarak sýralý bir biçimde çalýþacak bir görev kümesi tanýmlamanýza imkan verirler. Örneðin:

	@servers(['web' => '192.168.1.1'])

	@macro('deploy')
		falan
		filan
	@endmacro

	@task('falan')
		echo "MERHABA"
	@endtask

	@task('filan')
		echo "DÜNYA"
	@endtask

Artýk bu `deploy` makrosu tek, basit bir komut aracýlýðý ile çalýþtýrýlabilecektir:

	envoy run deploy

<a name="envoy-notifications"></a>
<a name="envoy-hipchat-notifications"></a>
### Bildirimler

#### HipChat

Bir görevi çalýþtýrdýktan sonra, basit `@hipchat` direktifini kullanarak ekibinizin HipChat odasýna bir bildirim gönderebilirsiniz:

	@servers(['web' => '192.168.1.1'])

	@task('falan', ['on' => 'web'])
		ls -la
	@endtask

	@after
		@hipchat('token', 'room', 'Envoy')
	@endafter

Ayrýca hipchat odasýna, özel bir mesaj da belirtebilirsiniz. ```@setup``` içinde deklare edilen veya ```@include``` ile dahil edilen her deðiþkenin mesajda kullanýlmasý mümkündür:

	@after
		@hipchat('token', 'room', 'Envoy', "$task ran on [$environment]")
	@endafter

Bu, ekibinizi sunucu üzerinde çalýþtýrýlan görevler hakkýnda haberdar tutmak için inanýlmaz basit bir yoludur.

#### Slack

[Slack'a](https://slack.com) bir bildirim göndermek için aþaðýdaki sözdizimi kullanýlabilir:

	@after
		@slack('team', 'token', 'channel')
	@endafter

<a name="envoy-updating-envoy"></a>
### Envoy'un Güncellenmesi

Envoy'u güncellemek için, tek yapacaðýnýz `self-update` komutunu çalýþtýrmaktýr:

	envoy self-update

Eðer Envoy yüklediðiniz yer `/usr/local/bin` ise, `sudo` kullanmanýz gerekebilir:

	sudo envoy self-update