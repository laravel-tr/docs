# SSH

- [Yapılandırma](#configuration)
- [Temel Kullanım](#basic-usage)
- [Görevler](#tasks)
- [SFTP Dosya İndirmeleri](#sftp-downloads)
- [SFTP Dosya Göndermeleri](#sftp-uploads)
- [Uzak Günlüklerin İzlenmesi](#tailing-remote-logs)
- [Envoy Görev Çalıştırıcısı](#envoy-task-runner)

<a name="configuration"></a>
## Yapılandırma

Laravel uzak sunuculara SSH (Secure Shell) iletişimi ve komutlar çalıştırmak için basit bir yol içerir ve uzak sunucularda çalışan Artisan görevlerini kolayca inşa etmenize imkan verir. `SSH` facade'ı uzak sunucularınıza bağlanmanız ve komutlar çalıştırmanız için erişim noktası sağlar.

Yapılandırma dosyası `app/config/remote.php` konumundadır ve uzak bağlantılarınızı yapılandırmak için gerekli tüm seçenekleri içerir. Bu dosyadaki `connections` dizisi, sürücülerinizin isimlerine göre anahtarlanmış bir listesini taşır. Bu `connections` dizisindeki host, username, password, key gibi erişim güven bilgilerini (credentials) doldurduktan sonra uzak görevleri çalıştırmaya hazır olacaksınız. Unutmayın, `SSH`, ya bir password ya da bir SSH key kullanarak kimlik doğrulaması yapabilmektedir.

> **Not:** Uzak sunucunuzda çeşitli görevleri kolayca çalıştırma ihtiyacınız mı var? [Envoy görev çalıştırıcısına](#envoy-task-runner) bir bakın!

<a name="basic-usage"></a>
## Temel Kullanım

#### Komutları Default Sunucuda Çalıştırmak

Komutlarınızı `default` uzak bağlantınızda çalıştırmak için `SSH::run` metodunu kullanın:

	SSH::run(array(
		'cd /var/www',
		'git pull origin master',
	));

#### Komutları Belirli Bir Bağlantıda Çalıştırmak

Alternatif olarak, `into` metodunu kullanmak suretiyle komutları belirli bir bağlantı üzerinde çalıştırabilirsiniz:

	SSH::into('staging')->run(array(
		'cd /var/www',
		'git pull origin master',
	));

#### Komut Çıktılarını Yakalamak

`run` metoduna bir Closure geçmek suretiyle, uzak komutlarınızın "canlı" çıktısını yakalayabilirsiniz:

	SSH::run($commands, function($line)
	{
		echo $line.PHP_EOL;
	});

## Görevler
<a name="tasks"></a>

Eğer her zaman birlikte çalışması gereken bir grup komut tanımlamanız gerekiyorsa, bir `task` (görev) tanımlamak için `define` metodunu kullanabilirsiniz:

	SSH::into('staging')->define('deploy', array(
		'cd /var/www',
		'git pull origin master',
		'php artisan migrate',
	));

Bu şekilde bir task tanımladıktan sonra, onu çalıştırmak için `task` metodunu kullanabilirsiniz:

	SSH::into('staging')->task('deploy', function($line)
	{
		echo $line.PHP_EOL;
	});

<a name="sftp-downloads"></a>
## SFTP Dosya İndirmeleri

`SSH` sınıfı `get` ve `getString` metodları kullanılarak, dosyalar indirmek için basit bir yol sağlar:

	SSH::into('staging')->get($remotePath, $localPath);

	$contents = SSH::into('staging')->getString($remotePath);

<a name="sftp-uploads"></a>
## SFTP Dosya Göndermeleri

`SSH` sınıfı aynı zamanda `put` ve `putString` metodları kullanılarak sunucuya dosyalar, hatta stringler upload etmek için de basit bir yol içerir:

	SSH::into('staging')->put($localFile, $remotePath);

	SSH::into('staging')->putString($remotePath, 'Falan');

<a name="tailing-remote-logs"></a>
## Uzak Günlüklerin İzlenmesi

Laravel sizin uzak bağlantılarınızın herhangi birindeki `laravel.log` dosyalarının izlenmesi için yararlı bir komut içermektedir. Bunun için basitçe `tail` Artisan komutunu kullanın ve izlemek istediğiniz uzak bağlantının adını belirtin:

	php artisan tail staging

	php artisan tail staging --path=/path/to/log.file

<a name="envoy-task-runner"></a>
## Envoy Görev Çalıştırıcısı

- [Yükleme](#envoy-installation)
- [Görevlerin Çalıştırılması](#envoy-running-tasks)
- [Birden Çok Sunucu](#envoy-multiple-servers)
- [Paralel Çalıştırma](#envoy-parallel-execution)
- [Task Makroları](#envoy-task-macros)
- [Bildirimler](#envoy-notifications)
- [Envoy'in Güncellenmesi](#envoy-updating-envoy)

Laravel Envoy, uzak sunucularınızda ortak görevler tanımlanması için temiz, minimal bir sözdizimi sağlar. [Blade](/docs/templates#blade-templating) tarzı bir sözdizimi kullanarak yayımlama, Artisan komutları ve başka şeyler için kolayca görevler inşa edebilirsiniz.

> **Not:** Envoy, PHP versiyon 5.4 veya daha üstünü gerektirir ve sadece Mac / Linux işletim sistemlerinde çalışır.

<a name="envoy-installation"></a>
### Yükleme

Önce, Composer `global` komutunu kullanarak Envoy'u yükleyin:

	composer global require "laravel/envoy=~1.0"

Terminalinizde `envoy` komutunu çalıştırdığınız zaman `envoy` çalıştırılabilir dosyasının bulunabilmesi için PATH ayarınızda `~/.composer/vendor/bin` dizininin yer aldığından emin olun.

Sonra da, projenizin kökünde bir `Envoy.blade.php` dosyası oluşturun. İşte başlayabileceğiniz bir örnek:

	@servers(['web' => '192.168.1.1'])

	@task('falan', ['on' => 'web'])
		ls -la
	@endtask

Görebileceğiniz gibi, dosyanın en üstünde bir `@servers` dizisi tanımlanır. Bu sunucuları, task (görev) deklarasyonlarınızın `on` seçeneğinde refere edebilirsiniz. Bu `@task` deklarasyonlarınızın içerisine, task çalıştırıldığı zaman sunucunuzda çalıştırılacak olan Bash kodunu koyacaksınız.

Bir iskelet Envoy dosyasını kolayca oluşturmak için `init` komutu kullanılabilir:

	envoy init user@192.168.1.1

<a name="envoy-running-tasks"></a>
### Görevlerin Çalıştırılması

Bir görevi çalıştırmak için Envoy yüklemenizin `run` komutunu kullanın:

	envoy run falan

Eğer gerekliyse, komut satırı seçeneklerini kullanarak Envoy dosyasına değişkenler geçebilirsiniz:

	envoy run deploy --branch=master

Bu seçenekleri, kullandığınız Blade sözdizimi aracılığıyla kullanabilirsiniz:

	@servers(['web' => '192.168.1.1'])

	@task('deploy', ['on' => 'web'])
		cd site
		git pull origin {{ $branch }}
		php artisan migrate
	@endtask

#### Bootstrapping

Envoy dosyasının içinde değişkenler deklare etmek ve genel PHP işi yapmak için ```@setup``` direktifini kullanabilirsiniz:

	@setup
		$now = new DateTime();

		$environment = isset($env) ? $env : "testing";
	@endsetup

Ayrıca bir PHP dosyası include etmek için ```@include``` kullanabilirsiniz:

	@include('vendor/autoload.php');

<a name="envoy-multiple-servers"></a>
### Birden Çok Sunucu

Bir görevi birden çok sunucuda kolaylıkla çalıştırabilirsiniz. Sadece task deklarasyonunda sunucuları listeleyin:

	@servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

	@task('deploy', ['on' => ['web-1', 'web-2']])
		cd site
		git pull origin {{ $branch }}
		php artisan migrate
	@endtask

Ön tanımlı olarak, ilgili görev her bir sunucuda seri olarak çalıştırılacaktır. Yani, görev bir sonraki sunucuda çalışmaya başlamadan önce, önceki çalışmasını tamamlayacaktır.

<a name="envoy-parallel-execution"></a>
### Paralel Çalıştırma

Eğer bir görevi birden çok sunucuda paralel olarak çalıştırmak istiyorsanız, yapmanız gereken tek şey task deklarasyonunuza `parallel` seçeneğini eklemektir:

	@servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

	@task('deploy', ['on' => ['web-1', 'web-2'], 'parallel' => true])
		cd site
		git pull origin {{ $branch }}
		php artisan migrate
	@endtask

<a name="envoy-task-macros"></a>
### Task Makroları

Makrolar basit bir komut kullanarak sıralı bir biçimde çalışacak bir görev kümesi tanımlamanıza imkan verirler. Örneğin:

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

Artık bu `deploy` makrosu tek, basit bir komut aracılığı ile çalıştırılabilecektir:

	envoy run deploy

<a name="envoy-notifications"></a>
<a name="envoy-hipchat-notifications"></a>
### Bildirimler

#### HipChat

Bir görevi çalıştırdıktan sonra, basit `@hipchat` direktifini kullanarak ekibinizin HipChat odasına bir bildirim gönderebilirsiniz:

	@servers(['web' => '192.168.1.1'])

	@task('falan', ['on' => 'web'])
		ls -la
	@endtask

	@after
		@hipchat('token', 'room', 'Envoy')
	@endafter

Ayrıca hipchat odasına, özel bir mesaj da belirtebilirsiniz. ```@setup``` içinde deklare edilen veya ```@include``` ile dahil edilen her değişkenin mesajda kullanılması mümkündür:

	@after
		@hipchat('token', 'room', 'Envoy', "$task ran on [$environment]")
	@endafter

Bu, ekibinizi sunucu üzerinde çalıştırılan görevler hakkında haberdar tutmak için inanılmaz basit bir yoludur.

#### Slack

[Slack'a](https://slack.com) bir bildirim göndermek için aşağıdaki sözdizimi kullanılabilir:

	@after
		@slack('team', 'token', 'channel')
	@endafter

<a name="envoy-updating-envoy"></a>
### Envoy'un Güncellenmesi

Envoy'u güncellemek için, tek yapacağınız `self-update` komutunu çalıştırmaktır:

	envoy self-update

Eğer Envoy yüklediğiniz yer `/usr/local/bin` ise, `sudo` kullanmanız gerekebilir:

	sudo envoy self-update