# Laravel Cashier

- [Giriş](#introduction)
- [Yapılandırma](#configuration)
- [Bir Plana Abone Olunması](#subscribing-to-a-plan)
- [Kredi Kartsız](#no-card-up-front)
- [Aboneliklerin Takas Edilmesi](#swapping-subscriptions)
- [Abonelik Miktarı](#subscription-quantity)
- [Bir Aboneliğin İptal Edilmesi](#cancelling-a-subscription)
- [Bir Aboneliğe Geri Dönülmesi](#resuming-a-subscription)
- [Abonelik Durumunun Yoklanması](#checking-subscription-status)
- [Başarısız Ödemelerin Halledilmesi](#handling-failed-payments)
- [Faturalar](#invoices)

<a name="introduction"></a>
## Giriş

Laravel Cashier [Stripe'in](https://stripe.com) abonelik faturalama hizmetleri için anlamlı, akıcı bir arayüz sağlar. Sizin yazmaktan ürktüğünüz klişe abonelik faturalama kodunun hemen tümünü halleder. Cashier, temel abonelik yönetimine ek olarak kuponları, abonelik takasını, abonelik "miktarlarını", ödemesiz dönemlerin iptal edilmesini halledebilir ve hatta fatura PDF'leri üretebilir.

<a name="configuration"></a>
## Yapılandırma

#### Composer

Öncelikle, `composer.json` dosyanıza Cashier paketini ekleyin:

	"laravel/cashier": "~2.0"

#### Service Provider

Daha sonra, `app` yapılandırma dosyanızda `Laravel\Cashier\CashierServiceProvider`i kayda geçirin.

#### Migration

Cashier kullanabilmemiz için, veritabanımıza birkaç sütun eklememiz gerekiyor. Endişe etmeyin, gerekli sütunları ekleyecek bir migrasyon oluşturmak için `cashier:table` Artisan komutunu kullanabilirsiniz. Bu migrasyonu oluşturduktan sonra basitçe `migrate` komutunu çalıştırın.

#### Model Ayarı

Ondan sonra da model tanımlamanıza BillableTrait ve uygun tarih değiştiricilerini ekleyin:

	use Laravel\Cashier\BillableTrait;
	use Laravel\Cashier\BillableInterface;

	class User extends Eloquent implements BillableInterface {

		use BillableTrait;

		protected $dates = ['trial_ends_at', 'subscription_ends_at'];

	}

#### Stripe Key

Son olarak, bootstrap dosyalarınızın birinde Stripe anahtarınızı ayarlayın:

	User::setStripeKey('stripe-key');

<a name="subscribing-to-a-plan"></a>
## Bir Plana Abone Olunması

Bir model olgusuna sahip olduktan sonra o kullanıcıyı verilen bir Stripe planına kolaylıkla abone edebilirsiniz:

	$user = User::find(1);

	$user->subscription('monthly')->create($creditCardToken);

Bir abonelik oluştururken bir kupon uygulamak isterseniz, `withCoupon` metodunu kullanabilirsiniz:

	$user->subscription('monthly')
	     ->withCoupon('code')
	     ->create($creditCardToken);

Bu `subscription` metodu ilgili Stripe aboneliğini otomatik olarak oluşturacaktır, bunun yanında veritabanınızı Stripe müşteri ID'si ve ilgili diğer faturalama bilgisiyle güncelleyecektir.

Eğer planınız bir deneme süresi ise, abonelikten sonra modelinizde deneme bitiş tarihi (trial end date) ayarladığınızdan emin olun:

	$user->trial_ends_at = Carbon::now()->addDays(14);

	$user->save();

<a name="no-card-up-front"></a>
## Kredi Kartsız

Eğer uygulamanız kredi kartı olmaksızın bedava bir deneme teklif ediyorsa, modelinizde `cardUpFront` özelliğini `false` olarak ayarlayın:

	protected $cardUpFront = false;

Hesap oluşturulmasında, modelde deneme bitiş tarihi ayarladığınızdan emin olun:

	$user->trial_ends_at = Carbon::now()->addDays(14);

	$user->save();

<a name="swapping-subscriptions"></a>
## Aboneliklerin Takas Edilmesi

Bir kullanıcıyı yeni bir aboneliğe takas etmek için, `swap` metodunu kullanın:

	$user->subscription('premium')->swap();

Eğer kullanıcı deneme (trial) durumundaysa, deneme normal şekilde sürdürülecektir. Ayrıca abonelik için eğer bir "miktar (quantity)" mevcutsa, miktar da sürdürülecektir.

<a name="subscription-quantity"></a>
## Abonelik Miktarı

Bazen abonelikler "miktar" ile etkilenir. Örneğin, uygulamanız bir hesap üzerinde kullanıcı başına ayda $10 ücretlendirme yapabilir. Abonelik miktarını kolayca artırmak ve azaltmak için `increment` ve `decrement` metodlarını kullanın:

	$user = User::find(1);

	$user->subscription()->increment();

	// Aboneliğin mevcut miktarına beş ekle...
	$user->subscription()->increment(5)

	$user->subscription()->decrement();

	// Aboneliğin mevcut miktarından beş çıkar...
	$user->subscription()->decrement(5)

<a name="cancelling-a-subscription"></a>
## Bir Aboneliğin İptal Edilmesi

Bir aboneliğin iptal edilmesi parkta bir yürüyüştür:

	$user->subscription()->cancel();

Bir abonelik iptal edildiği zaman, Cashier veritabanınızdaki `subscription_ends_at` sütununu otomatik olarak ayarlayacaktır. Bu sütun, `subscribed` metodunun ne zaman `false` döndürmeye başlaması gerektiğini bilmek için kullanılır. Örneğin, eğer bir müşteri 1 Martta bir aboneliği iptal ederse ama aboneliğin sona ermesi 5 Marta kadar planlanmamışsa, `subscribed` metodu 5 Marta kadar `true` döndürmeye devam edecektir.

<a name="resuming-a-subscription"></a>
## Bir Aboneliğe Geri Dönülmesi

Eğer bir kullanıcı aboneliğini iptal etmiş ve bu aboneliğe kaldığı yerden devam etmesini istiyorsanız, `resume` metodunu kullanın:

	$user->subscription('monthly')->resume($creditCardToken);

Eğer kullanıcı bir aboneliği iptal eder ve daha sonra bu abonelik tam olarak sona ermeden geri dönerse, onlara hemen fatura edilmeyecektir. Abonelikleri sadece tekrar etkinleştirilecektir ve orijinal faturalama döngüsüne göre fatura edilecektir.

<a name="checking-subscription-status"></a>
## Abonelik Durumunun Yoklanması

Bir kullanıcının uygulamanıza abone olduğunu doğrulamak için, `subscribed` komutunu kullanın:

	if ($user->subscribed())
	{
		//
	}

Bu `subscribed` metodu bir rota filtresi için harika bir adaydır:

	Route::filter('subscribed', function()
	{
		if (Auth::user() && ! Auth::user()->subscribed())
		{
			return Redirect::to('billing');
		}
	});

Ayrıca, `onTrial` metodunu kullanmak suretiyle kullanıcının hala deneme süresinde olup olmadığını (uygunsa) da tayin edebilirsiniz:

	if ($user->onTrial())
	{
		//
	}

Kullanıcının daha önce aktif bir abone olduğunu ama aboneliğini iptal etmiş olduğunu tayin etmek için `cancelled` metodunu kullanabilirsiniz:

	if ($user->cancelled())
	{
		//
	}

Ayrıca, bir kullanıcının aboneliğini iptal etmiş ama hala aboneliği tam sona erinceye kadar "yetkisiz kullanım süresinde (grace period)" olup olmadıklarını da belirleyebilirsiniz. Örneğin, bir kullanıcı 10 Martta sona ereceği planlanmış bir aboneliği 5 Martta iptal ederse, bu kullanıcı 10 Marta kadar "yetkisiz kullanım süresindedir". `Subscribed` metodunun bu zaman süresinde hala `true` döndürdüğüne dikkat ediniz.

	if ($user->onGracePeriod())
	{
		//
	}

Bir kullanıcının uygulamanızdaki bir plana hiç abone olup olmadığını tayin etmek için `everSubscribed` metodu kullanılabilir:

	if ($user->everSubscribed())
	{
		//
	}

<a name="handling-failed-payments"></a>
## Başarısız Ödemelerin Halledilmesi

Şayet bir müşterinin kredi kartı süresi dolarsa ne olur? Endişeye gerek yok - Cashier sizin için müşterinin üyeliğini kolaylıkla iptal edebileceğiniz bir Webhook controller içermektedir. Sadece bir rotada bu controlleri belirtin:

	Route::post('stripe/webhook', 'Laravel\Cashier\WebhookController@handleWebhook');

Hepsi bu kadar! Gerçekleşmemiş ödemeler bu controller tarafından yakalanacak ve halledilecektir. Bu controller üç başarısız ödeme girişiminden sonra ilgili müşterinin aboneliğini iptal edecektir. Bu örnekteki `stripe/webhook` URI sadece örnek içindir. Kendi Stripe ayarlarınızda bu URI'ı yapılandırmanız gerekir.

İşlemek istediğiniz başka Stripe webhook olaylarına sahipseniz, Webhook controller'i basitçe genişletin:

	class WebhookController extends Laravel\Cashier\WebhookController {

		public function handleWebhook()
		{
			// Diğer olayları hallet...

			// Başarısız ödeme kontrolü için son çare...
			return parent::handleWebhook();
		}

	}

> **Not:** Webhook controller veritabanınızdaki abonelik bilgilerini güncellemeye ek olarak Stripe API aracılığıyla aboneliği de iptal edecektir.

<a name="invoices"></a>
## Faturalar

`invoices` metodunu kullanarak bir kullanıcının faturalarından oluşan bir diziyi kolaylıkla elde edebilirsiniz:

	$invoices = $user->invoices();

Müşterinin faturalarını listelerken, ilgili fatura bilgisini göstermek için şu helper metodlarını kullanabilirsiniz:

	{{ $invoice->id }}

	{{ $invoice->dateString() }}

	{{ $invoice->dollars() }}

Bir faturanın indirilebilir bir PDF'sini üretmek için `downloadInvoice` metodunu kullanın. Evet, bu gerçekten bu kadar kolaydır:

	return $user->downloadInvoice($invoice->id, [
		'vendor'  => 'Şirketiniz',
		'product' => 'Ürününüz',
	]);
