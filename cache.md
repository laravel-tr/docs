# Önbellekleme (Cache)

- [Yapılandırma](#configuration)
- [Önbellekleme Kullanımı](#cache-usage)
- [Arttırma & Azaltma](#increments-and-decrements)
- [Önbellek Etiketleri (Tags)](#cache-tags)
- [Veritabanı Önbelleği](#database-cache)

<a name="configuration"></a>
## Yapılandırma

Laravel, çeşitli önbellekleme sistemleri için tümleşik bir API sağlar. Önbellekleme yapılandırma ayarları `app/config/cache.php` dosyasında bulunmaktadır. Bu dosyada uygulamanızda varsayılan olarak hangi önbellekleme sürücüsünü kullanmak istediğinizi belirtebilirsiniz. Laravel, [Memcached](http://memcached.org) ve [Redis](http://redis.io) gibi popüler önbellekleme sürücülerini barındırır.

Önbellekleme yapılandırma dosyası ayrıca dosyanın içinde açıklanmış çeşitli seçenekleri de içerir, bu yüzden o seçenekleri de okuduğunuzdan emin olun. Varsayılan olarak, Laravel, serileştirilerek önbelleklenmiş öğeleri dosya sisteminde depolayan `file` (dosya) önbellekleme sürücüsünü kullanmak üzere ayarlanmıştır. Daha büyük uygulamalar için, Memcached ve APC gibi bir önbellekleme uygulaması kullanmanız önerilir.

<a name="cache-usage"></a>
## Önbellekleme Kullanımı

#### Bir Öğeyi Önbelleğe Koymak

	Cache::put('anahtar', 'değer', $dakika);

#### Son Kullanım Zamanını Ayarlamak İçin Carbon Nesneleri Kullanılması

	$sonZaman = Carbon::now()->addMinutes(10);

	Cache::put('anahtar', 'değer', $sonZaman);

#### Eğer Öğe Önbellekte Yoksa, Öğeyi Önbelleğe Koymak

	Cache::add('anahtar', 'değer', $dakika);

Eğer ilgili öğe önbelleğe gerçekten **eklenirse** `add` metodu `true` döndürecektir. Aksi takdirde bu metod `false` döndürecektir.

#### Öğenin Önbellekte Var Olup Olmadığını Kontrol Etmek

	if (Cache::has('anahtar'))
	{
		//
	}

#### Önbellekten Bir Öğeyi Almak

	$value = Cache::get('anahtar');

#### Bir Önbellek Değeri Almak Veya Varsayılan Bir Değer Döndürmek

	$value = Cache::get('anahtar', 'varsayılanDeğer');

	$value = Cache::get('anahtar', function() { return 'varsayılanDeğer'; });

#### Bir Öğeyi Önbelleğe Kalıcı Olarak Koymak

	Cache::forever('anahtar', 'değer');

Bazen, önbellekten bir öğeyi almak isteyebilir ve ayrıca talep edilen öğe yoksa önbellekte varsayılan bir değer saklayabilirsiniz. Bunu, `Cache::remember` metodunu kullanarak yapabilirsiniz:

	$value = Cache::remember('kullanicilar', $dakika, function()
	{
		return DB::table('kullanicilar')->get();
	});

Ayrıca, `remember` ve `forever` methodlarını birlikte kullanabilirsiniz.

	$value = Cache::rememberForever('kullanicilar', function()
	{
		return DB::table('kullanicilar')->get();
	});

Önbellekte bütün öğelerin serileştirilmiş şekilde saklandığını unutmayın, yani her türlü veriyi saklayabilirsiniz.

#### Önbellekten Bir Öğeyi Silmek

	Cache::forget('anahtar');

<a name="increments-and-decrements"></a>
## Arttırma & Azaltma

`file` ve `database` hariç tüm sürücüler `increment` (artırma) ve `decrement` (azaltma) işlemlerini destekler:

#### Bir Değeri Arttırmak

	Cache::increment('anahtar');

	Cache::increment('anahtar', $miktar);

#### Bir Değeri Azaltmak

	Cache::decrement('anahtar');

	Cache::decrement('anahtar', $miktar);

<a name="cache-tags"></a>
## Önbellek Etiketleri (Tags)

> **Not:** Önbellek bölümleri `file` ve `database` önbellekleme sürücüleri kullanılırken desteklenmemektedir. Ayrıca, "forever" saklanır önbelleklerde birden çok tag kullanmanız halinde, bayat kayıtları otomatik olarak temizleyen `memcached` gibi bir sürücüyle performans en iyi olacaktır.

#### Etiketlenmiş Bir Önbelleğe Erişim

Cache tagları cache'deki birbirine yakın öğeleri etiketlemenize ve daha sonra verilen bir isimle etiketlenmiş tüm cache'leri yok etmenize (flush) imkan verir. Etiketlenmiş bir cache'e erişmek için, `tags` metodunu kullanın:

Parametreler olarak sıralı bir tag isimleri listesi geçerek veya parametre olarak sıralı tag isimlerinden oluşan bir dizi geçerek etiketlenmiş bir cache saklayabilirsiniz.

	Cache::tags('insanlar', 'yazarlar')->put('Can', $can, $dakika);

	Cache::tags(array('insanlar', 'artistler'))->put('Mine', $mine, $dakika);

Tag kombinasyonlarında `remember`, `forever` ve `rememberForever` gibi herhangi bir cache saklama metodunu kullanabilirsiniz. Ayrıca etiketlenmiş cache'lerde önbelleklenmiş öğelere erişebileceğiniz gibi, `increment` ve `decrement` gibi diğer önbellek metodlarını da kullanabilirsiniz.

#### Etiketlenmiş Bir Önbellekteki Öğelere Erişmek

Etiketlenmiş bir cache'ye erişmek için, onu saklarken kullandığınız aynı sıradaki tag listesini geçiniz.

	$mine = Cache::tags('insanlar', 'artistler')->get('Mine');

	$can = Cache::tags(array('insanlar', 'yazarlar'))->get('Can');

Bir isim veya bir isim listesi ile etiketlenmiş tüm ögeleri yok edebilirsiniz. Örneğin, aşağıdaki cümle `insanlar`, `yazarlar` veya her ikisi ile de etiketlenmiş tüm cache'leri çıkarıp atacaktır. Dolayısıyla, cache'den hem "Mine" hem de "Can" çıkartılacaktır:

	Cache::tags('insanlar', 'yazarlar')->flush();

Tersine, şu cümle sadece `yazarlar` etiketi verilmiş cache'leri çıkartacaktır, yani "Can" çıkartılacak ama "Mine" çıkartılmayacak.

	Cache::tags('yazarlar')->flush();

<a name="database-cache"></a>
## Veritabanı Önbelleği

`database` önbellek sürücüsü kullandığınız takdirde, önbellek öğelerini içeren bir tablo kurulumu gerekir. Bu tablo için örnek bir `şema` aşağıda gösterilmiştir:

	Schema::create('cache', function($table)
	{
		$table->string('key')->unique();
		$table->text('value');
		$table->integer('expiration');
	});
