# Eloquent ORM

- [Giriş](#introduction)
- [Temel Kullanım](#basic-usage)
- [Toplu Atama](#mass-assignment)
- [Ekleme, Güncelleme, Silme](#insert-update-delete)
- [Belirsiz Silme](#soft-deleting)
- [Zaman Damgaları](#timestamps)
- [Sorgu Kapsamları](#query-scopes)
- [İlişkiler](#relationships)
- [İlişkilerin Sorgulanması](#querying-relations)
- [Ateşli (Eager) Yüklemeler](#eager-loading)
- [İlişkili Modelleri Ekleme](#inserting-related-models)
- [Ebeveyn Zaman Damgalarına Dokunma](#touching-parent-timestamps)
- [Pivot Tablolarla Çalışmak](#working-with-pivot-tables)
- [Koleksiyonlar](#collections)
- [Erişimciler & Değiştiriciler (Accessors & Mutators)](#accessors-and-mutators)
- [Tarih Değiştiricileri](#date-mutators)
- [Model Olayları](#model-events)
- [Model Gözlemcileri](#model-observers)
- [Diziye / JSON'a Çevirme](#converting-to-arrays-or-json)

<a name="introduction"></a>
## Giriş

Laravel ile gelen Eloquent ORM, veritabanınızla çalışırken kullanacağınız güzel ve sade bir ActiveRecord uygulaması sağlamaktadır. Her veritabanı tablosu, bu tabloyla etkileşim için kullanılacak kendine has bir "Model" sahibidir.

Başlamadan önce, `app/config/database.php`'de bir veritabanı bağlantısı yapılandırmış olduğunuzdan emin olun.

<a name="basic-usage"></a>
## Temel Kullanım

Öncelikle bir Eloquent modeli oluşturunuz. Modeller tipik olarak `app/models` klasöründe yer alır, fakat siz modellerinizi `composer.json` dosyanıza göre otomatik yükleme yapabileceğiniz başka bir yere de koyabilirsiniz.

#### Bir Eloquent Modelinin Tanımlanması

	class Uye extends Eloquent {}

Dikkat ederseniz Eloquent'e `Uye` modelimiz için hangi tabloyu kullanacağımızı söylemedik. Eğer açıkça başka bir isim belirtilmezse tablo isimi olarak sınıf adının ingilizde çoğulunun küçük harf hali kullanılacaktır. Dolayısıyla bizim örneğimizde Eloquent, `Uye` modelinin  `uyes` tablosundaki kayıtları tutacağını varsayacaktır. Tablo ismini açıkça belirtmek için modelinizde bir `table` özelliği tanımlayınız:

	class Uye extends Eloquent {

		protected $table = 'uyeler';

	}

> **Not:** Eloquent'in başka bir ön kabulü de her tablonun `id` adında bir primer key sütunu olduğudur. Bu kuralı aşmak için de bir `primaryKey` özelliği tanımlamanız gerekecek. Benzer şekilde, modeliniz kullanılacağı zaman kullanılacak veritabanı bağlantısının adını değiştirmek için bir `connection` özelliği tanımlayabilirsiniz.

Bir model tanımladıktan sonra artık tablonuzda kayıt oluşturmaya ve ondan kayıt getirmeye başlayabilirsiniz. Tablolarınıza ön tanımlı olarak `updated_at` ve `created_at` sütunları koymanız gerektiğine dikkat ediniz. Şayet bu sütunların otomatik olarak tutulmasını istemiyorsanız, modelinizdeki `$timestamps` özelliğini `false` olarak ayarlayınız.

#### Tüm Modellerin Alınması

	$uyeler = Uye::all();

#### Birincil Anahtara Göre Bir Kaydın Alınması

	$uye = Uye::find(1);

	var_dump($uye->isim);

> **Not:** [Sorgu Oluşturucusu](/docs/queries)'nda bulunan tüm metodlar Eloquent modellerini sorgularken de kullanılabilir.

#### Birincil Anahtara Göre Bir Model Alınması ya da Ortaya Bir İstisna Çıkartılması

Bazı durumlarda bir model bulunamadığında bir istisna çıkartmak, böylece bir `App::error` işleyicisi kullanarak istisnayı yakalayabilmek ve bir 404 sayfası göstermek isteyebilirsiniz.

	$model = Uye::findOrFail(1);

	$model = Uye::where('oylar', '>', 100)->firstOrFail();

Bu hata işleyicinin kaydını yapmak için `ModelNotFoundException`'i dinlemek gerekir.

	use Illuminate\Database\Eloquent\ModelNotFoundException;

	App::error(function(ModelNotFoundException $e)
	{
		return Response::make('Bulunamadı', 404);
	});

#### Eloquent Modelleri Kullanarak Sorgu Yapma

	$uyeler = Uye::where('puan', '>', 100)->take(10)->get();

	foreach ($uyeler as $uye)
	{
		var_dump($uye->isim);
	}

#### Eloquent Küme Metodları

Tabi ki, sorgu oluşturucusunun kümeleme fonksiyonlarını da kullanabilirsiniz.

	$adet = Uye::where('puan', '>', 100)->count();

Gereken sorguyu fluent arayüzüyle üretemediğiniz zaman `whereRaw` kullanabilirsiniz:

	$uyeler = Uye::whereRaw('yas > ? and puan = 100', array(25))->get();

#### Sonuçları Öbeklere Bölme

Eğer bir sürü (binlerce) Eloquent kaydını işlemeniz gerekiyorsa, `chunk` komutunun kullanılması tüm bunları RAM'inizi yemeden yapmanıza imkan verecektir:

	Uye::chunk(200, function($uyeler)
	{
		foreach ($uyeler as $uye)
		{
			//
		}
	});

Metoda geçilen birinci parametre "öbek" başına almak istediğiniz kayıt sayısıdır. Veritabanından çekilen her bir öbek için ikinci parametre olarak geçilen Closure çağrılacaktır.

#### Query Bağlantısının Belirtilmesi

Bir Eloquent sorgusu çalıştırırken hangi bağlantının kullanılacağını da belirleyebilirsiniz. `On` metodunu kullanmanız yeterlidir:

	$uyeler = Uye::on('bağlantı-adı')->find(1);

<a name="mass-assignment"></a>
## Toplu Atama

Yeni bir model oluşturulurken model oluşturucuya niteliklerden oluşan bir dizi geçersiniz. Bu nitelikler bu durumda modele "toplu atama" aracılığıyla atanır. Bu gayet uygun bir yaklaşımdır, fakat bir kullanıcı girdisi bir modele körleme geçirildiği takdirde **ciddi** bir güvenlik sorunu olabilecektir. Kullanıcı girdisi bir modele körlemesine geçirilirse, bu kullanıcı modelin niteliklerinin **birisini (any)** ve **hepsini (all)** değiştirebilecektir. Bu sebepler yüzünden, tüm Eloquent modelleri ön tanımlı olarak toplu atamaya karşı koyar.

Başlamak için modelinizde `fillable` veya `guarded` özelliğini ayarlayınız.

#### Bir Modelde Fillable Niteliklerin Tanımlanması

Bunlardan `fillable` özelliği hangi niteliklerin toplu atanacaklarını belirler. Bu işlem sınıf ya da olgu düzeyinde ayarlanabilir.

	class Uye extends Eloquent {

		protected $fillable = array('ismi', 'soy_ismi', 'email');

	}

Bu örnekte, sadece belirttiğimiz üç nitelik toplu atanabilecektir.

#### Bir Modelde Guarded Niteliklerin Tanımlanması

`fillable`'in tersi `guarded`'dır ve bir "beyaz-liste" yerine bir "kara-liste" olarak iş görür:

	class Uye extends Eloquent {

		protected $guarded = array('id', 'parola');

	}

#### Toplu Atamanın Tüm Nitelikler İçin Engellenmesi

Yukardaki örneğe göre `id` ve `parola` nitelikleri toplu atana **mayacaktır**. Diğer tüm nitelikler toplu atanabilecektir. Toplu atamayı niteliklerin **hepsi** için bloke etmeyi de seçebilirsiniz:

	protected $guarded = array('*');

<a name="insert-update-delete"></a>
## Ekleme, Güncelleme, Silme

Veritabanında bir modelden yeni bir kayıt oluşturmak için, yeni bir model olgusu oluşturun ve `save` metodunu çağırın.

#### Yeni Bir Modelin Kaydedilmesi

	$uye = new Uye;

	$uye->isim = 'Can';

	$uye->save();

> **Not:** Tipik olarak, Eloquent modellerinizde otomatik artan anahtarlar olacaktır. Ama siz kendi keylerinizi belirlemek isterseniz, modelinizdeki `incrementing` özelliğini `false` olarak ayarlayın.

Yeni bir modeli tek satırda kaydetmek için `create` metodunu kullanabilirsiniz. Eklenen model olgusu bu metoddan döndürülecektir. Ancak, tüm Elequent modelleri toplu atamaya karşı korunumlu oldukları için, bunu yapmadan önce modelinizde bir `fillable` veya `guarded` özelliği belirlemeniz gerekecektir.

Otomatik artan IDler kullanan yeni bir modelin kaydedilmesi (olgu oluşturup, değer atanıp, "save" kullanılması) veya oluşturulması (üç işlemin hepsini birden yapan "create" metodunun kullanılması) sonrasında, nesnenin `id` niteliğine erişerek bu ID'i öğrenebilirsiniz:

	$insertedId = $user->id;

#### Modeldeki Korunumlu Niteliklerin Ayarlanması

	class Uye extends Eloquent {

		protected $guarded = array('id', 'hesap_no');

	}

#### Model Create Metodunun Kullanımı

	// Veritabanında yeni bir üye oluştur...
	$uye = Uye::create(array('isim' => 'Can'));

	// Bazı alanlarına göre üyeyi getir ya da öyle bir üye yoksa oluştur...
	$user = User::firstOrCreate(array('name' => 'John'));

	// Bazı alanlarına göre üyeyi getir ya da yeni bir üye olgusu başlat...
	$user = User::firstOrNew(array('name' => 'John'));

#### Getirilen Bir Modelin Güncellenmesi

Bir modeli güncellemek için onu getirir, bir niteliğini değiştirir, sonra da `save` metodunu kullanabilirsiniz:

	$uye = Uye::find(1);

	$uye->email = 'can@filan.com';

	$uye->save();

#### Bir Model ve İlişkilerinin Kaydedilmesi

Bazen sadece bir modeli değil, onun bütün ilişkilerini de kaydetmek isteyebilirsiniz. Bunu yapmak için `push` metodunu kullanın:

	$uye->push();

Ayrıca, bir modeller kümesinde güncelleme sorguları da çalıştırabilirsiniz:

	$satirSayisi = Uye::where('puan', '>', 100)->update(array('durum' => 2));

> **Not:** Eloquent sorgu oluşturucu aracılığıyla bir modeller kümesi güncellendiği zaman herhangi bir model olayı ateşlenmez.

#### Mevcut Bir Modelin Silinmesi

Bir modeli silmek için olgu üzerinde `delete` metodunu çağırın:

	$uye = Uye::find(1);

	$uye->delete();

#### Mevcut Bir Modelin Key Aracılığıyla Silinmesi

	Uye::destroy(1);

	Uye::destroy(array(1, 2, 3));

	Uye::destroy(1, 2, 3);

Elbette, bir modeller kümesinde bir silme sorgusu da çalıştırabilirsiniz:

	$satirSayisi = Uye::where('puan', '>', 100)->delete();

#### Bir Modelin Sadece Zaman Damgalarının Güncellenmesi

Eğer bir modelde sadece zaman damgalarını güncellemek istiyorsanız, `touch` metodunu kullanabilirsiniz:

	$uye->touch();

<a name="soft-deleting"></a>
## Belirsiz Silme

Bir model belirsiz silindiğinde, aslında veritabanınızdan çıkartılmaz. Onun yerinde kayıttaki bir `deleted_at` zaman damgası ayarlanır. Bir model için belirsiz silmeler yapılabilmesi için modelinize 'SoftDeletingTrait' uygulamanız gerekir:

	use Illuminate\Database\Eloquent\SoftDeletingTrait;

	class Uye extends Eloquent {

		use SoftDeletingTrait;

		protected $dates = ['deleted_at'];

	}

Tablonuza bir `deleted_at` sütunu eklemek için ise, bir migrasyondan `softDeletes` metodunu kullanabilirsiniz:

	$table->softDeletes();

Şimdi, artık modelinizde `delete` metodunu çağırdığınız zaman, bu `deleted_at` sütunu güncel zaman damgasına ayarlanacaktır. Belirsiz silme kullanılan bir model sorgulandığında, "silinmiş olan" modeller sorgu sonuçlarına dahil edilmeyecektir. 

#### Belirsiz Silinmiş Modelleri Sonuçlara Girmeye Zorlama

Bir sonuç kümesinde belirsiz silinmiş modellerin gözükmesini zorlamak için sorgunuzda `withTrashed` metodunu kullanınız:

	$uyeler = Uye::withTrashed()->where('hesap_no', 1)->get();

Bu `withTrashed` metodu tanımlanmış bir ilişki üzerinde kullanılabilir:

	$user->posts()->withTrashed()->get();

Sonuç kümenizde **sadece** belirsiz silinmiş modellerin olmasını istiyorsanız, `onlyTrashed` metodunu kullanabilirsiniz:

	$uyeler = Uye::onlyTrashed()->where('hesap_no', 1)->get();

Belirsiz silinmiş bir modeli tekrar etkin hale getirmek için, `restore` metodunu kullanın:

	$uye->restore();

`restore` metodunu bir sorguda da kullanabilirsiniz:

	Uye::withTrashed()->where('hesap_no', 1)->restore();

`restore` metodu ilişkilerde de kullanılabilir:

	$uye->postalar()->restore();

Bir modeli veritabanından gerçekten çıkartmak istediğinizde, `forceDelete` metodunu kullanabilirsiniz:

	$uye->forceDelete();

`forceDelete` metodu ilişkilerde de çalışır:

	$uye->postalar()->forceDelete();

Belli bir model olgusunun belirsiz silme özelliğine sahip olup olmadığını öğrenmek için, `trashed` metodunu kullanabilirsiniz:

	if ($uye->trashed())
	{
		//
	}

<a name="timestamps"></a>
## Zaman Damgaları

Ön tanımlı olarak, veritabanı tablonuzdaki `created_at` ve `updated_at` sütunlarının idamesini otomatik olarak Eloquent yapacaktır. Size tek düşen `datetime` tipindeki bu iki alanı tablonuza eklemektir, geri kalan işleri Eloquent üstlenecektir. Şayet siz bu sütunların idamesini Eloquent'in yapmasını istemiyorsanız, modelinize şu özelliği eklemeniz gerekir:

#### Otomatik Zaman Damgalarının Devre Dışı Bırakılması

	class Uye extends Eloquent {

		protected $table = 'uyeler';

		public $timestamps = false;

	}

#### Özel Bir Zaman Damgası Biçiminin Şart Koşulması

Zaman damgalarınızın biçimini özelleştirmek isterseniz, modelinizdeki `freshTimestamp` metodunu ezebilirsiniz (override):

	class Uye extends Eloquent {

		public function freshTimestamp()
		{
			return time();
		}

	}

<a name="query-scopes"></a>
## Sorgu Kapsamları

#### Bir Sorgu Kapsamının Tanımlanması

Kapsamlar size sorgu mantığınızı modellerinizde tekrar tekrar kullanma imkanı verir. Bir kapsam tanımlamak için bir model metodunun başına `scope` getirmeniz yeterlidir:

	class Uye extends Eloquent {

		public function scopePopular($query)
		{
			return $query->where('puan', '>', 100);
		}

		public function scopeKadin($query)
		{
			return $query->whereGender('W');
		}

	}

#### Bir Sorgu Kapsamının Kullanılması

	$uyeler = Uye::popular()->kadin()->orderBy('created_at')->get();

#### Dinamik Kapsamlar

Bazen parametreler kabul eden kapsam tanımlamak isteyebilirsiniz. Yapmanız gereken kapsam metoduna parametrelerinizi eklemek:

	class Uye extends Eloquent {

		public function scopeOfType($query, $type)
		{
			return $query->whereType($type);
		}

	}

Parametreyi kapsamın çağrısına geçin:

	$uyeler = Uye::ofType('moderator')->get();

<a name="relationships"></a>
## İlişkiler

Pek tabii, veritabanı tablolarınız büyük ihtimalle bir diğeriyle ilişkilidir. Örneğin bir blog yazısında çok sayıda yorum olabilir veya bir sipariş onu ısmarlayan kullanıcı ile ilişkili olacaktır. Eloquent bu ilişkileri kolayca yönetmenizi ve rahat çalışmanızı sağlar. Laravel birçok ilişki tipini desteklemektedir:

- [Birden Bire](#one-to-one)
- [Birden Birçoğa](#one-to-many)
- [Birçoktan Birçoğa](#many-to-many)
- [Aracılığıyla Birçoğa Sahip (Has Many Through)](#has-many-through)
- [Çokbiçimli İlişkiler](#polymorphic-relations)
- [Birçoktan Birçoğa Çokbiçimli İlişkiler](#many-to-many-polymorphic-relations)

<a name="one-to-one"></a>
### Birden Bire

#### Birden Bire Tarzı İlişki Tanımlama

Birden bire şeklindeki bir ilişki çok basit bir ilişkidir. Örneğin, bir `Uye` modelinin bir `Telefon`'u olabilir. Eloquent'de bu ilişkiyi şöyle tanımlayabiliriz:

	class Uye extends Eloquent {

		public function tel()
		{
			return $this->hasOne('Telefon');
		}

	}

`hasOne` metoduna geçirilen ilk parametre ilişkili modelin adıdır. İlişki tanımlandıktan sonra onu Eloquent'in [dinamik özellikler](#dynamic-properties)'ini kullanarak elde edebiliriz:

	$tel = Uye::find(1)->tel;

Bu cümlenin gerçekleştirdiği SQL şunlardır (tablo isimleri model tanımında özel olarak belirtilmedi ise tablo ismi olarak model isminin küçük harfli çoğul halinin kullanıldığını hatırlayınız):

	select * from uyes where id = 1

	select * from telefons where uye_id = 1

Eloquent'in ilişkideki yabancı key'in ne olduğuna model adına göre karar verdiğine dikkat ediniz. Şimdiki örnekte `Telefon` modelinin `uye_id` adlı bir yabancı key kullandığı varsayılmaktadır. Siz bu ön kuralı değiştirmek istiyorsanız `hasOne` metoduna ikinci bir parametre geçebilirsiniz. Dahası, ilişki için hangi local alanın kullanılması gerektiğini belirtmek için üçüncü bir parametre geçebilirsiniz:

	return $this->hasOne('Telefon', 'foreign_key');

	return $this->hasOne('Telefon', 'foreign_key', 'local_key');

#### Bir İlişkinin Tersinin Tanımlanması

`Telefon` modeli üzerinde ilişkinin tersini tanımlamak için, `belongsTo` metodunu kullanınız:

	class Telefon extends Eloquent {

		public function uye()
		{
			return $this->belongsTo('Uye');
		}

	}

Yukarıdaki örnekte, Eloquent, `telefons` tablosunda bir `user_id` alanı arayacaktır. Eğer farklı bir yabancı anahtar sütunu tanımlamak isterseniz, onu `belongsTo` metoduna ikinci parametre olarak geçebilirsiniz:

	class Telefon extends Eloquent {

		public function uye()
		{
			return $this->belongsTo('Uye', 'local_key');
		}

	}

Ek olarak, ebeveyn tabloda ilişkilendirilen sütunun adını belirten üçüncü bir parametre geçebilirsiniz:

	class Telefon extends Eloquent {

		public function uye()
		{
			return $this->belongsTo('Uye', 'local_key', 'parent_key');
		}

	}

<a name="one-to-many"></a>
### Birden Birçoğa

Birden birçoğa ilişki örneği olarak birçok yorum yapılmış bir blog yazısı verilebilir. Bu ilişkiyi de şöyle modelleyebiliriz:

	class Makale extends Eloquent {

		public function yorumlar()
		{
			return $this->hasMany('Yorum');
		}

	}

Şimdi artık bir makalenin yorumlarına [dinamik özellik](#dynamic-properties) aracılığıyla ulaşabiliriz:

	$yorumlar = Makale::find(1)->yorumlar;

Hangi yorumların alınacağını daha da kısıtlamak için `yorumlar` metodunu çağırabilir ve şartlar koşmayı sürdürebilirsiniz:

	$yorumlar = Makale::find(1)->yorumlar()->where('baslik', '=', 'bu')->first();

Tıpkı hasOne'de olduğu gibi konvansiyonel yabancı key varsayımını `hasMany` metoduna ikinci bir parametre geçerek değiştirebilirsiniz. Ve, aynı şekilde local sütun da belirtilebilir:

	return $this->hasMany('Yorum', 'foreign_key');

	return $this->hasMany('Yorum', 'foreign_key', 'local_key');

#### Bir İlişkinin Tersinin Tanımlanması

İlişkinin tersini `Yorum` modelinde tanımlamak için, `belongsTo` metodu kullanılmaktadır:

	class Yorum extends Eloquent {

		public function makale()
		{
			return $this->belongsTo('Makale');
		}

	}

<a name="many-to-many"></a>
### Birçoktan Birçoğa

Birçoktan birçoğa ilişkiler daha karmaşık bir ilişki tipidir. Bu tarz bir ilişki örneği bir üyenin birçok rolü olması, aynı zamanda bu rollerin başka kullanıcılar tarafından da paylaşılmasıdır. Örneğin birçok üye "Müdür" rolünde olabilir. Bu ilişki için üç veritabanı tablosu gereklidir: `uyeler`, `roller` ve `rol_uye`. Bu `rol_uye` tablosu ilişkili model isimlerinin alfabetik sıralamasına göre adlandırılır ve `uye_id` ve `rol_id` sütunlarına sahip olmalıdır (model isimlerine alttire ve id eklenmiş iki alan).

Birçoktan birçoğa ilişkileri `belongsToMany` metodunu kullanarak tanımlayabiliyoruz:

	class Uye extends Eloquent {

		public function roller()
		{
			return $this->belongsToMany('Rol');
		}

	}

Artık rolleri `Uye` modeli aracılığıyla getirebiliriz:

	$roller = Uye::find(1)->roller;

Pivot tablo ismi olarak ön kabullü tablo ismi yerine başka bir isim kullanmak isterseniz, bunu `belongsToMany` metoduna ikinci bir parametre geçerek gerçekleştirebilirsiniz:

	return $this->belongsToMany('Rol', 'uye_rolleri');

İlişkili keyler için konvansiyonel yaklaşımı da değiştirebilirsiniz:

	return $this->belongsToMany('Rol', 'uye_rolleri', 'uye_id', 'foo_id');

Ve tabii ki ilişkinin tersini `Rol` modelinde de tanımlayabilirsiniz:

	class Rol extends Eloquent {

		public function uyeler()
		{
			return $this->belongsToMany('Uye');
		}

	}

<a name="has-many-through"></a>
### Aracılığıyla Birçoğa Sahip (Has Many Through)

Bu "has many through" ilişkisi, aradaki bir ilişki aracılığıyla uzak ilişkilere erişim için uygun bir kestirme yol sağlar. Örneğin, bir `Memleket` modeli bir `Uye` modeli aracılığıyla bir çok `Post` sahibi olabilir. Bu ilişkinin tabloları şunun gibi gözükecektir:

	memlekets
		id - integer
		isim - string

	uyes
		id - integer
		memleket_id - integer
		isim - string

	posts
		id - integer
		uye_id - integer
		baslik - string

Bu `posts` tablosunda bir `memleket_id` sütunu olmamasına rağmen, `hasManyThrough` ilişkisi bir memleketin post'larına `$memleket->posts` aracılığıyla erişebilmemize imkan verecektir. Önce bu ilişkiyi tanımlayalım:

	class Memleket extends Eloquent {

		public function posts()
		{
			return $this->hasManyThrough('Post', 'Uye');
		}

	}

Eğer ilişkinin keylerini elle belirtmek isterseniz, metoda üçüncü ve dördüncü parametreler geçebilirsiniz:

	class Memleket extends Eloquent {

		public function posts()
		{
			return $this->hasManyThrough('Post', 'Uye', 'memleket_id', 'uye_id');
		}

	}

<a name="polymorphic-relations"></a>
### Çokbiçimli İlişkiler

Çokbiçimli (Polimorfik) İlişkiler bir modelin tek bir ilişkilendirme ile birden çok modele ait olmasına imkan verir. Örneğin, kendisi ya bir personel modeline ya da bir siparis modeline ait olan bir foto modeliniz olduğunu düşünün. Bu ilişkiyi şu şekilde tanımlayacağız:

	class Foto extends Eloquent {

		public function resim()
		{
			return $this->morphTo();
		}

	}

	class Personel extends Eloquent {

		public function fotolar()
		{
			return $this->morphMany('Foto', 'resim');
		}

	}

	class Siparis extends Eloquent {

		public function fotolar()
		{
			return $this->morphMany('Foto', 'resim');
		}

	}

#### Çokbiçimli Bir İlişkinin Getirilmesi

Artık bir personel ya da siparişe ait fotoları elde edebiliriz:

	$personel = Personel::find(1);

	foreach ($personel->fotolar as $foto)
	{
		//
	}

#### Çokbiçimli Bir İlişkinin Sahibinin Getirilmesi

Ancak, "çokbiçimli" ilişkinin gerçek farkını bir personel veya siparişe `Foto` modelinden erişebilmekle görürsünüz:

	$foto = Foto::find(1);

	$resim = $foto->resim;

`Foto` modelindeki `resim` ilişkisi, fotonun sahibi olan modele bağlı olarak ya bir `Personel` ya da bir `Siparis` olgusu döndürecektir.

#### Çokbiçimli İlişki Tablo Yapısı

Bunun nasıl çalıştığını anlamanıza yardımcı olmak için bir polimorfik ilişkinin veritabanı yapısını keşfedelim:

	personel
		id - integer
		isim - string

	siparisler
		id - integer
		fiyat - integer

	fotolar
		id - integer
		dosyayolu - string
		resim_id - integer
		resim_type - string

Buradaki anahtar alanların `fotolar` tablosundaki `resim_id` and `resim_type` olduğuna dikkat ediniz. Buradaki ID, fotonun sahibi olan personel veya siparişin ID'ini, TYPE ise sahip olan modelin sınıf adını tutacaktır. Böylece ORM, `resim` ilişkisiyle erişildiğinde döndürülecek sahip modelin hangisi olduğunu tespit edebilecektir.

<a name="many-to-many-polymorphic-relations"></a>
### Birçoktan Birçoğa Çokbiçimli İlişkiler

#### Çokbiçimli Birçoktan Birçoğa İlişkilerin Tablo Yapısı

Geleneksel çokbiçimli ilişkilere ek olarak, birçoktan birçoğa çokbiçimli ilişkiler de belirleyebilirsiniz. Örneğin, bir blog `Post` ve `Video` modeli bir `Tag` modeline polimorfik bir ilişki paylaşabilirler. Önce, tablo yapısını inceleyelim:

	posts
		id - integer
		name - string

	videos
		id - integer
		name - string

	tags
		id - integer
		name - string

	taggables
		tag_id - integer
		taggable_id - integer
		taggable_type - string

Sonra da, model üzerinde ilişkileri kurmaya geçelim. `Post` ve `Video` modellerinin her ikisi de bir `tags` metodu aracılığıyla bir `morphToMany` ilişkisine sahip olacaktır:

	class Post extends Eloquent {

		public function tags()
		{
			return $this->morphToMany('Tag', 'taggable');
		}

	}

`Tag` modeli ise ilişkilerinin her biri için bir metod tanımlayabilir:

	class Tag extends Eloquent {

		public function posts()
		{
			return $this->morphedByMany('Post', 'taggable');
		}

		public function videos()
		{
			return $this->morphedByMany('Video', 'taggable');
		}

	}

<a name="querying-relations"></a>
## İlişkilerin Sorgulanması

#### Seçerken İlişkilerin Yoklanması

Bir modelin kayıtlarına erişirken, sonuçları bir ilişki varlığına göre sınırlamak isteyebilirsiniz. Diyelim ki, en az bir yorum yapılmış tüm blog makalelerini çekmek istediniz. Bunu yapmak için `has` metodunu kullanabilirsiniz:

	$makaleler = Makale::has('yorumlar')->get();

Ayrıca, bir işlemci ve bir sayı da belirleyebilirsiniz, örneğin üç ve daha çok yorum almış makaleleri getirmek için:

	$makaleler = Makale::has('yorumlar', '>=', 3)->get();

Hatta daha fazla güce ihtiyacınız varsa, `has` sorgularınıza "where" şartları koymak için `whereHas` ve `orWhereHas` metodlarını kullanabilirsiniz:

	$makaleler = Makale::whereHas('yorumlar', function($q)
	{
		$q->where('content', 'like', 'filan%');

	})->get();

<a name="dynamic-properties"></a>
### Dinamik Özellikler

Eloquent, ilişkilerinize dinamik özellikler yoluyla erişme imkanı verir. Eloquent ilişkiyi sizin için otomatik olarak yükleyecektir. Hatta, `get` (birden birçoğa ilişkiler için) metodunun mu yoksa `first` (birden bire ilişkiler için) metodunun mu çağırılacağını bilecek kadar akılllıdır. İlişkiyle aynı isimli dinamik bir özellik aracılığı ile erişilebilir olacaktır. Örneğin, şu `$telefon` modelinde:

	class Telefon extends Eloquent {

		public function uye()
		{
			return $this->belongsTo('Uye');
		}

	}

	$telefon= Telefon::find(1);

Bu üyenin email'ini şu şekilde göstermek yerine:

	echo $telefon->uye()->first()->email;

Buradaki gibi basit bir hale kısaltılabilir:

	echo $telefon->uye->email;

> **Not:** Çoklu sonuç döndüren ilişkiler bir `Illuminate\Database\Eloquent\Collection` sınıf olgusu döndürecektir.

<a name="eager-loading"></a>
## Ateşli (Eager) Yüklemeler

Ateşli yükleme N + 1 sorgu problemini gidermek içindir. Örnek olarak, `Yazar` ile ilişkilendirilmiş bir `Kitap` modelini düşünün. İlişki de şöyle tanımlanmış olsun:

	class Kitap extends Eloquent {

		public function yazar()
		{
			return $this->belongsTo('Yazar');
		}

	}

Şimdi, şu kodu ele alalım:

	foreach (Kitap::all() as $kitap)
	{
		echo $kitap->yazar->isim;
	}

Bu döngü tablodaki kitapların hepsini almak için 1 sorgu çalıştıracak, sonra da yazarını elde etmek için her bir kitabı sorgulayacaktır. Yani, eğer 25 kitabımız varsa bu döngü 26 sorgu çalıştıracaktır.

Neyseki, sorgu sayısını büyük ölçüde azaltan ateşli yükleme kullanabiliriz. Ateşli yüklenecek ilişkiler `with` metodu aracılığıyla belirlenebilmektedir:

	foreach (Kitap::with('yazar')->get() as $kitap)
	{
		echo $kitap->yazar->isim;
	}

Yukardaki döngüde sadece iki sorgu çalıştırılacaktır (model tanımında tablo isimleri açıkça belirtilmediyse ingilizce küçük harf çoğul kabulünü hatırlayınız):

	select * from kitaps

	select * from yazars where id in (1, 2, 3, 4, 5, ...)

Ateşli yüklemenin akıllıca kullanımı uygulamanızın performansını önemli ölçüde artırabilir.

Tabii ki, bir defada birden çok ilişkiyi ateşli yükleyebilirsiniz:

	$kitaplar = Kitap::with('yazar', 'kitabevi')->get();

Hatta içi içe ilişkileri de ateşleyebilirsiniz:

	$kitaplar = Kitap::with('yazar.kisiler')->get();

Yukarıdaki örnekte `yazar` ilişkisi ateşli yüklenecektir ve yazarın `kisiler` ilişkisi de ateşli yüklenecektir.

### Ateşli Yükleme Sınırlamaları

Bazen bir ilişkiyi ateşli yüklemek, ama ateşli yükleme için de bir şart belirlemek isteyebiliriz. İşte bir örnek:

	$uyeler = Uye::with(array('makaleler' => function($query)
	{
		$query->where('baslik', 'like', '%birinci%');

	}))->get();

Bu örnekte üyenin makalelerinden sadece baslik alanında "birinci" kelimesi geçen makalelerini ateşli yüklüyoruz.

Tabii ki, ateşli yükleme Closure'ları "sınırlamalara" sınırlı değildir. Sıralama da yapabilirsiniz:

	$uyeler = Uye::with(array('makaleler' => function($query)
	{
		$query->orderBy('created_at', 'desc');

	}))->get();

### Tembel Ateşli Yükleme

İlişkili modelleri, direkt olarak önceden mevcut model koleksiyonundan ateşli yüklemek de mümkündür. Bu özellikle ilişkili modeli önbellekleme ile birlikte yükleyip yüklememeye dinamik karar vereceğiniz zaman işe yarayabilir.

	$kitaplar= Kitap::all();

	$kitaplar->load('yazar', 'kitabevi');

<a name="inserting-related-models"></a>
## İlişkili Modelleri Ekleme

#### İlişkili Bir Modelin Eklenmesi

Yeni ilişkili model ekleme ihtiyacınız çok olacaktır. Örneğin, bir makale için yeni bir yorum eklemek isteyebilirsiniz. Model üzerinde `makale_id` yabancı key alanını elle ayarlamak yerine, doğrudan ebeveyn `Makale` modelinden yeni yorum ekleyebilirsiniz:

	$yorum = new Yorum(array('mesaj' => 'Yeni bir yorum.'));

	$makale = Makale::find(1);

	$yorum = $makale->yorumlar()->save($yorum);

Bu örnekte eklenen yorumdaki `makale_id` alanı otomatik olarak ayarlanmaktadır.

### Modellerin Üye Yapılması (Belongs To)

Bir `belongsTo` ilişkisi güncellenirken, `associate` metodunu kullanabilirsiniz. Bu metod çocuk modeldeki yabancı anahtarı ayarlayacaktır:

	$hesap = Hesap::find(10);

	$uye->hesap()->associate($hesap);

	$uye->save();

### İlişkili Model Ekleme (Birçoktan Birçoğa)

Birçoktan birçoğa ilişkilerle çalışırken de ilişkili model ekleyebilirsiniz. Daha önceki örneğimiz `Uye` ve `Rol` modellerini kullanmaya devam edelim. Bir üyeye yeni roller eklemeyi `attach` metodu ile yapabiliriz:

#### Birçoktan Birçoğa Modellerinin Eklenmesi

	$uye = Uye::find(1);

	$uye->roller()->attach(1);

İlişkiler için pivot tabloda tutulan nitelikleri bir dizi olarak da geçebilirsiniz:

	$uye->roller()->attach(1, array('sonaerme' => $sonaerme));

Tabii, `attach`'in ters işlemi `detach`'tir:

	$uye->roller()->detach(1);

#### Birçoktan Birçoğa Model Bağlamak İçin Sync Kullanımı

İlişkili modelleri bağlamak için `sync` metodunu da kullanabilirsiniz. Bu `sync` metodu parametre olarak pivot tablodaki yerlerin id'lerinden oluşan bir dizi geçirilmesini ister. Bu işlem tamamlandıktan sonra, model için kullanılacak ara tabloda sadece bu id'ler olacaktır:

	$uye->roller()->sync(array(1, 2, 3));

#### Sync Yaparken Pivot Veri Eklenmesi

Belli id değerleri olan başka pivot tabloyu da ilişkilendirebilirsiniz:

	$uye->roller()->sync(array(1 => array('sonaerme' => true)));

Bazen yeni bir ilişkili model oluşturmak ve tek bir komutla bunu eklemek isteyebilirsiniz. Bu işlem için, `save` metodunu kullanabilirsiniz:

	$rol = new Rol(array('isim' => 'Editor'));

	Uye::find(1)->roller()->save($rol);

Bu örnekte, yeni bir `Rol` modeli kaydedilecek ve uye modeline eklenecektir. Bu işlem için bağlı tablolardaki niteliklerden oluşan bir dizi de geçebilirsiniz:

	Uye::find(1)->roller()->save($rol, array('sonaerme' => $sonaerme));

<a name="touching-parent-timestamps"></a>
## Ebeveyn Zaman Damgalarına Dokunma

Bir `Yorum`'un bir `Makale`'ye ait olması örneğimizdeki gibi, bir model başka bir modele ait (`belongsTo`) olduğu takdirde, çocuk modeli güncellediğiniz zaman ebeveyn zaman damgasını da güncellemek iyidir. Örneğin, bir `Yorum` güncellendiğinde, bunun sahibi olan `Makale`'nin `updated_at` zaman damgasını otomatikman güncellemek isteyebilirsiniz. Bunu gerçekleştirmek için tek yapacağınız şey, çocuk modele ilişkilerin isimlerini içeren bir `touches` özelliği eklemektir:

	class Yorum extends Eloquent {

		protected $touches = array('makale');

		public function makale()
		{
			return $this->belongsTo('Makale');
		}

	}

Bunu yaptıktan sonra artık bir `Yorum` güncellediğinizde, sahibi olan `Makale` de güncellenmiş bir `updated_at` sütununa sahip olacaktır:

	$yorum = Yorum::find(1);

	$yorum->text = 'Bu yorumu düzelt!';

	$yorum->save();

<a name="working-with-pivot-tables"></a>
## Pivot Tablolarla Çalışmak

Daha önce öğrendiğiniz gibi, birçoktan birçoğa ilişkilerle çalışmak bir ara tablonun olmasını gerektirir. Eloquent işte bu tablo ile etkileşim için çok yararlı bazı yollar sağlamaktadır. Örneğin bizim bir `Uye` nesnemiz, bir de onun bağlı olduğu birçok `Rol` nesnelerimiz olsun. Bu ilişkiye eriştikten sonra, `pivot` tabloya modellerimiz üzerinden erişebiliriz:

	$uye = Uye::find(1);

	foreach ($uye->roller as $rol)
	{
		echo $rol->pivot->created_at;
	}

Dikkat ederseniz, elde ettiğimiz her bir `Rol` modeline otomatikman bir `pivot` niteliği atanmıştır. Bu nitelik, ara tabloyu temsil eden bir modeli taşır ve herhangi bir Eloquent modeli gibi kullanılabilir.

Ön tanımlı olarak, `pivot` nesnesinde sadece keyler olacaktır. Şayet pivot tablonuzda bunlardan başka nitelikler varsa, bunları ilişki tanımlama sırasında belirtmelisiniz:

	return $this->belongsToMany('Rol')->withPivot('falan', 'filan');

Şimdi `Rol` modelinin `pivot` nesnesinde `falan` ve `filan` nitelikleri erişilebilir olacaktır.

Eğer pivot tablonuzun `created_at` ve `updated_at` zaman damgalarını otomatik olarak halletmesini istiyorsanız, ilişki tanımlamasında `withTimestamps` metodunu kullanın:

	return $this->belongsToMany('Rol')->withTimestamps();

#### Bir Pivot Tablodaki Kayıtların Silinmesi

Bir modelin pivot tablosundaki tüm kayıtları silmek için, `detach` metodunu kullanabilirsiniz:

	Uye::find(1)->roller()->detach();

Bu operasyonun `roller` tablosundan kayıt silmediğine, sadece pivot tablodan sildiğine dikkat ediniz.

#### Bir Pivot Tablodaki Bir Kaydın Güncellenmesi

Bazen pivot tablonuzu güncellemeniz ama onu detach etmemeniz gerekebilir. Eğer pivot tablonuzu "yerinde" güncellemek istiyorsanız aşağıdakine benzer şekilde `updateExistingPivot` metodunu kullanabilirsiniz:

	Uye::find(1)->roller()->updateExistingPivot($roleId, $attributes);

#### Özel Bir Pivot Model Tanımlanması

Laravel size özel bir Pivot model tanımlama imkanı da verir. Özel bir model tanımlamak için, öncelikle `Eloquent`'i genişleten kendi "Taban" model sınıfınızı oluşturun. Diğer Eloquent modellerinizde, defaut `Eloquent` taban yerine bu özel taban modeli genişletin. Taban modelinize, sizin özel Pivot modelinizin bir olgusunu döndüren aşağıdaki fonksiyonu ekleyin:

	public function newPivot(Model $parent, array $attributes, $table, $exists)
	{
		return new YourCustomPivot($parent, $attributes, $table, $exists);
	}

<a name="collections"></a>
## Koleksiyonlar

Eloquent tarafından `get` metodu veya bir `relationship` (ilişki) aracılığıyla döndürülen tüm çoklu sonuç kümeleri bir Eloquent `Collection` nesnesi döndürecektir. Bu nesne PHP'nin `IteratorAggregate` arayüzünün bir uygulama biçimidir ve tıpkı bir dizide dolaşır gibi dolaşılabilinmektedir. Bunun yanında, bu nesne sonuç kümeleriyle çalışırken işe yarayan başka bir takım metodlara da sahiptir.

#### Bir Koleksiyonun Bir Key Taşıyıp Taşımadığının Yoklanması

Örneğin biz `contains` metodunu kullanarak bir sonuç kümesinin belli bir primer key içerip içermediğini tespit edebiliriz:

	$roller = Uye::find(1)->roller;

	if ($roller->contains(2))
	{
		//
	}

Koleksiyonlar aynı zamanda bir dizi ya da JSON'a dünüştürülebilmektedir:

	$roller = Uye::find(1)->roller->toArray();

	$roller = Uye::find(1)->roller->toJson();

Eğer bir koleksiyon bir string kalıbına çevrilirse JSON olarak döndürülecektir:

	$roller = (string) Uye::find(1)->roller;

#### Koleksiyonlarda Tekrarlı İşlemler

Eloquent koleksiyonları içerdikleri elemanları dolaşmak ve filtre etmekle ilgili bazı metodlara da sahiptir:

	$roller = $uye->roller->each(function($rol)
	{

	});

#### Koleksiyonlarda Filtreleme

Verilen Closure (callback) [array_filter()](http://php.net/manual/en/function.array-filter.php) için fonksiyon olarak kullanılacak.

	$uyeler = $uyeler->filter(function($uye)
	{
		return $uye->isAdmin();
	});

> **Not:** Bir koleksiyonu filtreler ve onu JSON'a döndürürken, dizinin keylerini reset etmek için önce `values` fonksiyonunu çağırmayı deneyin.

#### Her Bir Koleksiyon Nesnesine Bir Anonim Fonksiyon (Callback) Uygulamak

	$roller = Uye::find(1)->roller;

	$roller->each(function($rol)
	{
		//
	});

#### Bir Koleksiyonu Bir Değere Göre Sıralama

	$roller = $roller->sortBy(function($rol)
	{
		return $rol->created_at;
	});

#### Bir Koleksiyonu Bir Değere Göre Sıralama

	$roller = $roller->sortBy('created_at');

#### Özel Bir Koleksiyon Tipinin Döndürülmesi

Bazen de, kendi eklediğiniz metodları olan özel bir koleksiyon nesnesi döndürmek isteyebilirsiniz. Bunu, Eloquent modeliniz üzerinde `newCollection` metodunu ezerek yapabilirsiniz:

	class Uye extends Eloquent {

		public function newCollection(array $models = array())
		{
			return new CustomCollection($models);
		}

	}

<a name="accessors-and-mutators"></a>
## Erişimciler & Değiştiriciler (Accessors & Mutators)

#### Bir Erişimci Tanımlanması

Eloquent model niteliklerini alıp getirirken veya onları ayarlarken dönüşüm yapmak için uygun bir yol sağlar. Bir erişimci beyan etmek için modeliniz üzerinde sadece bir `getFilanAttribute` metodu tanımlamak yeterlidir. Yalnız unutmamanız gereken şey, veritabanı sütunlarınızın isimleri yılan tarzı (küçük harfli kelimelerin boşluk olmaksızın alt tire ile birbirine bağlanması) olsa dahi, metodlarınızın deve tarzı (birinci kelimenin tümü küçük harf olmak ve sonraki kelimelerin ilk harfi büyük diğer hafleri küçük olmak üzere boşluk olmaksızın kelimelerin yanyana dizilmesi) olması gerektiğidir:

	class Uye extends Eloquent {

		public function getSoyAdiAttribute($value)
		{
			return ucfirst($value);
		}

	}

Yukarıdaki örnekte `soy_adi` sütununun bir erişimcisi vardır. Niteliğin değerinin erişimciye geçildiğine dikkat ediniz.

#### Bir Değiştirici Tanımlanması

Değiştiriciler de benzer şekilde deklare edilir:

	class Uye extends Eloquent {

		public function setSoyAdiAttribute($value)
		{
			$this->attributes['soy_adi'] = strtolower($value);
		}

	}

<a name="date-mutators"></a>
## Tarih Değiştiricileri

Ön tanımlı olarak, Eloquent `created_at`, `updated_at` ve `deleted_at` sütunlarını [Carbon](https://github.com/briannesbitt/Carbon), olgularına çevirecektir. Carbon çeşitli yardımcı metodlar sağlar ve PHP'nin `DateTime` sınıfını genişletir.

Siz hangi alanların otomatik olarak değiştirileceğini isteğinize göre ayarlayabilirsiniz, hatta modeldeki `getDates` metodunu ezmek suretiyle bu davranışı tamamen devre dışı bırakabilirsiniz:

	public function getDates()
	{
		return array('created_at');
	}

Bir sütun bir tarih olarak kabul edildiğinde, bunun değerini bir UNIX timetamp, date string (`Y-m-d`), date-time string ve tabii ki bir `DateTime` / `Carbon` olgusuna ayarlayabilirsiniz.

Tarih değiştiricilerini tümden devre dışı bırakmak için `getDates` metodunda boş bir dizi döndürünüz:

	public function getDates()
	{
		return array();
	}

<a name="model-events"></a>
## Model Olayları

Eloquent modelleri bazı olayları tetikleyerek, modelin yaşam döngüsündeki çeşitli noktalarda müdahale etmenize imkan verir. Bu amaçla şu metodlar kullanılmaktadır: `creating`, `created`, `updating`, `updated`, `saving`, `saved`, `deleting`, `deleted`, `restoring`, `restored`.

Yeni bir öğe ilk defa kaydedilir kaydedilmez `creating` ve `created` olayları ateşlenecektir. Eğer bir öğe yeni değilse ve `save` metodu çağrılırsa, `updating` / `updated` olayları ateşlenecektir. Her iki durumda da `saving` / `saved` olayları ateşlenecektir.

#### Saklama Operasyonlarının Olaylar Aracığıyla İptal Edilmesi

Eğer `creating`, `updating`, `saving` veya `deleting` olaylarından `false` döndürülürse, eylem iptal edilecektir:

	Uye::creating(function($uye)
	{
		if ( ! $uye->isValid()) return false;
	});

#### Bir Model Boot Metodunun Ayarlanması

Eloquent modelleri bunun dışında static bir `boot` metodu içermekte olup, olay bağlamanızı kayıt etmeniz için uygun bir yerdir.

	class Uye extends Eloquent {

		public static function boot()
		{
			parent::boot();

			// Olay bağlamayı ayarla...
		}

	}

<a name="model-observers"></a>
## Model Gözlemcileri

Model olaylarının işlenmesini pekiştirmek için, bir model gözlemcisi kaydı yapabilirsiniz. Bir gözlemci sınıfında çeşitli model olaylarına tekabül eden metodlar bulunabilir. Örneğin bir gözlemcide, diğer model olay isimlerine ek olarak `creating`, `updating`, `saving` metodları olabilir.

Yani, bir model gözlemcisi şöyle olabilir:

	class UyeGozlemcisi {

		public function saving($model)
		{
			//
		}

		public function saved($model)
		{
			//
		}

	}

Modelinizde `observe` metodunu kullanarak bir gözlemci olgusu kaydı yapabilirsiniz:

	Uye::observe(new UyeGozlemcisi);

<a name="converting-to-arrays-or-json"></a>
## Diziye / JSON'a Çevirme

#### Bir Modelin Bir Diziye Çevrilmesi

JSON APIler oluşturulurken, çoğu defa modellerinizi ve ilişkilerini dizilere veya JSON'a çevirmeniz gerekecektir. Bu yüzden Eloquent bunları yapacak metodlar içermektedir. Bir modeli ve onun yüklenen ilişkilerini bir diziye çevirmek için `toArray` metodunu kullanabilirsiniz:

	$uye = Uye::with('roller')->first();

	return $uye->toArray();

Modellerin koleksiyonlarının da bütün olarak dizilere dönüştürülebildiğini unutmayın:

	return Uye::all()->toArray();

#### Bir Modelin JSON'a Çevrilmesi

Bir Modeli JSON'a çevirmek için, `toJson` metodunu kullanabilirsiniz:

	return Uye::find(1)->toJson();

#### Bir Modelin Bir Rotadan Döndürülmesi

Bir model veya koleksiyon bir string kalıbına sokulduğu takdirde, JSON'a çevrileceğine dikkat ediniz. Yani Elequent nesnelerini direkt olarak uygulamanızın rotalarından döndürebilirsiniz!

	Route::get('uyeler', function()
	{
		return Uye:all();
	});

#### Niteliklerin Dizi veya JSON'a Çevrilmekten Saklanması

Bazen bazı nitelikleri (örneğin şifreleri) modelinizin dizi veya JSON biçimlerinden hariç tutmak isteyebilirsiniz. Bunu yapmak için modelinize bir `hidden` özelliği ekleyiniz:

	class Uye extends Eloquent {

		protected $hidden = array('parola');

	}

> **Not:** İlişkileri gizlerken dinamik erişimci ismini değil ilişkinin **metod** ismini kullanın.

Alternatif olarak, beyaz bir liste tanımlamak için `visible` özelliğini kullanabilirsiniz:

	protected $visible = array('adi', 'soy_adi');

<a name="array-appends"></a>
Kimi durumlarda, veritabanınızdaki bir sütuna tekabül etmeyen dizi nitelikleri eklemeniz gerekebilir. Bunu yapmak için, değer için bir erişimci tanımlamanız yeterlidir:

	public function getIsAdminAttribute()
	{
		return $this->attributes['admin'] == 'yes';
	}

Erişimciyi oluşturduktan sonra, ilgili modeldeki `appends` özelliğine değeri ekleyin:

	protected $appends = array('is_admin');

Nitelik `appends` listesine eklendikten sonra modelin hem dizi hem de JSON formlarına dahil edilecektir.
