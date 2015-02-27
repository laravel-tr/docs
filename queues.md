# Kuyruklar

- [Yapılandırma](#configuration)
- [Temel Kullanım](#basic-usage)
- [Kuyruğa Closure Fonksiyonu Ekleme](#queueing-closures)
- [Kuyruk Dinleyicileri Çalıştırma](#running-the-queue-listener)
- [Daemon Kuyruk İşçisi](#daemon-queue-worker)
- [Push Kuyrukları](#push-queues)
- [Başarısız İşler](#failed-jobs)

<a name="configuration"></a>
## Yapılandırma

Laravel'in Queue (kuyruk) bileşeni bir takım farklı kuyruk servisleri için tek bir API sağlamaktadır. Kuyruklar e-mail göndermek gibi zaman harcayan görevleri ileri bir zamana kadar ertelemenize imkan verir ve böylece uygulamanıza yapılan web istekleri büyük ölçüde hızlanır.

Kuyruk yapılandırma dosyası `config/queue.php` olarak saklanır. Bu dosyada framework'e dahil edilmiş kuyruk sürücülerinin her birisi için bağlantı yapılandırmaları bulacaksınız. Laravel'deki kuyruk sürücüleri arasında veritabanı, [Beanstalkd](http://kr.github.com/beanstalkd), [IronMQ](http://iron.io), [Amazon SQS](http://aws.amazon.com/sqs), [Redis](http://redis.io), null ve senkronize (lokal kullanım için) sürücü yer almaktadır. `null` kuyruk sürücüsü işleri kuyruğa sokar fakat bunları asla çalıştırma.

### Kuyruk Veritabanı Tablosu

Kuyruk sürücüsü olarak `veritabanı` kullanırsanız, işleri tutmak için bir veritabanı tablosuna ihtiyacınız olacak. Bu tabloyu yaratmak için bir migration oluşturmak için `queue:table` Artisan komutnu çalıştırın:

	php artisan queue:table

### Diğer Kuyruk Bağımlılıkları

Listelenen bu kuyruk sürücüleri için aşağıdaki bağımlılıklar gereklidir:

- Amazon SQS: `aws/aws-sdk-php`
- Beanstalkd: `pda/pheanstalk ~3.0`
- IronMQ: `iron-io/iron_mq`
- Redis: `predis/predis ~1.0`

<a name="basic-usage"></a>
## Temel Kullanım

#### Bir İşin Kuyruğa Sokulması

All of the queueable jobs for your application are stored in the `App\Commands` directory. You may generate a new queued command using the Artisan CLI:

	php artisan make:command SendEmail --queued

Kuyruğa yeni bir iş itmek için `Queue::push` metodunu kullanın:

	Queue::push(new SendEmail($message));

By default, the `make:command` Artisan command generates a "self-handling" command, meaning a `handle` method is added to the command itself. This method will be called when the job is executed by the queue. You may type-hint any dependencies you need on the `handle` method and the [IoC container](/docs/master/container) will automatically inject them:

	public function handle(UserRepository $users)
	{
		//
	}

If you would like your command to have a separate handler class, you should add the `--handler` flag to the `make:command` command:

	php artisan make:command SendEmail --queued --handler

The generated handler will be placed in `App\Handlers\Commands` and will be resolved out of the IoC container.

#### Bir İş İçin Kuyruk / Tüp Belirtilmesi

Bir işin gönderilmesi gereken kuyruğu / tüpü de belirtebilirsiniz:

	Queue::pushOn('emails', new SendEmail($message));

#### Birden Çok İş İçin Aynı Yükün Geçilmesi

Birkaç kuyruk işi için aynı veriyi geçmeniz gerekiyorsa, `Queue::bulk` metodunu kullanabilirsiniz:

	Queue::bulk(array(new SendEmail($message), new AnotherCommand));

#### Bir İşin Çalıştırılmasının Geciktirilmesi

Kimi zaman sıraya sokulmuş bir işin çalıştırılmasını geciktirmek isteyebilirsiniz. Örneğin, bir müşteriye kayıt olduktan 15 dakika sonra bir e-posta gönderen bir işi kuyruğa koymak isteyebilirsiniz. Bunu `Queue::later` metodunu kullanarak başarabilirsiniz:

	$date = Carbon::now()->addMinutes(15);

	Queue::later($date, new SendEmail($message));

Bu örnekte, işe atamak istediğimiz gecikme süresini belirtmek için [Carbon](https://github.com/briannesbitt/Carbon) date kitaplığını kullanıyoruz. Alternatif olarak geciktirmek istediğiniz saniye sayısını tam sayı olarak geçebilirsiniz.

#### Kuyruklar Ve Eloquent Modelleri

Eğer sizin kuyruğa alınmış işleriniz constructor'ında bir Eloquent modeli kabul ediyorsa, only the identifier for the model will be serialized onto the queue. When the job is actually handled, the queue system will automatically re-retrieve the full model instance from the database. It's all totally transparent to your application and prevents issues that can arise from serializing full Eloquent model instances.

#### İşlenmiş Bir İşin Silinmesi

Once you have processed a job, it must be deleted from the queue. If no exception is thrown during the execution of your job, this will be done automatically.

If you would like to `delete` or `release` the job manually, the `Illuminate\Queue\InteractsWithQueue` trait provides access to the queue job `release` and `delete` methods. The `release` method accepts a single value: the number of seconds you wish to wait until the job is made available again.

	public function handle(SendEmail $command)
	{
		if (true)
		{
			$this->release(30);
		}
	}

#### Releasing A Job Back Onto The Queue

IF an exception is thrown while the job is being processed, it will automatically be released back onto the queue so it may be attempted again. The job will continue to be released until it has been attempted the maximum number of times allowed by your application. The number of maximum attempts is defined by the `--tries` switch used on the `queue:listen` or `queue:work` Artisan commands.

#### Çalıştırma Girişimlerinin Sayısını Yoklama

İş işlenirken bir istisna oluşursa, otomatik olarak kuyruğa tekrar salınacaktır. `attempts` metodunu kullanarak, işi çalıştırmak için yapılmış olan girişim sayısını da yoklayabilirsiniz:

	if ($this->attempts() > 3)
	{
		//
	}

> **Not:** Your command / handler must use the `Illuminate\Queue\InteractsWithQueue` trait in order to call this method.

<a name="queueing-closures"></a>
## Kuyruğa Closure Fonksiyonu Ekleme

Kuyruğa bir Closure de push edebilirsiniz. Bu, kuyruğa sokulması gerekecek hızlı, basit görevler için çok uygundur:

#### Kuyruğa Bir Closure Sokulması

	Queue::push(function($job) use ($id)
	{
		Account::delete($id);

		$job->delete();
	});

> **Not:** Kuyruğa sokulmuş Closure'lar için nesneleri `use` direktifi aracılığıyla kullanılabilir yapmak yerine, birincil anahtarları geçmeyi ve ilgili modeli kuyruk işiniz içinden tekrar çekmeyi düşünün. Bu, beklenmedik serileştirme davranışlarını çoğu keresinde önleyecektir.

Iron.io [push kuyrukları](#push-queues) kullanılıyorken, Closure'ların kuyruğa sokulmasında daha fazla önlem almalısınız. Kuyruk mesajlarızı alan son nokta, isteğin gerçekten Iron.io'den mi geldiğini doğrulayacak bir jeton yoklaması yapmalıdır. Örneğin, sizin push kuyruk son noktanız şuna benzer bir şey olmalıdır: https://uygulamaniz.com/queue/receive?token=SecretToken. Böylece, kuyruk istek sıralamasından önce uygulamanızdaki gizli jetonun değerini kontrol edebilirsiniz.

<a name="running-the-queue-listener"></a>
## Kuyruk Dinleyicileri Çalıştırma

Laravel, kuyruğa itildikçe yeni işler çalıştıran bir Artisan görevi içermektedir. Bu görevi çalıştırmak için `queue:listen` komutunu kullanabilirsiniz:

#### Kuyruk Dinleyici Başlatılması

	php artisan queue:listen

Ayrıca dinleyicinin kullanacağı kuyruk bağlantısını da belirtebilirsiniz:

	php artisan queue:listen connection

Unutmamanız gereken şey, bu görev başlatıldıktan sonra elle durdurulana kadar çalışmaya devam edeceğidir. Kuyruk dinleyicinin çalışmayı durdurmamasından emin olmak için [Supervisor](http://supervisord.org/) gibi bir süreç monitörü kullanabilirsiniz.

Kuyruk önceliklerini ayarlamak için `listen` komutuna virgülle ayrılmış bir kuyruk bağlantıları listesi geçebilirsiniz:

	php artisan queue:listen --queue=high,low

Bu örnekte, `high-connection` üzerindeki işler, her zaman için `low-connection`'dan gelen işlere geçmeden önce yürütülecektir.

#### İş Zaman Aşımı Parametresi Belirleme

Ayrıca her işin çalışmasına izin verilecek zaman süresini (saniye cinsinden) de ayarlayabilirsiniz:

	php artisan queue:listen --timeout=60

#### Kuyruk Uyku Süresinin Belirtilmesi

Ek olarak, yeni işin eyleme alınmadan önce beklenileceği süreyi saniye cinsinden belirtebilirsiniz:

	php artisan queue:listen --sleep=5

Not: kuyruk sadece kuyrukta iş olmadığı takdirde "uyur". Eğer kuyrukta başka işler varsa, kuyruk uyumaksızın onları çalışmaya devam edecektir.

#### Kuyruktaki İlk İşin İşleme Geçirilmesi

Kuyruktaki sadece ilk sıradaki işi yürütmek için `queue:work` komutunu kullanabilirsiniz:

	php artisan queue:work

<a name="daemon-queue-worker"></a>
## Daemon Kuyruk İşçisi

`queue:work` ayrıca işlerin işlenmesinin framework tekrar boot edilmeksizin devam etmesi için kuyruk işçisinin zorlanması için bir `--daemon` seçeneği içermektedir. Bu, `queue:listen` komutuyla karşılaştırıldığında CPU kullanımında önemli bir azalmayla sonuçlanır ama yayımlama sırasında halihazırda çalışmakta olan kuyrukların drene edilmesi gerekliliği karmaşıklığını ekler.

Bir kuyruk işçisini daemon modunda başlatmak için, `--daemon` flagını kullanın:

	php artisan queue:work connection --daemon

	php artisan queue:work connection --daemon --sleep=3

	php artisan queue:work connection --daemon --sleep=3 --tries=3

Gördüğünüz gibi, `queue:work` komutu `queue:listen` için kullanılan seçeneklerin pek çoğunu desteklemektedir. Mevcut seçeneklerin tümünü görmek için php artisan help `queue:work` komutunu kullanabilirsiniz.

### Daemon Kuyruk İşçileriyle Yayımlama

Bir uygulamayı daemon kuyruk işçileri kullanarak yayımlamanın en basit yolu yayımlamanızın en başında uygulamanızı bakım (maintenance) moduna koymaktır. Bu `php artisan down` komutu kullanılarak yapılabilir. Uygulama bakım moduna alındıktan sonra, Laravel artık kuyruğa yeni işler kabul etmeyecektir ama mevcut işleri işlemeye devam edecektir.

Worker'larınızı yeniden başlatmanın en kolay yolu yayımlama scriptinize aşağıdaki komutu dahil etmektir:

	php artisan queue:restart

Bu komut tüm kuyruk işçilerine mevcut işlerini işlemeyi bitirdikten sonra yeniden başlatmaları talimatı verecektir.

> **Not:** Bu komut restart planlamak için cache sistemine dayanmaktadır. APCu default olarak CLI komutları için çalışmaz. Eğer APCu kullanıyorsanız, APCu yapılandırmanıza `apc.enable_cli=1` ekleyin.

### Daemon Kuyruk İşçileri İçin Kodlama

Daemon kuyruk işçileri her biri işlerini işlemeden önce frameworkü yeniden başlatmazlar. Bu nedenle, işlerinizi bitirmeden önce çok büyük kaynakları serbest bırakmaya özen göstermelisiniz. Örneğin, GD kitaplığıyla resim manipulasyonu yapıyorsanız, yaptıktan sonra `imagedestroy` ile belleği rahatlatmalısınız.

Benzer şekilde, uzun çalışan daemon'larla kullanıldığı zaman veritabanı bağlantınız kopabilir. Taze bir bağlantınız olmasını temin etmek için `DB::reconnect` metodunu kullanabilirsiniz.

<a name="push-queues"></a>
## Push Kuyrukları

Push kuyrukları size herhangi bir art alan veya arka plan dinleyici çalıştırmaksızın güçlü Laravel 4 kuyruk araçlarını kullanmanıza imkan verir. Push kuyrukları şu anda sadece [Iron.io](http://iron.io) sürücüsü tarafından desteklenmektedir. Başlamak için önce bir Iron.io hesabı oluşturun ve Iron kimlik bilgilerinizi `config/queue.php` yapılandırma dosyasına ekleyin.

#### Bir Push Kuyruk Aboneliğinin Kayda Geçirilmesi

Daha sonra, yeni push edilmiş kuyruk işlerini alacak bir URL son noktasını kayda geçirmek için `queue:subscribe` Artisan komutunu kullanabilirsiniz:

	php artisan queue:subscribe queue_name http://foo.com/queue/receive

Şimdi, sizin Iron panonuza giriş yaptığınız zaman, yeni push kuyruğunuzu ve abone olunan URL'yi göreceksiniz. Verilen bir kuyruk için istediğiniz kadar çok URL kaydedebilirsiniz. Sonra da, `queue/receive` son noktanız için bir rota oluşturun ve `Queue::marshal` metodundan cevap döndürün:

	Route::post('queue/receive', function()
	{
		return Queue::marshal();
	});

Doğru iş işleyici sınıfının ateşlenmesiyle `marshal` metodu ilgilenecektir. Push kuyruğundaki işleri ateşlemek için, konvansiyonal kuyruklar için kullanılan aynı `Queue::push` metodunu kullanmanız yeterlidir.

<a name="failed-jobs"></a>
## Başarısız İşler

İşler her zaman planladığımız şekilde gitmediğinden, bazen kuyruğa soktuğumuz işler başarılamaz. Dert etmeyin, bu en iyilerimizin bile başına gelir! Laravel bir işin en fazla kaç defa denenmesi gerektiğini belirtmeniz için kolay bir yol içerir. Bir iş bu girişme miktarını aştıktan sonra, `failed_jobs` (başarısız işler) tablosuna eklenecektir. Başarısız işler tablosunun adını `config/queue.php` yapılandırma dosyasında ayarlayabilirsiniz.

`failed_jobs` tablosu için bir migrasyon oluşturmak için, `queue:failed-table` komutunu kullanabilirsiniz:

	php artisan queue:failed-table

Bir işin maksimum kaç defa yapılma girişiminde bulunulacağını `queue:listen` komutunda `--tries` anahtarını kullanarak belirtebilirsiniz:

	php artisan queue:listen connection-name --tries=3

Eğer bir kuyruk işi başarısız olduğu takdirde çağrılacak bir olay kayda geçirmek isterseniz, `Queue::failing` metodunu kullanabilirsiniz. Bu olay ekibinizi bir e-posta veya [HipChat](https://www.hipchat.com) aracılığıyla bilgilendirmek için harika bir fırsattır.

	Queue::failing(function($connection, $job, $data)
	{
		//
	});

Ayrıca bir kuyruk iş sınıfına bir `failed` metodu tanımalayarak bir başarısızlık oluştuğunda belirli eylemleri gerçekleştirebilirsiniz:

	public function failed()
	{
		// Called when the job is failing...
	}

### Başarısız İşleri Yeniden Denemek

Başarısız olmuş işlerinizin tümünü görmek için `queue:failed` Artisan komutunu kullanabilirsiniz:

	php artisan queue:failed

Bu `queue:failed` komutu iş ID, bağlantı, kuyruk ve başarısızlık zamanını listeleyecektir. Bunlardan iş ID başarısız işi yeniden denemek için kullanılabilir. Örneğin, ID'si 5 olan başarısız bir işi yeniden denemek için aşağıdaki komut verilmelidir:

	php artisan queue:retry 5

Başarısız bir işi silmek isterseniz, `queue:forget` komutunu kullanabilirsiniz:

	php artisan queue:forget 5

Başarısız işlerinizin tümünü silmek için `queue:flush` komutunu kullanabilirsiniz:

	php artisan queue:flush
