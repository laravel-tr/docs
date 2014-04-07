# IoC Konteyneri

- [Giriş](#introduction)
- [Temel Kullanım](#basic-usage)
- [Bağlamaların Kayda Geçirileceği Yer](#where-to-register)
- [Otomatik Çözümleme](#automatic-resolution)
- [Pratik Kullanım](#practical-usage)
- [Hizmet Sağlayıcıları](#service-providers)
- [Konteyner Olayları](#container-events)

<a name="introduction"></a>
## Giriş

Laravel'in "inversion of control (kontrolün tersine çevrilmesi)" konteyneri, sınıf bağımlılıklarının yönetiminde güçlü bir araçtır. Bağımlılık enjeksiyonu ağır kodlanmış sınıf bağımlılıklarının kaldırılması için bir yöntemdir. Bunun yerine, bağımlılıklar çalışma zamanında enjekte edilmekte, bağımlılık işlemleri kolayca takas edilebildiği için daha büyük esneklik sağlamaktadır.

Laravel IoC konteyner'inin anlaşılması hem güçlü, büyük bir uygulama oluşturmak için hem de Laravel'in kendi çekirdeğine katkıda bulunmak için esastır.

<a name="basic-usage"></a>
## Temel Kullanım

IoC konteyneri bağımlılıkları iki yolla çözebilmektedir: ya Closure geri çağrıları yoluyla ya da otomatik çözülüm yoluyla. Önce Closure geri çağrılarını ele alalım. Birincisi, bir "tip", konteynere bağlanabilir:

#### Bir Tipin Konteynere Bağlanması

	App::bind('falan', function($app)
	{
		return new FalanFilan;
	});

#### Bir Tipin Konteynerden Çözümlenmesi

	$deger = App::make('falan');

`App::make` metodu çağrıldığı zaman, ilgili Closure callback'i çalıştırılacak ve sonuç döndürülecektir.

Bazen, konteyner içine sadece bir kez çözümlenmesi ve konteynere sonraki çağrılarda aynı olgunun döndürülmesi gereken bir şeyler bağlamak isteyebilirsiniz:

#### Konteynere "Paylaşılan" Bir Tip Bağlama

	App::singleton('falan', function()
	{
		return new FalanFilan;
	});

`instance` metodunu kullanarak, konteynere mevcut bir nesne olgusunu da bağlayabilirsiniz:

#### Mevcut Bir Olgunun Konteynere Bağlanması

	$falan = new Falan;

	App::instance('falan', $falan);

<a name="where-to-register"></a>
## Bağlamaların Kayda Geçirileceği Yer

IoC bağlamaları tıpkı olay işleyiciler ve rota filtreleri gibi genelde "bootstrap (önce yüklenmesi gereken) kodu" başlığı altına düşer. Başka bir deyişle, bunlar uygulamanızı istekleri gerçekten işlemeye hazırlarlar ve genel olarak bunların bir rota veya controller gerçekten çağrılmadan önce çalıştırılması gereklidir. Diğer çoğu bootstrap koduna benzer olarak, IoC bağlamalarının kayda geçirilmesi için bir seçenek her zaman olduğu gibi `start` dosyalarıdır. Alternatif olarak, bir `app/ioc.php` (dosya adının ne olduğu önemli değildir) dosyası oluşturabilir ve bu dosyayı `start` dosyanızdan "require" yapabilirsiniz.

Eğer uygulamanızda çok büyük bir sayıda IoC bağlaması varsa veya siz basitçe IoC bağlamalarınızı kategorilere göre ayrı dosyalarda organize etmek istiyorsanız, bağlamalarınızı bir [servis sağlayıcısında](#service-providers) kayda geçirebilirsiniz.

<a name="automatic-resolution"></a>
## Otomatik Çözümleme

IoC konteyneri birçok durumda hiçbir yapılandırmaya gerek kalmadan sınıfları çözümleyecek kadar güçlüdür. Örneğin:

#### Bir Sınıfın Çözümlenmesi

	class FalanFilan {

		public function __construct(Baz $baz)
		{
			$this->baz = $baz;
		}

	}

	$falanFilan = App::make('FalanFilan');

Dikkat ederseniz, FalanFilan sınıfını konteynerde kayıt etmemiş olsak bile konteyner bu sınıfı hala çözümleyecek, hatta `Baz` bağımlılığını otomatik olarak enjekte edebilecektir!

Bir tip konteynerde bağlı olmadığı durumlarda, sınıfı görmek ve sınıf yapıcısının tip dayatmalarını okumak için PHP'nin Reflection araçlarını kullanacaktır. Konteyner bu bilgiyi kullanmak suretiyle sınıfın bir olgusunu otomatik olarak inşa edecektir.

Buna karşın, bazı durumlarda, bir sınıf "somut tipte" olmayıp, arayüz tatbikatına (implementasyonuna) bağımlı olabilir. Böyle olduğu takdirde, hangi arayüz tatbikatının enjekte edileceği konusunda konteyneri bilgilendirmek için `App::bind` metodu kullanılmalıdır:

#### Bir Implementasyona Bir Interface Bağlanması

	App::bind('UyeRepositoryInterface', 'DbUyeRepository');

Şimdi şu denetçiyi ele alalım:

	class UyeController extends BaseController {

		public function __construct(UyeRepositoryInterface $uyeler)
		{
			$this->uyeler = $uyeler;
		}

	}

Biz `UyeRepositoryInterface`'i somut bir tipe bağlamış olduğumuz için, UyeController oluşturulduğu zaman `DbUserRepository` otomatik olarak bu controllere enjekte edilecektir.

<a name="practical-usage"></a>
## Pratik Kullanım

Laravel uygulamanızın esneklik ve test edilebilirliğini artırmak amacıyla IoC konteyneri kullanmak için çeşitli fırsatlar sağlar. En başta gelen örnek, denetçilerin çözümlenmesidir. Bütün denetçiler IoC konteyneri tarafından çözümlenir, yani bir kontroller sınıfının yapıcı metodunda tip dayatmalı bağımlılıklar verebilirsiniz ve bunlar otomatik olarak enjekte edilecektir.

#### Tip Dayatmalı Controller Bağımlılıkları

	class SiparisController extends BaseController {

		public function __construct(SiparisRepository $siparisler)
		{
			$this->siparisler = $siparisler;
		}

		public function getIndex()
		{
			$all = $this->siparisler->all();

			return View::make('siparisler', compact('all'));
		}

	}

Bu örnekteki `SiparisRepository` sınıfı kontroller'e otomatik olarak enjekte edilecektir. Bu şu anlama gelir: [unit testi](/docs/testing) sırasında "sahte" bir `SiparisRepository` konteynere bağlanabilir ve denetçiye enjekte edilebilir, böylece sorunsuz bir veritabanı katmanı etkileşimi mümkün olur.

[Filtreler](/docs/routing#route-filters), [kompozitörler](/docs/responses#view-composers) ve [olay işleyicileri](/docs/events#using-classes-as-listeners) de IoC konteynerinde çözülebilirler. Bunları kayda geçirdiğiniz zaman, sadece kullanılması gereken sınıfın adını vermeniz yeterlidir:

#### Diğer IoC Kullanım Örnekleri

	Route::filter('falan', 'FalanFilter');

	View::composer('falan', 'FalanComposer');

	Event::listen('falan', 'FalanHandler');

<a name="service-providers"></a>
## Hizmet Sağlayıcıları

Hizmet Sağlayıcıları birbirine yakın IoC kayıtlarını tek bir yerleşimde gruplamak için harika bir yoldur. Bunları uygulamanızdaki bileşenleri önceden yüklemenin bir yolu olarak düşünün. Bir hizmet sağlayıcısının içinde özel kimlik doğrulama sürücünüzü kayda geçirebilir, uygulamanızın ambar sınıflarını IoC konteyneri ile kayda geçirebilir, hatta özel bir Artisan komutu dahi kurabilirsiniz.

Aslında, çekirdek Laravel bileşenlerinin pek çoğu hizmet sağlayıcıları içermektedir. Uygulamanızdaki kayıtlı hizmet sağlayıcılarının hepsi, `app/config/app.php` yapılandırma dosyasının `providers` dizisinde listelenmektedir.

Bir hizmet sağlayıcı oluşturmak için, sadece `Illuminate\Support\ServiceProvider` sınıfını genişletin ve bir `register` metodu tanımlayın:

#### Bir Hizmet Sağlayıcı Tanımlanması

	use Illuminate\Support\ServiceProvider;

	class FalanServiceProvider extends ServiceProvider {

		public function register()
		{
			$this->app->bind('falan', function()
			{
				return new Falan;
			});
		}

	}

Bu `register` metodunda, uygulama IoC konteynerinin `$this->app` özelliği aracılığıyla kullanılabildiğini unutmayın. Bir sağlayıcı oluşturdunuz ve uygulamanızla kayda geçirmeye hazırsanız, yapmanız gereken şey onu `app` yapılandırma dosyanızdaki `providers` dizisine eklemektir.

Bir hizmet sağlayıcıyı `App::register` metodunu kullanarak çalışma zamanında da kayda geçirebilirsiniz:

#### Bir Hizmet Sağlayıcının Çalışma Zamanında Kayda Geçirilmesi

	App::register('FalanServiceProvider');

<a name="container-events"></a>
## Konteyner Olayları

Konteyner ne zaman bir nesne çüzümlese bir olay ateşler. `resolving` metodunu kullanarak bu olayı dinleyebilirsiniz:

#### Bir Resolving Dinleyicisinin Kayda Geçirilmesi

	App::resolvingAny(function($object)
	{
		//
	});

	App::resolving('falan', function($falan)
	{
		//
	});

Çözülen nesnenin geri çağrıya geçirileceğini unutmayın.