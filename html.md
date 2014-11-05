# Formlar & HTML

- [Form Açmak](#opening-a-form)
- [CSRF Koruması](#csrf-protection)
- [Forma Model Bağlanması](#form-model-binding)
- [Label Elementi](#labels)
- [Text, Textarea, Password & Hidden Alanlar](#text)
- [Onay Kutuları ve Seçenek Düğmeleri](#checkboxes-and-radio-buttons)
- [File Inputu](#file-input)
- [Aşağı Açılır Listeler](#drop-down-lists)
- [Düğmeler](#buttons)
- [Özel Makrolar](#custom-macros)
- [URL Oluşturma](#generating-urls)

<a name="opening-a-form"></a>
## Form Açmak

#### Form Açmak

	{{ Form::open(array('url' => 'falan/filan')) }}
		//
	{{ Form::close() }}

Varsayılan olarak, `POST` metodu kullanılır; ancak, istediğiniz bir metodu da belirtebilirsiniz:

	echo Form::open(array('url' => 'falan/filan', 'method' => 'put'))

> **Not:** HTML formları, sadece `POST` ve `GET` metotlarını desteklediği için, `PUT` ve `DELETE` metotları formunuza otomatik olarak bir `_method` gizli alanı eklenmek suretiyle taklit edilecektir.

İsimli rotalara veya controller eylemlerine işaret eden formlar da açabilirsiniz:

	echo Form::open(array('route' => 'route.name'))

	echo Form::open(array('action' => 'Controller@method'))

Ayrıca, rota parametreleri de geçebilirsiniz:

	echo Form::open(array('route' => array('route.name', $user->id)))

	echo Form::open(array('action' => array('Controller@method', $user->id)))

Formunuz dosya yüklemelerini kabul edecekse, diziye `files` seçeneğini ekleyin:

	echo Form::open(array('url' => 'falan/filan', 'files' => true))

<a name="csrf-protection"></a>
## CSRF Koruması

#### Bir Forma CSRF Değeri Eklemek

Laravel, uygulamanızı siteler arası istek sahtekarlıklarından korumak için kolay bir metot sunar. Öncelikle, kullanıcının oturumuna rastgele bir değer yerleştirilir. Eğer `Form::open` metodunu `POST`, `PUT` veya `DELETE` ile kullanırsanız, CSRF değeri, formlarınıza gizli bir alan olarak otomatik olarak yerleştirilir. Alternatif olarak, gizli CSRF alanı için HTML kodu üretmek isterseniz, `token` metodunu kullanabilirsiniz:

	echo Form::token();

#### Bir Rotaya CSRF Filtresi Eklemek

	Route::post('profil', array('before' => 'csrf', function()
	{
		//
	}));

<a name="form-model-binding"></a>
## Forma Model Bağlanması

#### Bir Model Formu Açmak

Sıklıkla, bir modelin içeriğine dayanan bir form oluşturmak isteyebilirsiniz. Bunu yapmak için, `Form::model` metodunu kullanın:

	echo Form::model($user, array('route' => array('user.update', $user->id)))

Şimdi, bir form elementi oluşturduğunuzda, mesela bir text input, elementin ismiyle eşleşen modelin değeri, otomatik olarak alanın değeri olarak belirlenir. Yani, örneğin, `email` ismine sahip bir text alanı için, kullanıcı modelinin `email` niteliği değer olarak atanır. Bununla birlikte, dahası da var! Oturum flaş verisinde inputa uyan bir öğe mevcutsa, bu değer, model'in değerine nazaran öncelik alacaktır. Yani, öncelik şu şekildedir:

1. Oturum Flaş Verisi (Önceki Girdi)
2. Doğrudan Atanmış Değer
3. Model Nitelik Değeri

Bu size model değerlerine bağlanan formları sadece çabukça oluşturmanıza imkan vermekle kalmaz, sunucu tarafında bir geçerlilik hatası olduğunda tekrar kolayca doldurmanızı da sağlayacaktır!

> **Not:** `Form::model` kullanıyor olduğunuzda, `Form::close` ile formunuzu kapatmayı unutmayın!

<a name="labels"></a>
## Label Elementi

#### Bir Label Elementi Üretilmesi

	echo Form::label('email', 'E-Mail Adresi');

#### Ek HTML Nitelikleri Belirtme

	echo Form::label('email', 'E-Mail Adresi', array('class' => 'awesome'));

> **Not:** Bir label oluştururken, label ismiyle aynı isimde oluşturduğunuz bir form elementi otomatik olarak label ile aynı isimde bir ID de alacaktır.

<a name="text"></a>
## Text, Textarea, Password & Hidden Alanlar

#### Bir Text Inputu Üretilmesi

	echo Form::text('uyeadi');

#### Ön Tanımlı Bir Değer Belirtilmesi

	echo Form::text('email', 'ornek@gmail.com');

> **Not:** *hidden* ve *textarea* metodları *text* metodu ile aynı şekilde yazılır.

#### Bir Password Inputu Üretilmesi

	echo Form::password('parola');

#### Diğer Inputlar Üretilmesi

	echo Form::email($name, $value = null, $attributes = array());
	echo Form::file($name, $attributes = array());

<a name="checkboxes-and-radio-buttons"></a>
## Onay Kutuları ve Seçenek Düğmeleri

#### Bir Checkbox Veya Radio Inputu Üretilmesi

	echo Form::checkbox('isim', 'deger');

	echo Form::radio('isim', 'deger');

#### Seçilmiş Bir Checkbox Veya Radio Inputu Üretilmesi

	echo Form::checkbox('isim', 'deger', true);

	echo Form::radio('isim', 'deger', true);

<a name="file-input"></a>
## File Inputu

#### Bir File Inputu Üretilmesi

	echo Form::file('resim');

> **Not:** Bu form `files` opsiyonu `true` ayarlanmış olarak açılmış olmalıdır.

<a name="drop-down-lists"></a>
## Aşağı Açılır Listeler

#### Aşağı Açılır Bir Liste Üretilmesi

	echo Form::select('boyut', array('B' => 'Büyük', 'K' => 'Küçük'));

#### Ön Tanımlı Seçilmiş Bir Aşağı Açılır Liste Üretilmesi

	echo Form::select('size', array('B' => 'Büyük', 'K' => 'Küçük''), 'K');

#### Gruplanmış Bir Liste Üretilmesi

	echo Form::select('hayvan', array(
		'Kediler' => array('tekir' => 'Tekir'),
		'Köpekler' => array('kangal' => 'Kangal'),
	));

#### Bir Aralık Olan Bir Aşağı Açılır Liste Üretilmesi

    echo Form::selectRange('number', 10, 20);

#### Ay İsimleri Olan Bir Liste Üretilmesi

    echo Form::selectMonth('month');

<a name="buttons"></a>
## Düğmeler

#### Bir Submit Düğmesinin Üretilmesi

	echo Form::submit('Tıkla beni!');

> **Not:** Bir button elemanı üretmeniz gerekiyorsa, *button* metodunu kullanın. Bu aynı *submit* gibi yazılır.

<a name="custom-macros"></a>
## Özel Makrolar

#### Bir Form Makrosunun Kayda Geçirilmesi

"Makrolar" denen kendi özel Form sınıf yardımcılarınızı tanımlamak kolaydır. Nasıl çalıştığını görün: Önce belli bir isim ve Closure fonksiyonu ile makroyu kayda geçirin:

	Form::macro('makroAlan', function()
	{
		return '<input type="awesome">';
	});

Şimdi adını kullanarak makronuzu çağırabilirsiniz:

#### Özel Bir Form Makrosunun Çağırılması

	echo Form::makroAlan();

<a name="generating-urls"></a>
## URL Oluşturma

URL oluşturma ile ilgili detaylı bilgi için dokümantasyonun <a href="/docs/helpers#urls">Yardımcı (Helper) Fonksiyonları</a> bölümüne bakınız.
