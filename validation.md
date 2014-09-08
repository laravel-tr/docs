# Geçerlilik Denetimi

- [Temel Kullanım](#basic-usage)
- [Hata Mesajlarıyla Çalışmak](#working-with-error-messages)
- [Hata Mesajları & Görünümler](#error-messages-and-views)
- [Mevcut Geçerlilik Kuralları](#available-validation-rules)
- [Duruma Göre Kurallar Ekleme](#conditionally-adding-rules)
- [Özel Hata Mesajları](#custom-error-messages)
- [Özel Geçerlilik Kuralları](#custom-validation-rules)

<a name="basic-usage"></a>
## Temel Kullanım

Laravel, `Validation` sınıfı aracığıyla verilerin geçerlilik denetimi ve geçerlilik hata mesajlarının gösterilmesi için temel ve kullanışlı bir araçla birlikte gelmektedir.

#### Temel Bir Geçerlilik Denetimi Örneği

	$gecerlilikYoklayici = Validator::make(
		array('isim' => 'Tuana Şeyma'),
		array('yas'  => 'required|min:5')
	);

Buradaki `make` metoduna geçilen ilk parametre, geçerli olup olmadığına bakılacak veridir. İkinci parametre ise, bu veriye tatbik edilecek geçerlilik kurallarıdır.

#### Kuralları Belirtmek İçin Dizi Kullanımı

Birden çok kural ya bir "pipe" karakteri (|) ile ayrılır, ya da ayrı dizi elemanları olarak verilebilir.

	$gecerlilikYoklayici = Validator::make(
		array('isim' => 'Tuana Şeyma'),
		array('yas'  => array('required', 'min:5'))
	);

#### Birçok Alana Tek Seferde Geçerlilik Denetimi Yapmak

    $validator = Validator::make(
        array(
            'name' => 'Tuana Şeyma',
            'password' => 'aksakşifre',
            'email' => 'tuanaseyma@eldem.com'
        ),
        array(
            'name' => 'required',
            'password' => 'required|min:8',
            'email' => 'required|email|unique'
        )
    );

Bir `Validator` olgusu oluşturulduktan sonra, geçerlilik denetimi yapmak için `fails` veya `passes` metodları kullanılabilir.

	if ($gecerlilikYoklayici->fails())
	{
		// İlgili veri geçerlik denetimini geçememiştir
	}

Şayet geçerlilik denetimi başarısız olursa, geçerlik yoklayıcısından hata mesajları alabilirsiniz:

	$mesajlar = $gecerlilikYoklayici->messages();

Ayrıca, başarısız olan geçerlilik kurallarına bir dizi olarak da erişebilirsiniz. Bunu yapmak için `failed` metodunu kullanabilirsiniz:

	$kalanlar = $gecerlilikYoklayici->failed();

#### Dosyalar İçin Geçerlilik Denetimi

`Validator` sınıfı dosyaların geçerliliği konusunda `size`, `mimes` ve benzeri kurallar sağlar. Dosyaları geçerlilikten geçirirken, tıpkı diğer verilerde olduğu gibi bunları da geçerlilik denetçisine parametre olarak geçersiniz.

<a name="working-with-error-messages"></a>
## Hata Mesajlarıyla Çalışmak

Bir `Validator` olgusunda `messages` metodunu çağırdıktan sonra, bir `MessageBag` olgusu alacaksınız. MessageBag sınıfında hata mesajlarıyla çalışmak için bir takım yararlı metodlar vardır.

#### Bir Alan İçin İlk Hata Mesajının Elde Edilmesi

	echo $mesajlar->first('email');

#### Bir Alan İçin Tüm Hata Mesajlarının Elde Edilmesi

	foreach ($mesajlar->get('email') as $mesaj)
	{
		//
	}

#### Tüm Alanlar İçin Tüm Hata Mesajlarının Elde Edilmesi

	foreach ($mesajlar->all() as $mesaj)
	{
		//
	}

#### Bir Alan İçin Hata Mevcut Olup Olmadığının Tespiti

	if ($mesajlar->has('email'))
	{
		//
	}

#### Bir Hata Mesajının Biçimlendirilmiş Olarak Alınması

	echo $mesajlar->first('email', '<p>:message</p>');

> **Not:** Ön tanımlı olarak, mesajlar Bootstrap'a uyumlu bir söz dizimiyle biçimlendirilir.

#### Tüm Hata Mesajlarının Biçimlendirilmiş Olarak Alınması

	foreach ($mesajlar->all('<li>:message</li>') as $mesaj)
	{
		//
	}

<a name="error-messages-and-views"></a>
## Hata Mesajları & Görünümler

Geçerlilik denetimi yaptıktan sonra aldığınız hata mesajlarını görünümlerinize gönderecek kolay bir yola ihtiyacınız olacak. Bu iş Laravel tarafından pratik bir şekilde halledilmektedir. Bir örnek olarak şu rotaları ele alalım:

	Route::get('kayit', function()
	{
		return View::make('uye.kayit');
	});

	Route::post('kayit', function()
	{
		$kurallar = array(...);

		$gecerlilikYoklayici = Validator::make(Input::all(), $kurallar);

		if ($gecerlilikYoklayici->fails())
		{
			return Redirect::to('kayit')->withErrors($gecerlilikYoklayici);
		}
	});

Dikkat ederseniz, geçerlilik başarısız olduğunda, `Validator` olgusunu `withErrors` metodunu kullanarak Redirect'e geçiriyoruz. Bu metod, hata mesajlarını oturuma flaş tipinde aktaracak, yani bir sonraki isteğe kadar kullanılabilir olacaktır.

Buna karşın, yine dikkat ederseniz GET rotamızda hata mesajlarını görünüme açık olarak bağlamak zorunda değiliz. Bunun nedeni, Laravel'in oturum verisinde hatalar olup olmadığını her zaman yoklaması ve olduğunu tespit etmesi halinde bunları otomatik olarak görünüme bağlamasıdır. **Bu itibarla, her istekte tüm görünümleriniz için bir `$errors` değişkeni mevcut olacağını unutmayın**, dolayısıyla siz `$errors` değişkeninin her zaman tanımlanmış olduğunu iç rahatı ile varsayıp, güvenle kullanabilirsiniz. Bu `$errors` değişkeni `MessageBag` sınıfından bir olgu olacaktır.

Bu durumda, yeniden yön verme sonrasında, otomatikman bağlanan `$errors` değişkenini görünümlerinizde kullanabilirsiniz:

	<?php echo $errors->first('email'); ?>

### İsimli Hata Çantaları

Eğer tek bir sayfa üzerinde birden çok formunuz varsa, hata `MessageBag`'lerini isimlendirmek isteyebilirsiniz. Bu size belirli bir form için olan hata mesajlarını elde etme imkanı verecektir. Yapacağınız tek şey `withErrors` metoduna ikinci bir parametre olarak bir isim geçmektir:

	return Redirect::to('register')->withErrors($validator, 'login');

Ondan sonra `$errors` değişkeninden isimli `MessageBag` olgusuna erişebilirsiniz:

	<?php echo $errors->login->first('email'); ?>

<a name="available-validation-rules"></a>
## Mevcut Geçerlilik Kuralları

Mevcut tüm geçerlilik kuralları ve bunların işlevleri aşağıda verilmiştir:

- [Accepted](#rule-accepted)
- [Active URL](#rule-active-url)
- [After (Date)](#rule-after)
- [Alpha](#rule-alpha)
- [Alpha Dash](#rule-alpha-dash)
- [Alpha Numeric](#rule-alpha-num)
- [Array](#rule-array)
- [Before (Date)](#rule-before)
- [Between](#rule-between)
- [Boolean](#rule-boolean)
- [Confirmed](#rule-confirmed)
- [Date](#rule-date)
- [Date Format](#rule-date-format)
- [Different](#rule-different)
- [Digits](#rule-digits)
- [Digits Between](#rule-digits-between)
- [E-Mail](#rule-email)
- [Exists (Database)](#rule-exists)
- [Image (File)](#rule-image)
- [In](#rule-in)
- [Integer](#rule-integer)
- [IP Address](#rule-ip)
- [Max](#rule-max)
- [MIME Types](#rule-mimes)
- [Min](#rule-min)
- [Not In](#rule-not-in)
- [Numeric](#rule-numeric)
- [Regular Expression](#rule-regex)
- [Required](#rule-required)
- [Required If](#rule-required-if)
- [Required With](#rule-required-with)
- [Required With All](#rule-required-with-all)
- [Required Without](#rule-required-without)
- [Required Without All](#rule-required-without-all)
- [Same](#rule-same)
- [Size](#rule-size)
- [Timezone](#rule-timezone)
- [Unique (Database)](#rule-unique)
- [URL](#rule-url)

<a name="rule-accepted"></a>
#### accepted

Geçerlilik bakılan alan _yes_, _on_ veya _1_ olmalıdır. Bu, "Hizmet Şartlarının" kabul edildiğinin doğrulanmasında işe yarar.

<a name="rule-active-url"></a>
#### active_url

Geçerlilik bakılan alan `checkdnsrr` PHP fonksiyonuna göre geçerli bir URL olmalıdır.

<a name="rule-after"></a>
#### after: _tarih_

Geçerlilik bakılan alan verilen bir tarihten sonraki bir değer olmalıdır. Tarihler PHP `strtotime` fonksiyonuna geçirilecektir.

<a name="rule-alpha"></a>
#### alpha

Geçerlilik bakılan alan tamamen alfabe harfleri olmalıdır.

<a name="rule-alpha-dash"></a>
#### alpha_dash

Geçerlilik bakılan alan alfa-numerik karakterler yanında tire ve alt tire de olabilir.

<a name="rule-alpha-num"></a>
#### alpha_num

Geçerlilik bakılan alan tamamen alfa-numerik karakterler olmalıdır.

<a name="rule-array"></a>
#### array

Geçerlili bakılan alan array tipinde olmalıdır.

<a name="rule-before"></a>
#### before: _tarih_

Geçerlilik bakılan alan verilen bir tarihten önceki bir değer olmalıdır. Tarihler PHP `strtotime` fonksiyonuna geçirilecektir.

<a name="rule-between"></a>
#### between: _min_, _max_

Geçerlilik bakılan alan verilen _min_ ile _max_ arasında bir büyüklükte olmalıdır. Stringler, sayılar ve dosyalar `size` kuralıyla aynı tarzda değerlendirilir.

<a name="rule-boolean"></a>
#### boolean

Geçerlilik bakılan alan bir boolean olarak çevrilebilmelidir. Kabul edilen girdiler `true`, `false`, `1`, `0`, `"1"` ve `"0"` dır.

<a name="rule-confirmed"></a>
#### confirmed

Geçerlilik bakılan alana uyan eden bir `falan_confirmation` alanı olmalıdır. Örneğin, geçerlilik bakılan alan `parola` ise, inputta karşılık gelen bir `parola_confirmation` alanı olmalıdır.

<a name="rule-date"></a>
#### date

Geçerlilik bakılan alan `strtotime` PHP fonksiyonua göre uygun bir tarih olmalıdır.

<a name="rule-date-format"></a>
#### date_format: _format_

Geçerlilik bakılan alan `date_parse_from_format` PHP fonksiyona göre tanımlanmış bir _format_'a uygun olmalıdır.

<a name="rule-different"></a>
#### different: _alan_

Verilen _alan_, geçerlilik bakılan alandan farklı olmalıdır.

<a name="rule-digits"></a>
#### digits: _value_

Geçerlilik bakılan alan _numerik_ olmalıdır ve tam _value_ uzunluğunda olmalıdır.

<a name="rule-digits-between"></a>
#### digits_between:_min_,_max_

Geçerlilik bakılan alan verilen _min_ ile _max_ arasında bir uzunlukta olmalıdır.

<a name="rule-email"></a>
#### email

Geçerlilik bakılan alan bir e-mail adresi şeklinde biçimlendirilmiş olmalıdır.

<a name="rule-exists"></a>
#### exists: _tablo_, _sütun_

Geçerlilik bakılan alan verilen bir veritabanı tablosunda mevcut olmalıdır.

#### Exists Kuralının Basit Kullanım Şekli

	'il' => 'exists:iller'

#### Özel Bir Sütun İsminin Belirtilmesi

	'il' => 'exists:iller,kisa_hali'

Sorguya "where" cümleciği olarak eklenecek daha fazla şart da belirtebilirsiniz:

	'email' => 'exists:personel,email,hesap_id,1'

Bir "where" cümle parçası olarak `NULL` geçilmesi bir `NULL` veritabanı değeri yönünden kontrol ekleyecektir:

	'email' => 'exists:staff,email,deleted_at,NULL'

<a name="rule-image"></a>
#### image

Geçerlilik bakılan alan bir görsel (jpeg, png, bmp veya gif) olmalıdır.

<a name="rule-in"></a>
#### in: _falan_, _filan_,...

Geçerlilik bakılan alan verilen bir değerler listesinde olmalıdır.

<a name="rule-integer"></a>
#### integer

Geçerlilik bakılan alan bir tamsayı olmalıdır.

<a name="rule-ip"></a>
#### ip

Geçerlilik bakılan alan bir IP adresi olarak biçimlendirilmiş olmalıdır.

<a name="rule-max"></a>
#### max: _deger_

Geçerlilik bakılan alan bir maksimum _deger_'e eşit veya ondan az olmalıdır. Stringler, sayılar ve dosyalar `size` kuralıyla aynı tarzda değerlendirilir.

<a name="rule-mimes"></a>
#### mimes: _falan_, _filan_,...

Geçerlilik bakılan alan listelenen uzantılardan birine tekabül eden bir MIME tipinde olmalıdır.

#### MIME Kuralının Basit Kullanım Şekli

	'resim' => 'mimes:jpeg,bmp,png'

<a name="rule-min"></a>
#### min: _deger_

Geçerlilik bakılan alan bir asgari _deger_'den büyük olmalıdır. Stringler, sayılar ve dosyalar `size` kuralıyla aynı tarzda değerlendirilir.

<a name="rule-not-in"></a>
#### not_in: _falan_,_filan_,...

Geçerlilik bakılan alan verilen değerler listesinde yer almamalıdır.

<a name="rule-numeric"></a>
#### numeric

Geçerlilik bakılan alan sayısal bir değer olmalıdır.

<a name="rule-regex"></a>
#### regex: _desen_

Geçerlilik bakılan alan verilen düzenli ifadeye uygun olmalıdır.

**Not:** `regex` deseni kullanırken, özellikle düzenli ifade bir pipe karakteri (|) içeriyorsa, kuralları belirtmek için pipe ayıracı kullanmak yerine bir dizide belirtmek gerekli olabilir.

<a name="rule-required"></a>
#### required

Geçerlilik bakılan alan input verisinde bulunmak zorundadır.

<a name="rule-required-if"></a>
#### required_if: _alan_, _deger_

Şayet _alan_ alanı _deger_'e eşit ise, geçerlilik bakılan alan girilmek zorundadır.

<a name="rule-required-with"></a>
#### required_with: _falan_, _filan_,...

Geçerlilik bakılan alan, sadece belirtilen alanların bulunması durumunda bulunmak zorundadır.

<a name="rule-required-with-all"></a>
#### required_with_all:_foo_,_bar_,...

Geçerlilik bakılan alan, sadece belirtilen diğer alanların tümünün mevcut olması durumunda bulunmalıdır.

<a name="rule-required-without"></a>
#### required_without: _falan_, _filan_,...

Geçerlilik bakılan alan, sadece diğer belirtilen alanlar olmadığı takdirde bulunmak zorundadır.

<a name="rule-required-without-all"></a>
#### required_without_all:_foo_,_bar_,...

Geçerlilik bakılan alan, sadece belirtilen diğer alanların hiçbirinin mevcut olmaması durumunda bulunmalıdır.

<a name="rule-same"></a>
#### same: _alan_

Verilen _alan_ geçerlilik bakılan alanla aynı olmalıdır.

<a name="rule-size"></a>
#### size: _deger_

Geçerlilik bakılan alan verilen _deger_'le aynı büyüklükte olmalıdır. String veriler için, _deger_ harf sayısı anlamına gelir. Numerik veriler için, _deger_ verilen bir tamsayı değeridir. Dosyalar için, _size_ kilobayt cinsinden dosya boyutuna karşılık gelir.

<a name="rule-timezone"></a>
#### timezone

Geçerlilik bakılan alan `timezone_identifiers_list` PHP fonksiyonuna göre geçerli bir timezone tanımlayıcısı olmalıdır.

<a name="rule-unique"></a>
#### unique: _tablo_, _sütun_, _haric_, _idSütunu_

Geçerlilik bakılan alan verilen bir veritabanı tablosunda benzersiz olmalıdır. Eğer `sütun` seçeneği belirtilmemişse, geçerlilik bakılan alan aynı zamanda sütun adı olarak kabul edilecektir.

#### Unique Kuralının Basit Kullanım Şekli

	'email' => 'unique:uyeler'

#### Özel Bir Sütun Adının Belirtilmesi

	'email' => 'unique:uyeler,email_adresi'

#### Verilen Bir ID İçin Unique Kuralının Göz Ardı Edilmesi

	'email' => 'unique:uyeler,email_adresi,10'

#### Ek olarak Where Cümlecikleri Ekleme

Bir sorguya  "where" cümlecikleri ekler gibi daha fazla şart ekleyebilirsiniz:

	'email' => 'unique:users,email_address,NULL,id,account_id,1'

Yukarıdaki kuralda, sadece `account_id`'si `1` olan satırlar unique yoklamasına dahil edilecektir.

<a name="rule-url"></a>
#### url

Geçerlilik bakılan alan bir URL şeklinde biçimlendirilmiş olmalıdır.

> **Not:** Bu fonksiyon PHP'nin `filter_var` metodunu kullanır.

<a name="conditionally-adding-rules"></a>
## Duruma Göre Kurallar Ekleme

Bazı durumlarda, bir alanla ilgili geçerlilik denetimini **yalnızca** bu alan input dizisinde mevcut olduğu zaman çalıştırmak isteyebilirsiniz. Bunu hızla gerçekleştirmek için kural listenize `sometimes` ekleyiniz:

	$v = Validator::make($data, array(
		'email' => 'sometimes|required|email',
	));

Yukarıdaki örnekte `email` alanı sadece bu alan `$data` dizisinde mevcutsa geçerlilik denetiminden geçirilecektir.

#### Karmaşık Şartlı Geçerlilik Denetimi

Bazen belli bir alanın başka bir alan 100'den büyük bir değere sahip olduğunda gerekli olmasını isteyebilirsiniz. Bu geçerlilik kurallarının eklenmesi sorun oluşturmak zorunda değildir. Öncelikle, asla değişmeyecek _statik kuralları_ nızın olduğu bir `Validator` olgusu oluşturun:

	$v = Validator::make($data, array(
		'email' => 'required|email',
		'games' => 'required|numeric',
	));

Diyelim ki, game toplayıcıları için bir web uygulamamız var. Eğer bir game toplayıcısı uygulamamıza üye olursa ve onun 100'den fazla oyunu varsa, neden bu kadar çok oyunu olduğunu açıklamasını isteyelim. Örneğin, belki oyunları yeniden satan bir dükkanı vardır veya sadece toplamaktan hoşlanıyor olabilir. Bu gereksinimi, duruma göre eklemek için `Validator` olgusunda `sometimes` metodunu kullanabiliriz.

	$v->sometimes('reason', 'required|max:500', function($input)
	{
		return $input->games >= 100;
	});

Bu `sometimes` metoduna geçirilen ilk parametre duruma bağlı olarak geçerlilik denetiminden geçireceğimiz alanın adıdır. İkinci parametre ise, eklemek istedğimiz kurallardır. Üçüncü parametre olarak geçirilen `Closure` `true` döndürürse, ilgili kural eklenecektir. Bu metod, karmaşık şartlı geçerlilikler inşa edilmesini çok kolaylaştırır. Bir defada birden çok alan için şartlı geçerlilik eklemeniz de mümkündür:

	$v->sometimes(array('reason', 'cost'), 'required', function($input)
	{
		return $input->games >= 100;
	});

> **Not:** `Closure`'a geçirilen `$input` parametresi `Illuminate\Support\Fluent`'ın bir olgusu olacaktır ve input ve dosyalarınıza erişeceğiniz bir nesne olarak kullanılabilir.

<a name="custom-error-messages"></a>
## Özel Hata Mesajları

Gerek duyduğunuzda, geçerlilik için ön tanımlı hata mesajları yerine özel hata mesajları kullanabilirsiniz. Özel mesaj belirtmek için birkaç yol var.

#### Validator'e Özel Mesaj Geçilmesi

	$mesajlar = array(
		'required' => ':attribute alanı gereklidir.',
	);

	$gecerlilikYoklayici = Validator::make($input, $kurallar, $mesajlar);

> *Not:* Buradaki `:attribute` yer tutucusu geçerlilik bakılan alanın gerçek adıyla değiştirilecektir. Geçerlilik mesajlarınızda diğer yer tutucuları da kullanabilirsiniz.

#### Diğer Geçerlilik Yer Tutucuları

	$mesajlar = array(
		'same'    => ':attribute ve :other aynı olmalıdır.',
		'size'    => ':attribute tam olarak :size olmalıdır.',
		'between' => ':attribute :min ile :max arasında olmalıdır.',
		'in'      => ':attribute şu tiplerden birisi olmalıdır: :values',
	);

#### Belli Bir Attribute İçin Özel Mesaj Belirlenmesi

Bazen sadece belirli bir alan için özel hata mesajları belirlemek isteyebilirsiniz:

	$mesajlar = array(
		'email.required' => 'e-mail adresinizi bilmemiz gerekiyor!',
	);

<a name="localization"></a>
#### Özel Mesajların Dil Dosyalarında Belirtilmesi

Bazı durumlarda, özel hata mesajlarınızı doğrudan `Validator`'e geçirmek yerine bir dil dosyasında belirtmek isteyebilirsiniz. Bunu yapmak için, mesajlarınızı `app/lang/xx/validation.php` dil dosyasındaki `custom` dizisine ekleyiniz.

	'custom' => array(
		'email' => array(
			'required' => 'e-mail adresinizi bilmemiz gerekiyor!',
		),
	),

<a name="custom-validation-rules"></a>
## Özel Geçerlilik Kuralları

#### Özel Bir Geçerlilik Kuralını Kayda Geçirme

Laravel'de her biri yararlı çok sayıda geçerlilik kuralı bulunmaktadır; bununla birlikte siz kendiniz de bazı kurallar belirlemek isteyebilirsiniz. Özel geçerlilik kuralı kayda geçirmenin bir yolu `Validator::extend` metodunu kullanmaktır:

	Validator::extend('falan', function($attribute, $value, $parameters)
	{
		return $value === 'falan';
	});

Özel bir geçerlilik anonim fonksiyonu (Closure) üç parametre alır: geçerlilik bakılacak `$attribute`'ın adı, bu niteliğin `$value`'i ve kurala geçilecek bir `$parameters` dizisi.

Bu `extend` metoduna bir isimsiz fonksiyon yerine bir sınıf ve metod da geçebilirsiniz:

	Validator::extend('falan', 'FalanValidator@validate');

Özel kurallarınız için aynı zamanda bir hata mesajı da tanımlamanız gerekeceğini unutmayın. Bunu, ya aynı satırda özel hata mesaj dizisi kullanarak ya da geçerlilik dil dosyasına bir giriş eklemek suretiyle yapabilirsiniz.

#### Validator Sınıfının Genişletilmesi

Validator'ü genişletmek için bir isimsiz fonksiyon çağrısı kullanmak yerine, Validator sınıfının kendisini de genişletebilirsiniz. Bunu yapmak için, `Illuminate\Validation\Validator`'ü genişleten bir Validator sınıfı yazın. Validation metodlarınızı, başına `validate` getirerek bu sınıfa ekleyebilirsiniz:

	<?php

	class OzelValidator extends Illuminate\Validation\Validator {

		public function validateFalan($attribute, $value, $parameters)
		{
			return $value === 'falan';
		}

	}

#### Özel Bir Validator Çözümleyicisinin Kayda Geçirilmesi

Daha sonra, özel Validator uzantınızı kayda geçirmeniz gerekiyor:

	Validator::resolver(function($translator, $data, $rules, $messages)
	{
		return new OzelValidator($translator, $data, $rules, $messages);
	});

Özel bir geçerlilik kuralı oluştururken, bazı durumlarda, hata mesajlarıyla değiştirilecek özel yer tutucular tanımlamanız gerekebilir. Bunu, aynen yukarıda tarif edildiği gibi özel bir Validator oluşturup, bu validatore bir `replaceXXX` fonksiyonu ekleyerek gerçekleştirebilirsiniz.

	protected function replaceFalan($message, $attribute, $rule, $parameters)
	{
		return str_replace(':falan', $parameters[0], $message);
	}

Şayet `Validator` sınıfını genişletmeksizin özel bir mesaj "değiştirici" eklemek isterseniz, `Validator::replacer` metodunu kullanabilirsiniz:

	Validator::replacer('rule', function($message, $attribute, $rule, $parameters)
	{
		//
	});
