# Güvenlik

- [Yapılandırma](#configuration)
- [Şifrelerin Saklanması](#storing-passwords)
- [Kullanıcı Kimliklerinin Doğrulanması](#authenticating-users)
- [Elle Kullanıcı Girişi](#manually)
- [Rotaların Korunması](#protecting-routes)
- [HTTP Basic Kimlik Doğrulaması](#http-basic-authentication)
- [Şifre Hatırlatıcıları & Sıfırlama](#password-reminders-and-reset)
- [Kriptolama](#encryption)
- [Kimlik Doğrulama Sürücüleri](#authentication-drivers)

<a name="configuration"></a>
## Yapılandırma

Laravel, kimlik doğrulanması işlerini çok basit hale getirmeyi amaçlamaktadır. Aslında, hemen her şey hazır yapılandırılmış durumdadır. Kimlik doğrulaması yapılandırma dosyası `app/config/auth.php` yerleşiminde bulunmaktadır ve kimlik doğrulama araçlarının davranışlarına nasıl ince ayarlar yapılacağı üzerine iyi belgelenmiş çeşitli seçenekler barındırır.

Ön tanımlı olarak, Laravel `app/models` dizininde bir `User` modeli içermektedir ve bu model ön tanımlı Eloquent kimlik doğrulama sürücüsü ile kullanıma hazırdır. Bu modelin şemasını oluştururken şifre alanının en az 60 karakter olmasını temin etmeniz gerektiğini unutmayın.

Şayet sizin uygulamanız Eloquent kullanmıyorsa, Laravel sorgu oluşturucusunu kullanan `database` kimlik doğrulama sürücüsünü kullanabilirsiniz.

> **Not:** Başlamadan önce `users` (veya dengi olan) tablonuzun 100 karakterlik string tipinde nullable bir `remember_token` sütunu taşıdığından emin olun. Bu sütun, uygulamanız tarafından sürdürülecek olan "remember me (beni hatırla)" session'ları için bir token saklamak amacıyla kullanılacaktır.

<a name="storing-passwords"></a>
## Şifrelerin Saklanması

Laravel'deki `Hash` sınıfı güvenli Bcrypt karıştırması (hashing) sağlar:

#### Bcrypt Kullanılarak Bir Şifrenin Karıştırılması

	$parola = Hash::make('secret');

#### Bir Şifrenin Karıştırılmışa Göre Doğrulanması

	if (Hash::check('secret', $karistirilmisParola))
	{
		// Parola doğrulanmıştır...
	}

#### Bir Şifrenin Yeniden Karıştırılması Gerekip Gerekmediğinin Yoklanması

	if (Hash::needsRehash($karistirilmis))
	{
		$karistirilmis = Hash::make('secret');
	}

<a name="authenticating-users"></a>
## Kullanıcı Kimliklerinin Doğrulanması

Bir kullanıcının uygulamanıza girişi için `Auth::attempt` metodunu kullanabilirsiniz.

	if (Auth::attempt(array('email' => $email, 'password' => $parola)))
	{
		return Redirect::intended('pano');
	}

Buradaki `email`'in gerekli bir seçenek değil, sadece örnek olsun diye kullanılmış olduğunu bilin. Veritabanınızda bir "kullanıcı adı"na ("username"e) karşılık gelen sütunu kullanmanız gerekiyor. `Redirect::intended` fonksiyonu, kullanıcıları kimlik doğrulama filtresi tarafından yakalanmadan önce erişmeye çalıştıkları URL'ye yönlendirecektir. Kullanıcının önceden girmeye çalıştığı bir url olmayan durumlarda kullanılabilsin diye bu metoda bir dönüş URI parametresi verilebilir.

`attempt` metodu çağrıldığında, `auth.attempt` [olayı](/docs/events) ateşlenecektir. Şayet kimlik doğrulama girişimi başarılı olur ve kullanıcı giriş yapmış olursa, `auth.login` olayı da ateşlenecektir.

#### Bir Kullanıcının Doğrulanmış Olup Olmadığının Tayin Edilmesi

Bir kullanıcının uygulamanıza zaten giriş yapmış olduğunu tayin etmek için `check` metodunu kullanabilirsiniz:

	if (Auth::check())
	{
		// Kullanıcı giriş yapmıştır...
	}

#### Bir Kullanıcının Kimliğinin Doğrulanması ve "Hatırlanması"

Şayet uygulamanıza "beni hatırla" işlevselliği vermek istiyorsanız, `attempt` metoduna ikinci parametre olarak `true` geçebilirsiniz, böylece bu kullanıcı süresiz olarak "doğrulanmış" tutulacaktır (ya da manuel olarak çıkış işlemi yapıncaya kadar). Tabi ki, `users` tablonuz "remember me" tokenini saklamakta kullanılacak olan string `remember_token` sütunu içermelidir.

	if (Auth::attempt(array('email' => $email, 'password' => $parola), true))
	{
		// Bu kullanıcı hatırlanacak...
	}

**Not:** `attempt` metodu `true` döndürürse, kullanıcı uygulamanıza girmiş kabul edilir.

#### Kullanıcının Remember Aracılığıyla mı Doğrulanmış Olduğunun Tayin Edilmesi
Eğer kullanıcı girişlerini "hatırlıyorsanız", bir kullanıcının "remember me" (beni hatırla) çerezi kullanılarak doğrulanmış olup olmadığını belirlemek için `viaRemember` metodunu kullanabilirsiniz:

	if (Auth::viaRemember())
	{
		//
	}

#### Bir Kullanıcının Ek Şartlara Göre Doğrulanması

Kimlik doğrulama sorgusuna ekstra şartlar da ekleyebilirsiniz:

    if (Auth::attempt(array('email' => $email, 'password' => $parola, 'aktif' => 1)))
    {
        // Bu kullanıcı aktiftir, üyeliği askıya alınmış değildir ve mevcuttur.
    }

> **Not:** Oturum sabitlemesine karşı koruma amacıyla, kimlik doğrulaması sonrasında kullanıcının oturum ID'si otomatik olarak yeniden üretilecektir.

#### Login Yapmış Kullanıcıya Erişme

Bir kullanıcının kimliği doğrulandıktan sonra, bu kullanıcının modeline / kaydına ulaşabilirsiniz:

	$email = Auth::user()->email;

Kimliği doğrulanmış bir kullanıcının ID'sini elde etmek için `id` metodunu kullanabilirsiniz:

	$id = Auth::id();

Bir kullanıcıyı sadece ID'si ile uygulamanıza giriş yaptırtmak için `loginUsingId` metodunu kullanın:

	Auth::loginUsingId(1);

#### Login Olmaksızın Kullanıcı Bilgilerinin Geçerlilik Denetimi

`validate` metodu gerçekte uygulamaya giriş yapılmaksızın bir kullanıcının kimlik bilgilerinin geçerlilik denetiminden geçirilmesine imkan verir:

	if (Auth::validate($kimlikbilgileri))
	{
		//
	}

#### Bir Kullanıca Tek Bir İstek İçin Giriş Yapma

Bir kullanıcıyı uygulamanıza tek bir istek için giriş yapmak için de `once` metodunu kullanabilirsiniz. Bu durumda oturum veya çerezler kullanılmayacaktır.

	if (Auth::once($kimlikbilgileri))
	{
		//
	}

#### Bir Kullanıcıya Uygulamadan Çıkış Yapma

	Auth::logout();

<a name="manually"></a>
## Elle Kullanıcı Girişi

Şayet, mevcut bir kullanıcı olgusunu uygulamanıza giriş yaptırmak istiyorsanız, bu olguda `login` metodunu çağırmanız yeterlidir:

	$uye = Uye::find(1);

	Auth::login($uye);

Bu metod, bir kullanıcıyı `attempt` metodu kullanarak kimlik bilgileri ile giriş yaptırmaya eşdeğerdir.

<a name="protecting-routes"></a>
## Rotaların Korunması

Belli bir rotaya sadece kimliği doğrulanmış kullanıcıların erişebilmesini sağlamak amacıyla rota filtreleri kullanılabilir. Laravel ön tanımlı olarak `auth` filtresi sağlamıştır ve `app/filters.php` içinde tanımlanmıştır.

#### Bir Rotanın Korunması

	Route::get('profil', array('before' => 'auth', function()
	{
		// Sadece kimliği doğrulanmış üyeler girebilir...
	}));

### CSRF Koruması

Laravel, uygulamanızı siteler arası istek sahtekarlıklarından (cross-site request forgeries [CSRF]) korumak için kolay bir metod sağlamaktadır.

#### Forma CSRF Jetonunun Eklenmesi

    <input type="hidden" name="_token" value="<?php echo csrf_token(); ?>">

#### Gönderilmiş CSRF Jetonunun Geçerlilik Yoklaması

    Route::post('register', array('before' => 'csrf', function()
    {
        return 'Geçerli bir CSRF jetonu verdiniz!';
    }));

<a name="http-basic-authentication"></a>
## HTTP Basic Kimlik Doğrulaması

HTTP Basic Kimlik Doğrulaması, kullanıcıları özel bir "giriş" sayfası açmadan uygulamanıza giriş yapabilmeleri için hızlı bir yoldur. Bunun için, rotanıza `auth.basic` filtresi tutturun:

#### HTTP Basic İle Bir Rotanın Korunması

	Route::get('profil', array('before' => 'auth.basic', function()
	{
		// Sadece kimliği doğrulanmış üyeler girebilir...
	}));

Ön tanımlı olarak, bu `basic` filtresi kimlik doğrulaması yaparken kullanıcı kaydındaki `email` sütununu kullanacaktır. Siz başka bir sütunu kullanmak istiyorsanız, `basic` metoduna birinci parametre olarak bu sütunun adını geçirin:

	Route::filter('auth.basic', function()
	{
		return Auth::basic('username');
	});

#### Durum Bilgisi Olmaksızın Bir HTTP Basic Filtresi Ayarlanması

HTTP Basic Kimlik Doğrulamasını oturumda kullanıcı tanıtıcı bir çerez ayarlamadan da kullanabilirsiniz, bu daha çok API kimlik doğrulamalarında işe yarayacaktır. Bunu yapmak için, `onceBasic` metodu döndüren bir filtre tanımlayın:

	Route::filter('basic.once', function()
	{
		return Auth::onceBasic();
	});

Eğer PHP FastCGI kullanıyorsanız, HTTP Basic kimlik doğrulaması ön tanımlı durumda düzgün çalışmayacaktır. Aşağıdaki satırlar `.htaccess` dosyanıza eklenmelidir:

	RewriteCond %{HTTP:Authorization} ^(.+)$
	RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]

<a name="password-reminders-and-reset"></a>
## Şifre Hatırlatıcıları & Yenileme

### Model & Table

Çoğu web uygulaması, kullanıcılarına unutulmuş şifrelerini yenileyecek bir yol verir. Her uygulamada bunu tekrar tekrar yapmaya zorlamak yerine Laravel size şifre hatırlatıcı mektup gönderme ve şifre yenilemesi yapılması için pratik metodlar sağlar. Başlamak için sizin `User` modelinizin `Illuminate\Auth\Reminders\RemindableInterface` sözleşmesini yerine getirdiğini doğrulayın. Tabii ki, Laravel'le gelen `User` modeli bu arayüz kontratını zaten yerine getirmektedir ve bu interface'i implemente etmek için gerekli metodları içermek için `Illuminate\Auth\Reminders\RemindableTrait` trait'ini kullanmaktadır.

#### RemindableInterface Implementasyonu

	use Illuminate\Auth\Reminders\RemindableTrait;
	use Illuminate\Auth\Reminders\RemindableInterface;

	class User extends Eloquent implements RemindableInterface {

		use RemindableTrait;

	}

#### Hatırlatıcı Tablo Migrasyonunun Üretilmesi

Daha sonra, şifre yenileme jetonlarının saklanacağı bir tablo oluşturulmalıdır. Bu tablo için bir migrasyon üretmek için yapacağınız tek şey `auth:reminders-table` Artisan komutunu çalıştırmaktır:

	php artisan auth:reminders-table

	php artisan migrate

### Şifre Hatırlatıcı Controller

Artık şifre hatırlatıcı controller üretmeye geçebiliriz. Bir controlleri otomatik olarak üretmek için, `auth:reminders-controller` Artisan komutunu kullanabilirsiniz, bu komut sizin `app/controllers` dizininizde bir `RemindersController.php` dosyası oluşturacaktır.

	php artisan auth:reminders-controller

Üretilen controllerde şifre hatırlatıcı formunuzu gösterilmesini halleden bir `getRemind` metodu olacaktır. Sizin yapmanız gereken tek şey bir `password.remind` [viewi](/docs/responses#views) oluşturmaktır. Bu view, bir  `email` alanı olan basit bir form olmalıdır. Bu form `RemindersController@postRemind` metoduna POST edilmelidir.

`password.remind` view'inde basit bir form bunun gibi olabilir:

	<form action="{{ action('RemindersController@postRemind') }}" method="POST">
		<input type="email" name="email">
		<input type="submit" value="Hatırlatıcı Gönder">
	</form>

Üretilen controllerde `getRemind` metoduna ek olarak kullacılarınıza şifre hatırlatıcı e-postalar gönderilmesini halleden bir `postRemind` metodu da olacaktır. Bu metod `POST` değişkenlerinde bir `email` alanı mevcut olmasını bekler. Eğer bir hatırlatıcı e-postası kullanıcıya başarıyla gönderilirse, oturuma bir `status` mesajı flaşlanacaktır. Eğer hatırlatıcı yapılamazsa o zaman bunun yerine bir `error` mesajı flaşlanacaktır.

Bu `postRemind` controller metodu içerisinde, kullanıcıya göndermeden önce mesaj olgusu üzerinde değişiklik yapabilirsiniz:

	Password::remind(Input::only('email'), function($message)
	{
		$message->subject('Şifre Hatırlatıcı');
	});

Kullanıcınız, controllerin `getReset` metodunu işaret eden bir bağlantısı olan bir e-posta alacaktır. Ayrıca, ilgili bir şifre hatırlatıcı girişimini tanımlamakta kullanılan bir şifre hatırlatıcı tokeni bu controller metoduna geçilecektir. Bu controller metodu bir `password.reset` view'i döndürecek şekilde zaten yapılandırılmıştır, ancak bu view'i sizin oluşturmanız gerekiyor. İlgili `token` view'e geçilecektir ve siz bu tokeni `token` adlı "hidden" bir form alanına koymanız gerekiyor. Sizin şifre reset (yenileme) formunuz bu `token`'e ek olarak `email`, `password` ve `password_confirmation` alanlarını da içermelidir. Bu form `RemindersController@postReset` metoduna post edilmelidir.

`password.reset` view'indeki basit bir form şöyle bir şey olabilir:

	<form action="{{ action('RemindersController@postReset') }}" method="POST">
		<input type="hidden" name="token" value="{{ $token }}">
		<input type="email" name="email">
		<input type="password" name="password">
		<input type="password" name="password_confirmation">
		<input type="submit" value="Şifreyi Yenile">
	</form>

Son olarak, veritabanındaki şifrenizin gerçekten değiştirilmesinden bu `postReset` metodu sorumludur. Bu controller eyleminde, `Password::reset` metoduna geçilen Closure, `User`'in  `password` niteliğini ayarlar ve `save` metodunu çağırır. Pek tabii, bu Closure sizin `User` modelinizin bir [Eloquent modeli](/docs/eloquent) olmasını beklemektedir; bununla birlikte siz bu Closure üzerinde uygulamanızın veritabanı depolama sistemiyle uyumlu olması için gerekli değişiklikler yapmakta özgürsünüz.

Eğer şifre başarılı bir biçimde yenilenirse, kullanıcı uygulamanızın köküne yönlendirilecektir. Aynı şekilde, siz bu redirect URL'sini de değiştirme serbestisine sahipsiniz. Eğer şifre yenileme başarısız olursa, kullanıcı tekrar reset formuna yönlendirilecektir ve session'a bir `error` mesajı flaşlanacaktır.

### Şifre Geçerlilik Denetimi

Ön tanımlı olarak, `Password::reset` metodu şifrelerin eşleşiyor ve >= altı karakter olmasını soruşturacaktır. Siz, parametre olarak bir Closure alan `Password::validator` metodunu kullanarak bu kuralları özelleştirebilirsiniz. Bu Closure içerisinde, istediğiniz her türlü şifre geçerlilik denetimini yapabilirsiniz. Şuna dikkat ediniz: şifrelerin eşleşiyor olduğunu doğrulamanız gerekmiyor, çünkü bu kısım framework tarafından otomatik olarak yapılacaktır.

	Password::validator(function($credentials)
	{
		return strlen($credentials['password']) >= 8;
	});

> **Not:** Ön tanımlı olarak, şifre yenileme tokenlerinin ömrü bir saat sonra sona erer. Siz `app/config/auth.php` dosyanızın `reminder.expire` seçeneği aracılığıyla bunu değiştirebilirsiniz.

<a name="encryption"></a>
## Kriptolama

Laravel, mcrypt PHP uzantısı aracılığıyla güçlü AES kriptolama imkanı sağlamaktadır:

#### Bir Değerin Kriptolanması

	$kriptolu = Crypt::encrypt('secret');

> **Not:** `app/config/app.php` dosyasının `key` seçeneğinde 16, 24 veya 32 karakterli rastgele bir string ayarladığınızdan emin olun. Aksi takdirde kriptolanmış değerler güvenli olmayacaktır.

#### Kriptolu Bir Değerin Çözülmesi

	$cozuk = Crypt::decrypt($kriptolu);

#### Cipher ve Mod Ayarlanması

Ayrıca, kriptocu tarafından kullanılan cipher ve mod da ayarlayabilirsiniz

	Crypt::setMode('crt');

	Crypt::setCipher($cipher);

<a name="authentication-drivers"></a>
## Kimlik Doğrulama Sürücüleri

Laravel kutusunda `database` ve `eloquent` kimlik doğrulama sürücüleriyle gelir. Diğer kimlik doğrulama sürücüleri eklenmesiyle ilgili daha fazla bilgi için [Authentication genişletme dokümantasyonu](/docs/extending#authentication) kesimini kontrol ediniz.
