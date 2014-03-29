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

<a name="cache-tags"></a>
## Cache Tags

> **Note:** Cache tags are not supported when using the `file` or `database` cache drivers. Furthermore, when using multiple tags with caches that are stored "forever", performance will be best with a driver such as `memcached`, which automatically purges stale records.

Cache tags allow you to tag related items in the cache, and then flush all caches tagged with a given name. To access a tagged cache, use the `tags` method:

#### Accessing A Tagged Cache

You may store a tagged cache by passing in an ordered list of tag names as arguments, or as an ordered array of tag names.

	Cache::tags('people', 'authors')->put('John', $john, $minutes);

	Cache::tags(array('people', 'artists'))->put('Anne', $anne, $minutes);

You may use any cache storage method in combination with tags, including `remember`, `forever`, and `rememberForever`. You may also access cached items from the tagged cache, as well as use the other cache methods such as `increment` and `decrement`:

#### Accessing Items In A Tagged Cache

To access a tagged cache, pass the same ordered list of tags used to save it.

	$anne = Cache::tags('people', 'artists')->get('Anne');

	$john = Cache::tags(array('people', 'authors'))->get('John');

You may flush all items tagged with a name or list of names. For example, this statement would remove all caches tagged with either `people`, `authors`, or both. So, both "Anne" and "John" would be removed from the cache:

	Cache::tags('people', 'authors')->flush();

In contrast, this statement would remove only caches tagged with `authors`, so "John" would be removed, but not "Anne".

	Cache::tags('authors')->flush();

<a name="database-cache"></a>
## Veritabanı Önbelleği

`Veritabanı` önbellek sürücüsü kullanırken, önbellek öğelerini içeren bir tablo kurulumu gerekir. Bu tablo için örnek bir `şema` aşağıda gösterilmiştir:

	Schema::create('cache', function($table)
	{
		$table->string('key')->unique();
		$table->text('value');
		$table->integer('expiration');
	});
