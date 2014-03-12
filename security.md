# Güvenlik

- [Yapılandırma](#configuration)
- [Şifrelerin Saklanması](#storing-passwords)
- [Kullanıcı Kimliklerinin Doğrulanması](#authenticating-users)
- [Elle Kullanıcı Girişi](#manually)
- [Rotaların Korunması](#protecting-routes)
- [HTTP Temel Kimlik Doğrulaması](#http-basic-authentication)
- [Şifre Hatırlatıcıları & Sıfırlama](#password-reminders-and-reset)
- [Kriptolama](#encryption)

<a name="configuration"></a>
## Yapılandırma

Laravel, kimlik doğrulanması işlerini çok basit hale getirmeyi amaçlamaktadır. Aslında, hemen her şey hazır yapılandırılmış durumdadır. Kimlik doğrulaması yapılandırma dosyası `app/config/auth.php` yerleşiminde bulunmaktadır ve kimlik doğrulama araçlarının davranışlarına nasıl ince ayarlar yapılacağı üzerine iyi belgelenmiş çeşitli seçenekler barındırır.

Ön tanımlı olarak, Laravel `app/models` dizininde bir `User` modeli içermektedir ve bu model ön tanımlı Eloquent kimlik doğrulama sürücüsü ile kullanıma hazırdır. Bu modelin şemasını oluştururken şifre alanının en az 60 karakter olmasını temin etmeniz gerektiğini unutmayın.

Şayet sizin uygulamanız Eloquent kullanmıyorsa, Laravel sorgu oluşturucusunu kullanan `database` kimlik doğrulama sürücüsünü kullanabilirsiniz.

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

Bir kullanıcının uygulamanıza zaten giriş yapmış olduğunu tayin etmek için `check` metodunu kullanabilirsiniz:

#### Bir Kullanıcının Doğrulanmış Olup Olmadığının Tayin Edilmesi

	if (Auth::check())
	{
		// Kullanıcı giriş yapmıştır...
	}

Şayet uygulamanıza "beni hatırla" işlevselliği vermek istiyorsanız, `attempt` metoduna ikinci parametre olarak `true` geçebilirsiniz, böylece bu kullanıcı süresiz olarak "doğrulanmış" tutulacaktır (yada manuel olarak çıkış işlemi yapıncaya kadar):

#### Bir Kullanıcının Kimliğinin Doğrulanması ve "Hatırlanması"

	if (Auth::attempt(array('email' => $email, 'password' => $parola), true))
	{
		// Bu kullanıcı hatırlanacak...
	}

**Not:** `attempt` metodu `true` döndürürse, kullanıcı uygulamanıza girmiş kabul edilir.

#### Determining If User Authed Via Remember
If you are "remembering" user logins, you may use the `viaRemember` method to determine if the user was authenticated using the "remember me" cookie:

	if (Auth::viaRemember())
	{
		//
	}

Kimlik doğrulama sorgusuna ekstra şartlar da ekleyebilirsiniz:

#### Bir Kullanıcının Ek Şartlara Göre Doğrulanması

    if (Auth::attempt(array('email' => $email, 'password' => $parola, 'aktif' => 1)))
    {
        // Bu kullanıcı aktiftir, üyeliği askıya alınmış değildir ve mevcuttur.
    }

> **Note:** For added protection against session fixation, the user's session ID will automatically be regenerated after authenticating.

Bir kullanıcının kimliği doğrulandıktan sonra, bu kullanıcının model / kaydına ulaşabilirsiniz:

#### Login Yapmış Kullanıcıya Erişme

	$email = Auth::user()->email;

Bir kullanıcıyı sadece ID'i ile uygulamanıza giriş yaptırtmak için `loginUsingId` metodunu kullanın:

	Auth::loginUsingId(1);

`validate` metodu gerçekte uygulamaya giriş yapılmaksızın bir kullanıcının kimlik bilgilerinin geçerlilik denetiminden geçirilmesine imkan verir:

#### Login Olmaksızın Kullanıcı Bilgilerinin Geçerlilik Denetimi

	if (Auth::validate($kimlikbilgileri))
	{
		//
	}

Bir kullanıcıyı uygulamanıza tek bir istek için giriş yapmak için de `once` metodunu kullanabilirsiniz. Bu durumda oturum veya çerezler kullanılmayacaktır.

#### Bir Kullanıca Tek Bir İstek İçin Giriş Yapma

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
## HTTP Temel Kimlik Doğrulaması

HTTP Temel Kimlik Doğrulaması, kullanıcıları özel bir "giriş" sayfası açmadan uygulamanıza giriş yapabilmeleri için hızlı bir yoldur. Bunun için, rotanıza `auth.basic` filtresi tutturun:

#### HTTP Temel İle Bir Rotanın Korunması

	Route::get('profil', array('before' => 'auth.basic', function()
	{
		// Sadece kimliği doğrulanmış üyeler girebilir...
	}));

Ön tanımlı olarak, bu `basic` filtresi kimlik doğrulaması yaparken kullanıcı kaydındaki `email` sütununu kullanacaktır. Siz başka bir sütunu kullanmak istiyorsanız, `basic` metoduna birinci parametre olarak bu sütunun adını geçirin:

	Route::filter('auth.basic', function()
	{
		return Auth::basic('username');
	});

HTTP Basit Kimlik Doğrulamasını oturumda kullanıcı tanıtıcı bir çerez ayarlamadan da kullanabilirsiniz, bu daha çok API kimlik doğrulamalarında işe yarayacaktır. Bunu yapmak için, `onceBasic` metodu döndüren bir filtre tanımlayın:

#### Durum Bilgisi Olmaksızın Bir HTTP Basit Filtresi Ayarlanması

	Route::filter('basic.once', function()
	{
		return Auth::onceBasic();
	});

If you are using PHP FastCGI, HTTP Basic authentication will not work correctly by default. The following lines should be added to your `.htaccess` file:

	RewriteCond %{HTTP:Authorization} ^(.+)$
	RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]

<a name="password-reminders-and-reset"></a>
## Şifre Hatırlatıcıları & Sıfırlama

### Model & Table

Çoğu web uygulaması, kullanıcılarına unutulmuş şifrelerini sıfırlayacak bir yol verir. Her uygulamada bunu tekrar tekrar  yapmaya zorlamak yerine Laravel size şifre hatırlatıcı mektup gönderme ve şifre sıfırlaması yapılması için pratik metodlar sağlar. Başlamak için sizin `User` modelinizin `Illuminate\Auth\Reminders\RemindableInterface` sözleşmesini yerine getirdiğini doğrulayın. Tabii ki, Laravel'le gelen `User` modeli bu arayüz kontratını zaten yerine getirmektedir.

#### RemindableInterface Implementasyonu

	class User extends Eloquent implements RemindableInterface {

		public function getReminderEmail()
		{
			return $this->email;
		}

	}

Daha sonra, şifre sıfırlama jetonlarının saklanacağı bir tablo oluşturulmalıdır. Bu tablo için bir migrasyon üretmek için yapacağınız tek şey `auth:reminders` Artisan komutunu çalıştırmaktır:

#### Hatırlatıcı Tablo Migrasyonunun Üretilmesi

	php artisan auth:reminders-table

	php artisan migrate

### Password Reminder Controller

Now we're ready to generate the password reminder controller. To automatically generate a controller, you may use the `auth:reminders-controller` Artisan command, which will create a `RemindersController.php` file in your `app/controllers` directory.

	php artisan auth:reminders-controller

The generated controller will already have a `getRemind` method that handles showing your password reminder form. All you need to do is create a `password.remind` [view](/docs/responses#views). This view should have a basic form with an `email` field. The form should POST to the `RemindersController@postRemind` action.

A simple form on the `password.remind` view might look like this:

	<form action="{{ action('RemindersController@postRemind') }}" method="POST">
		<input type="email" name="email">
		<input type="submit" value="Send Reminder">
	</form>

In addition to `getRemind`, the generated controller will already have a `postRemind` method that handles sending the password reminder e-mails to your users. This method expects the `email` field to be present in the `POST` variables. If the reminder e-mail is successfully sent to the user, a `status` message will be flashed to the session. If the reminder fails, an `error` message will be flashed instead.

Within the `postRemind` controller method you may modify the message instance before it is sent to the user:

	Password::remind(Input::only('email'), function($message)
	{
		$message->subject('Password Reminder');
	});

Your user will receive an e-mail with a link that points to the `getReset` method of the controller. The password reminder token, which is used to identify a given password reminder attempt, will also be passed to the controller method. The action is already configured to return a `password.reset` view which you should build. The `token` will be passed to the view, and you should place this token in a hidden form field named `token`. In addition to the `token`, your password reset form should contain `email`, `password`, and `password_confirmation` fields. The form should POST to the `RemindersController@postReset` method.

A simple form on the `password.reset` view might look like this:

	<form action="{{ action('RemindersController@postReset') }}" method="POST">
		<input type="hidden" name="token" value="{{ $token }}">
		<input type="email" name="email">
		<input type="password" name="password">
		<input type="password" name="password_confirmation">
		<input type="submit" value="Reset Password">
	</form>

Finally, the `postReset` method is responsible for actually changing the password in storage. In this controller action, the Closure passed to the `Password::reset` method sets the `password` attribute on the `User` and calls the `save` method. Of course, this Closure is assuming your `User` model is an [Eloquent model](/docs/eloquent); however, you are free to change this Closure as needed to be compatible with your application's database storage system.

If the password is successfully reset, the user will be redirected to the root of your application. Again, you are free to change this redirect URL. If the password reset fails, the user will be redirect back to the reset form, and an `error` message will be flashed to the session.

### Password Validation

By default, the `Password::reset` method will verify that the passwords match and are >= six characters. You may customize these rules using the `Password::validator` method, which accepts a Closure. Within this Closure, you may do any password validation you wish. Note that you are not required to verify that the passwords match, as this will be done automatically by the framework.

	Password::validator(function($credentials)
	{
		return strlen($credentials['password']) >= 8;
	});

> **Note:** By default, password reset tokens expire after one hour. You may change this via the `reminder.expire` option of your `app/config/auth.php` file.

<a name="encryption"></a>
## Kriptolama

Laravel, mcrypt PHP uzantısı aracılığıyla güçlü AES-256 kriptolama imkanı sağlamaktadır:

#### Bir Değerin Kriptolanması

	$kriptolu = Crypt::encrypt('secret');

> **Not:** `app/config/app.php` dosyasının `key` seçeneğinde 32 karakterli rasgele string ayarladığınızdan emin olun. Aksi Takdirde kriptolanmış değerler güvenli olmayacaktır.

#### Kriptolu Bir Değerin Çözülmesi

	$cozuk = Crypt::decrypt($kriptoluDeger);

Ayrıca, kriptocu tarafından kullanılan cipher ve mod da ayarlayabilirsiniz

#### Cipher ve Mod Ayarlanması

	Crypt::setMode('crt');

	Crypt::setCipher($cipher);