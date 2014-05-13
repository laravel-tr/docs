# Olaylar (Events)

- [Temel Kullanım](#basic-usage)
- [Joker Dinleyiciler](#wildcard-listeners)
- [Dinleyici Olarak Sınıfları Kullanma](#using-classes-as-listeners)
- [Olayları Sıraya Sokma](#queued-events)
- [Olay Aboneleri](#event-subscribers)

<a name="basic-usage"></a>
## Temel Kullanım

Laravel'in `Event` sınıfı, uygulamanızdaki olaylara abone olmanıza ve dinlemenize imkan veren basit bir gözlemci aracıdır.

#### Bir Olaya Abone Olma

	Event::listen('uye.login', function($uye)
	{
		$uye->last_login = new DateTime;

		$uye->save();
	});

#### Bir Olayı Ateşleme

	$olay = Event::fire('uye.login', array($uye));

#### Bir Olaya Abone Olurken Öncelik Belirtme

Olaylara abone olurken bir öncelik de belirtebilirsiniz. Daha yüksek önceliği olan dinleyiciler daha önce çalışacak, aynı önceliğe sahip dinleyiciler ise abonelik sırasına göre çalışacaklardır.

	Event::listen('uye.login', 'LoginHandler', 10);

	Event::listen('uye.login', 'DigerHandler', 5);

#### Bir Olayın Yayılımının Durdurulması

Bazen bir olayın diğer dinleyicilere yayılmasını durdurmak isteyebilirsiniz. Dinleyicinizden `false` döndürerek bunu gerçekleştirebilirsiniz:

	Event::listen('uye.login', function($event)
	{
		// Olayı işle...

		return false;
	});

### Olayların Kayda Geçirileceği Yer

Tamam, olayların nasıl kayda geçirileceğini biliyorsunuz ama onların _nerede_ kayda geçirileceğini merak ediyor olabilirsiniz. Dert etmeyin, bu çok sorulan bir şey. Ne yazık ki bu cevaplandırması zor bir soru, çünkü bir olayı neredeyse her yerde kayda geçirebilirsiniz! Fakat, işte bazı ipuçları. Aynı şekilde, diğer pek çok bootstrapping (önce yüklenen) koduna benzer olarak, olayları `app/start/global.php` gibi `start` dosyalarınızın birisinde kayda geçirebilirsiniz.

Eğer `start` dosyalarınız çok kalabalık bir hale gelirse, bir `start` dosyanızda "include" edilen ayrı bir `app/events.php` dosyası oluşturabilirsiniz. Bu, sizin olay kaydetme işinizi, geri kalan bootstrapping kodundan temiz bir şekilde ayrı tutmanın basit bir çözümüdür. Eğer sınıf temelli bir yaklaşımı tercih ederseniz, olaylarınızı bir [servis sağlayıcı](/docs/ioc#service-providers) ile kayda geçirebilirsiniz. Bu yaklaşımlardan hiçbiri "mutlak" doğru olmadığından, ugulamanızın büyüklüğüne göre rahatlık hissedeceğiniz bir yaklaşımı seçin.

<a name="wildcard-listeners"></a>
## Joker Dinleyiciler

#### Joker Olay Dinleyicilerin Kayda Geçirilmesi

Bir olay dinleyiciyi kayda geçirirken, joker dinleyicileri belirtmek üzere yıldız işareti kullanabilirsiniz:

	Event::listen('falan.*', function($param)
	{
		// Olayı işle...
	});

Bu dinleyici `falan.` ile başlayan tüm olayları işleyecektir.

Tam olarak hangi olayın ateşlendiğini tespit etmek için `Event::firing` metodunu kullanabilirsiniz:

	Event::listen('falan.*', function($param)
	{
		if (Event::firing() == 'falan.filan')
		{
			//
		}
	});

<a name="using-classes-as-listeners"></a>
## Dinleyici Olarak Sınıfları Kullanma

Bazı durumlarda, bir olayı işlemek için bir anonim fonksiyon yerine bir sınıf kullanmak isteyebilirsiniz. Sınıf olay dinleyicileri [Laravel'in IoC konteyneri](/docs/ioc) ile çözümlenecek, böylece size dinleyicileriniz üzerinde tam bir bağımlılık enjeksiyonu gücü verecektir.

#### Bir Sınıf Dinleyicinin Kayda Geçirilmesi

	Event::listen('uye.login', 'LoginIsleyici');

#### Bir Olay Dinleyici Sınıfının Tanımlanması

Ön tanımlı olarak, `LoginIsleyici` sınıfındaki `handle` metodu çağrılacaktır:

	class LoginIsleyici {

		public function handle($data)
		{
			//
		}

	}

#### Hangi Metoda Abone Olunduğunun Tanımlanması

Eğer ön tanımlı `handle` metodunu kullanmak istemiyorsanız, abone olunacak metodu belirleyebilirsiniz:

	Event::listen('uye.login', 'LoginIsleyici@onLogin');

<a name="queued-events"></a>
## Olayları Sıraya Sokma

#### Sıralı Bir Olayın Kayda Geçirilmesi

`queue` ve `flush` metodlarını kullanarak, bir olayı hemen ateşlemeyip, ateşlenmek üzere "sıraya" sokabilirsiniz:

	Event::queue('falan', array($uye));

#### Bir Olay Flusher'ın Kayda Geçirilmesi

	Event::flusher('falan', function($uye)
	{
		//
	});

Son olarak, ilgili "flusher"ı çalıştırabilir ve `flush` metodunu kullanarak sıradaki tüm olayları harekete geçirebilirsiniz:

	Event::flush('falan');

<a name="event-subscribers"></a>
## Olay Abonecileri

#### Bir Olay Abonecisi Tanımlanması

Olay abonecileri, sınıfın kendi içinden birden çok olaya abone olabilen sınıflardır. Aboneciler bir `subscribe` metodu ile tanımlanırlar ve bu metoda parametre olarak bir olay sevkiyatçısı olgusu geçilecektir:

	class UyeOlayIsleyici {

		/**
		 * Uye login olaylarını işle.
		 */
		public function onUyeLogin($event)
		{
			//
		}

		/**
		 * Uye logout olaylarını hallet.
		 */
		public function onUyeLogout($event)
		{
			//
		}

		/**
		 * Abone dinleyicilerini kayda geçir.
		 *
		 * @param  Illuminate\Events\Dispatcher  $events
		 * @return array
		 */
		public function subscribe($events)
		{
			$events->listen('uye.login', 'UyeOlayIsleyici@onUyeLogin');

			$events->listen('uye.logout', 'UyeOlayIsleyici@onUyeLogout');
		}

	}

#### Bir Olay Abonecisinin Kayda Geçirilmesi

Aboneci tanımlandıktan sonra, `Event` sınıfı kullanılarak kayda geçirilebilir.

	$aboneci = new UyeOlayIsleyici;

	Event::subscribe($aboneci);

Ayrıca abonecinizi çözümlemek için [Laravel IoC konteynerini](/docs/ioc) de kullanabilirsiniz. Bunu yapmak için `subscribe` metoduna sadece abonecinizin ismini geçiniz:

	Event::subscribe('UyeOlayIsleyici');
