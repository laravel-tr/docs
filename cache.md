# Önbellekleme (Cache)

- [Yapılandırma](#configuration)
- [Önbellekleme Kullanımı](#cache-usage)
- [Arttırma & Azaltma](#increments-and-decrements)
- [Önbellek Bölümleri](#cache-sections)
- [Veritabanı Önbelleği](#database-cache)

<a name="configuration"></a>
## Yapılandırma

Laravel, çeşitli önbellekleme sistemleri için tümleşik bir API sağlar. Önbellekleme yapılandırma ayarları `app/config/cache.php` dosyasında bulunmaktadır. Bu dosyada uygulamanızda varsayılan olarak hangi önbellekleme sürücüsünü kullanmak istediğinizi belirtebilirsiniz. Laravel, [Memcached](http://memcached.org) ve [Redis](http://redis.io) gibi popüler önbellekleme sürücülerini barındırır.

Önbellekleme yapılandırma dosyası ayrıca dosyanın içinde açıklanmış çeşitli seçenekleri de içerir, bu yüzden o seçenekleri de okuduğunuzdan emin olun. Varsayılan olarak, Laravel, sıralanarak önbelleklenmiş öğeleri dosya sisteminde depolayan `file` (dosya) önbellekleme sürücüsünü kullanmak üzere ayarlanmıştır. Daha büyük uygulamalar için, Memcached ve APC gibi bir önbellekleme uygulaması kullanmanız önerilir.

<a name="cache-usage"></a>
## Önbellekleme Kullanımı

#### Bir Öğeyi Önbelleğe Koymak

	Cache::put('key', 'value', $minutes);

#### Using Carbon Objects To Set Expire Time

	$expiresAt = Carbon::now()->addMinutes(10);

	Cache::put('key', 'value', $expiresAt);

#### Eğer Öğe Önbellekte Yoksa, Öğeyi Önbelleğe Koymak

	Cache::add('key', 'value', $minutes);

The `add` method will return `true` if the item is actually **added** to the cache. Otherwise, the method will return `false`.

#### Öğenin Önbellekte Var Olup Olmadığını Kontrol Etmek

	if (Cache::has('key'))
	{
		//
	}

#### Önbellekten Bir Öğeyi Almak

	$value = Cache::get('key');

#### Bir Önbellek Değeri Almak Veya Varsayılan Bir Değer Döndürmek

	$value = Cache::get('key', 'varsayılanDeğer');

	$value = Cache::get('key', function() { return 'varsayılanDeğer'; });

#### Bir Öğeyi Kalıcı Olarak Önbelleğe Koymak

	Cache::forever('key', 'value');

Bazen, önbellekten bir öğeyi almak isteyebilir ve ayrıca talep edilen öğe yoksa önbellekte varsayılan bir değer saklayabilirsiniz. Bunu, `Cache::remember` metodunu kullanarak yapabilirsiniz:

	$value = Cache::remember('kullanicilar', $minutes, function()
	{
		return DB::table('kullanicilar')->get();
	});

Ayrıca, `remember` ve `forever` methodlarını birlikte kullanabilirsiniz.

	$value = Cache::rememberForever('kullanicilar', function()
	{
		return DB::table('kullanicilar')->get();
	});

Önbellekte bütün öğelerin sıralanmış şekilde saklandığını unutmayın, yani her türlü veriyi saklayabilirsiniz.

#### Önbellekten Bir Öğeyi Silmek

	Cache::forget('key');

<a name="increments-and-decrements"></a>
## Arttırma & Azaltma

`file` ve `database` hariç tüm sürücüler `increment` (artma) ve `decrement` (azalma) işlemlerini destekler:

#### Bir Değeri Arttırmak

	Cache::increment('key');

	Cache::increment('key', $miktar);

#### Bir Değeri Azaltmak

	Cache::decrement('key');

	Cache::decrement('key', $miktar);

<a name="cache-sections"></a>
## Önbellek Bölümleri

> **Not:** Önbellek bölümleri `dosya` ve `veritabanı` önbellekleme sürücüleri kullanılırken desteklenmemektedir.

Önbellek bölümleri, önbellekteki ilişkili öğeleri gruplamanıza ve tüm bölümü temizlemenize olanak sağlar. Bölüme erişim için `section` metodu kullanılır:

#### Bir Önbellek Bölümününe Erişim

You may store a tagged cache by passing in an ordered list of tag names as arguments, or as an ordered array of tag names.

	Cache::section('insanlar')->put('Mehmet', $mehmet, $dakika);

	Cache::section('insanlar')->put('Şeyma', $ayse, $dakika);

You may use any cache storage method in combination with tags, including `remember`, `forever`, and `rememberForever`. Ayrıca bölümlerde önbelleklenmiş öğelere, diğer önbellek metodlarında olduğu gibi `increment` ve `decrement` ile de erişebilirsiniz.

#### Önbellek Bölümündeki Öğelere Erişmek

To access a tagged cache, pass the same ordered list of tags used to save it.

	$anne = Cache::section('insanlar')->get('Tuana Şeyma');

	$mehmet = Cache::tags(array('insanlar', 'yazarlar'))->get('Mehmet');

You may flush all items tagged with a name or list of names. For example, this statement would remove all caches tagged with either `insanlar`, `yazarlar`, or both. So, both "Tuana Şeyma" and "Mehmet" would be removed from the cache:

	Cache::tags('insanlar', 'yazarlar')->flush();

In contrast, this statement would remove only caches tagged with `yazarlar`, so "Mehmet" would be removed, but not "Tuana Şeyma".

	Cache::tags('yazarlar')->flush();

<a name="database-cache"></a>
## Veritabanı Önbelleği

`Veritabanı` önbellek sürücüsü kullanırken, önbellek öğelerini içeren bir tablo kurulumu gerekir. Bu tablo için örnek bir `şema` aşağıda gösterilmiştir:

	Schema::create('cache', function($table)
	{
		$table->string('key')->unique();
		$table->text('value');
		$table->integer('expiration');
	});