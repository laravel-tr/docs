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

Olaylara abone olurken bir öncelik de belirtebilirsiniz. Daha yüksek önceliği olan dinleyiciler daha önce çalışacak, aynı önceliğe sahip dinleyiciler ise abonelik sırasına göre çalışacaklardır.

#### Bir Olaya Abone Olurken Öncelik Belirtme

	Event::listen('uye.login', 'LoginHandler', 10);

	Event::listen('uye.login', 'DigerHandler', 5);

Bazen bir olayın diğer dinleyicilere yayılmasını durdurmak isteyebilirsiniz. Dinleyicinizden `false` döndürerek bunu gerçekleştirebilirsiniz:

**Bir Olayın Yayılımının Durdurulması**

	Event::listen('uye.login', function($event)
	{
		// Olayı işle...

		return false;
	});

### Where To Register Events

So, you know how to register events, but you may be wondering _where_ to register them. Don't worry, this is a common question. Unfortunately, it's a hard question to answer because you can register an event almost anywhere! But, here are some tips. Again, like most other bootstrapping code, you may register events in one of your `start` files such as `app/start/global.php`.

If your `start` files are getting too crowded, you could create a separate `app/events.php` file that is included from a `start` file. This is a simple solution that keeps your event registration cleanly separated from the rest of your bootstrapping. If you prefer a class based approach, you may register your events in a [service provider](/docs/ioc#service-providers). Since none of these approaches is inherently "correct", choose an approach you feel comfortable with based on the size of your application.

<a name="wildcard-listeners"></a>
## Joker Dinleyiciler

Bir olay dinleyiciyi kayda geçirirken, joker dinleyicileri belirtmek üzere yıldız işareti kullanabilirsiniz:

#### Joker Olay Dinleyicilerin Kayda Geçirilmesi

	Event::listen('falan.*', function($param, $event)
	{
		// Olayı işle...
	});

Bu dinleyici `falan.` ile başlayan tüm olayları işleyecektir. Tam olay adının işleyiciye son parametre olarak geçildiğine dikkat ediniz.

You may use the `Event::firing` method to determine exactly which event was fired:

	Event::listen('foo.*', function($param)
	{
		if (Event::firing() == 'foo.bar')
		{
			//
		}
	});

<a name="using-classes-as-listeners"></a>
## Dinleyici Olarak Sınıfları Kullanma

Bazı durumlarda, bir olayı işlemek için bir bitirme fonksiyonu yerine bir sınıf kullanmak isteyebilirsiniz. Sınıf olay dinleyicileri [Laravel'in IoC konteyneri](/docs/ioc) ile çözümlenecek, böylece size dinleyicileriniz üzerinde tam bir koloni enjeksiyonu gücü verecektir.

#### Bir Sınıf Dinleyicinin Kayda Geçirilmesi

	Event::listen('uye.login', 'LoginIsleyici');

Ön tanımlı olarak, `LoginHandler` sınıfındaki `handle` metodu çağrılacaktır:

#### Bir Olay Dinleyici Sınıfının Tanımlanması

	class LoginIsleyici {

		public function handle($data)
		{
			//
		}

	}

Eğer ön tanımlı `handle` metodunu kullanmak istemiyorsanız, abone olunacak metodu belirleyebilirsiniz:

#### Hangi Metoda Abone Olunduğunun Tanımlanması

	Event::listen('uye.login', 'LoginIsleyici@onLogin');

<a name="queued-events"></a>
## Olayları Sıraya Sokma

`queue` ve `flush` metodlarını kullanarak, bir olayı hemen ateşlemeyip, ateşlenmek üzere "sıraya" sokabilirsiniz:

#### Sıralı Bir Olayın Kayda Geçirilmesi

	Event::queue('falan', array($uye));

#### Bir Olay Flusher'ın Kayda Geçirilmesi

	Event::flusher('falan', function($uye)
	{
		//
	});

Son olarak, ilgili "flusher"ı çalıştırabilir ve `flush` metodunu kullanarak sıradaki tüm olayları harekete geçirebilirsiniz:

	Event::flush('falan');

<a name="event-subscribers"></a>
## Olay Aboneleri

Olay aboneleri, sınıfın kendi içinden birden çok olaya abone olabilen sınıflardır. Aboneler bir `subscribe` metodu ile tanımlanırlar ve bu metoda parametre olarak bir olay sevkiyatçısı olgusu geçilecektir:

#### Bir Olay Abonesi Tanımlanması

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

Abone tanımlandıktan sonra, `Event` sınıfı kullanılarak kayda geçirilebilir.

#### Bir Olay Abonesinin Kayda Geçirilmesi

	$abone = new UyeOlayIsleyici;

	Event::subscribe($abone);