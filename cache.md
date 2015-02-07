# Cache

- [Yapılandırma](#configuration)
- [Önbellekleme Kullanımı](#cache-usage)
- [Arttırma & Azaltma](#increments-and-decrements)
- [Önbellek Etiketleri (Tags)](#cache-tags)
- [Veritabanı Önbelleği](#database-cache)

<a name="configuration"></a>
## Yapılandırma

Laravel, çeşitli önbellekleme sistemleri için tümleşik bir API sağlar. Önbellekleme yapılandırma ayarları `config/cache.php` dosyasında bulunmaktadır. Bu dosyada uygulamanızda varsayılan olarak hangi önbellekleme sürücüsünü kullanmak istediğinizi belirtebilirsiniz. Laravel, [Memcached](http://memcached.org) ve [Redis](http://redis.io) gibi popüler önbellekleme sürücülerini barındırır.

Önbellekleme yapılandırma dosyası ayrıca dosyanın içinde açıklanmış çeşitli seçenekleri de içerir, bu yüzden o seçenekleri de okuduğunuzdan emin olun. Varsayılan olarak, Laravel, serileştirilerek önbelleklenmiş öğeleri dosya sisteminde depolayan `file` (dosya) önbellekleme sürücüsünü kullanmak üzere ayarlanmıştır. Daha büyük uygulamalar için, Memcached ve APC gibi bir önbellekleme uygulaması kullanmanız önerilir.

Laravel ile Redis Önbellekleme kullanmadan önce, Composer ile `predis/predis` (~1.0) pakedini yüklemeniz gerekir.

<a name="cache-usage"></a>
## Önbellekleme Kullanımı

#### Bir Öğeyi Önbelleğe Koymak

	Cache::put('key', 'value', $minutes);

#### Son Kullanım Zamanını Ayarlamak İçin Carbon Nesneleri Kullanılması

	$expiresAt = Carbon::now()->addMinutes(10);

	Cache::put('key', 'value', $expiresAt);

#### Eğer Öğe Önbellekte Yoksa, Öğeyi Önbelleğe Koymak

	Cache::add('key', 'value', $minutes);

Eğer ilgili öğe önbelleğe gerçekten eklenirse `add` metodu `true` döndürecektir. Aksi takdirde bu metod `false` döndürecektir.

#### Öğenin Önbellekte Var Olup Olmadığını Kontrol Etmek

	if (Cache::has('key'))
	{
		//
	}

#### Önbellekten Bir Öğeyi Almak

	$value = Cache::get('key');

#### Bir Önbellek Değeri Almak Veya Varsayılan Bir Değer Döndürmek

	$value = Cache::get('key', 'default');

	$value = Cache::get('key', function() { return 'default'; });

#### Bir Öğeyi Önbelleğe Kalıcı Olarak Koymak

	Cache::forever('key', 'value');

Bazen, önbellekten bir öğeyi almak isteyebilir ve ayrıca talep edilen öğe yoksa önbellekte varsayılan bir değer saklayabilirsiniz. Bunu, `Cache::remember` metodunu kullanarak yapabilirsiniz:

	$value = Cache::remember('users', $minutes, function()
	{
		return DB::table('users')->get();
	});

Ayrıca, `remember` ve `forever` methodlarını birlikte kullanabilirsiniz.

	$value = Cache::rememberForever('users', function()
	{
		return DB::table('users')->get();
	});

Önbellekte bütün öğelerin serileştirilmiş şekilde saklandığını unutmayın, yani her türlü veriyi saklayabilirsiniz.

#### Önbellekten Bir Öğe Çekilmesi

Eğer önbellekten bir öğeyi elde etmek ve sonra da onu silmeniz gerekirse, `pull` metodunu kullanabilirsiniz:

	$value = Cache::pull('key');

#### Önbellekten Bir Öğeyi Silmek

	Cache::forget('key');

<a name="increments-and-decrements"></a>
## Arttırma & Azaltma

`file` ve `database` hariç tüm sürücüler `increment` (artırma) ve `decrement` (azaltma) işlemlerini destekler:

#### Bir Değeri Arttırmak

	Cache::increment('key');

	Cache::increment('key', $amount);

#### Bir Değeri Azaltmak

	Cache::decrement('key');

	Cache::decrement('key', $amount);

<a name="cache-tags"></a>
## Önbellek Etiketleri (Tags)

> --Not:** Önbellek bölümleri `file` ve `database` önbellekleme sürücüleri kullanılırken desteklenmemektedir. Ayrıca, "forever" saklanır önbelleklerde birden çok tag kullanmanız halinde, bayat kayıtları otomatik olarak temizleyen `memcached` gibi bir sürücüyle performans en iyi olacaktır.

#### Etiketlenmiş Bir Önbelleğe Erişim

Cache tagları cache'deki birbirine yakın öğeleri etiketlemenize ve daha sonra verilen bir isimle etiketlenmiş tüm cache'leri yok etmenize (flush) imkan verir. Etiketlenmiş bir cache'e erişmek için, `tags` metodunu kullanın:

Parametreler olarak sıralı bir tag isimleri listesi geçerek veya parametre olarak sıralı tag isimlerinden oluşan bir dizi geçerek etiketlenmiş bir cache saklayabilirsiniz:

	Cache::tags('people', 'authors')->put('John', $john, $minutes);

	Cache::tags(array('people', 'artists'))->put('Anne', $anne, $minutes);

Tag kombinasyonlarında `remember`, `forever` ve `rememberForever` gibi herhangi bir cache saklama metodunu kullanabilirsiniz. Ayrıca etiketlenmiş cache'lerde önbelleklenmiş öğelere erişebileceğiniz gibi, `increment` ve `decrement` gibi diğer önbellek metodlarını da kullanabilirsiniz.

#### Etiketlenmiş Bir Önbellekteki Öğelere Erişmek

Etiketlenmiş bir cache'ye erişmek için, onu saklarken kullandığınız aynı sıradaki tag listesini geçiniz.

	$anne = Cache::tags('people', 'artists')->get('Anne');

	$john = Cache::tags(array('people', 'authors'))->get('John');

Bir isim veya bir isim listesi ile etiketlenmiş tüm ögeleri yok edebilirsiniz. Örneğin, aşağıdaki cümle `insanlar`, `yazarlar` veya her ikisi ile de etiketlenmiş tüm cache'leri çıkarıp atacaktır. Dolayısıyla, cache'den hem "Mine" hem de "Can" çıkartılacaktır:

	Cache::tags('people', 'authors')->flush();

Tersine, şu cümle sadece `yazarlar` etiketi verilmiş cache'leri çıkartacaktır, yani "Can" çıkartılacak ama "Mine" çıkartılmayacak.

	Cache::tags('authors')->flush();

<a name="database-cache"></a>
## Veritabanı Önbelleği

`database` önbellek sürücüsü kullandığınız takdirde, önbellek öğelerini içeren bir tablo kurulumu gerekir. Bu tablo için örnek bir `şema` aşağıda gösterilmiştir:

	Schema::create('cache', function($table)
	{
		$table->string('key')->unique();
		$table->text('value');
		$table->integer('expiration');
	});
