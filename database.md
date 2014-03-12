# Temel Veritabanı Kullanımı

- [Yapılandırma](#configuration)
- [Read / Write Connections](#read-write-connections)
- [Sorguları Çalıştırma](#running-queries)
- [Veritabanı İşlemleri](#database-transactions)
- [Bağlantılara Erişme](#accessing-connections)
- [Sorgu Günlükleme](#query-logging)

<a name="configuration"></a>
## Yapılandırma

Laravel, veritabanı bağlantısını ve sorguları çalıştırmayı fazlasıyla kolay kılar. Veritabanı yapılandırma ayarları `app/config/database.php` dosyasında bulunmaktadır. Bu dosyada hem tüm veritabanı bağlantılarını tanımlayabilir, hem de hangi bağlantının varsayılan olarak kullanılacağını seçebilirsiniz. Bu dosyada, desteklenen veritabanı sistemlerinin tümü için örnekler verilmiştir.

Laravel tarafından desteklenen veritabanı sistemleri: MySQL, Postgres, SQLite, ve SQL Server.

<a name="read-write-connections"></a>
## Read / Write Connections

Sometimes you may wish to use one database connection for SELECT statements, and another for INSERT, UPDATE, and DELETE statements. Laravel makes this a breeze, and the proper connections will always be used whether you are using raw queries, the query builder, or the Eloquent ORM.

To see how read / write connections should be configured, let's look at this example:

	'mysql' => array(
		'read' => array(
			'host' => '192.168.1.1',
		),
		'write' => array(
			'host' => '196.168.1.2'
		),
		'driver'    => 'mysql',
		'database'  => 'database',
		'username'  => 'root',
		'password'  => '',
		'charset'   => 'utf8',
		'collation' => 'utf8_unicode_ci',
		'prefix'    => '',
	),

Note that two keys have been added to the configuration array: `read` and `write`. Both of these keys have array values containing a single key: `host`. The rest of the database options for the `read` and `write` connections will be merged from the main `mysql` array. So, we only need to place items in the `read` and `write` arrays if we wish to override the values in the main array. So, in this case, `192.168.1.1` will be used as the "read" connection, while `192.168.1.2` will be used as the "write" connection. The database credentials, prefix, character set, and all other options in the main `mysql` array will be shared across both connections.

<a name="running-queries"></a>
## Sorguları Çalıştırma

Veritabanı bağlantılarını bir kere yapılandırdıktan sonra `DB` sınıfını kullanarak sorgularını çalıştırabilirsiniz.

#### Kayıt Çekme (Select)

	$sonuclar = DB::select('select * from uyeler where id = ?', array(1));

`select` metodu sonuçları her zaman `dizi` tipinde döndürür.

#### Yeni Kayıt Ekleme (Insert)

	DB::insert('insert into uyeler (id, isim) values (?, ?)', array(1, 'Emre'));

#### Kayıt Güncelleme (Update)

	DB::update('update uyeler set oy = 100 where isim = ?', array('Hakan'));

#### Kayıt Silme (Delete)

	DB::delete('delete from uyeler');

> **Not:** `update` ve `delete` sorguları, bu işlemlerden etkilenen satır sayısını döndürür.

#### Genel Bir Sorgu Çalıştırma

	DB::statement('drop table uyeler');

`DB::listen` metodunu kullanarak sorgu olaylarını dinleyebilirsiniz:

#### Sorgu Olaylarını Dinleme

	DB::listen(function($sql, $bindings, $time)
	{
		//
	});

<a name="database-transactions"></a>
## Veritabanı İşlemleri

Bir veritabanı işleminde, birden fazla işlemi birden gerçekleştirmek için, 'transaction' metodunu kullanabilirsiniz:

	DB::transaction(function()
	{
		DB::table('uyeler')->update(array('votes' => 1));

		DB::table('posts')->delete();
	});

> **Note:** Any exception thrown within the `transaction` closure will cause the transaction to be rolled back automatically.

Sometimes you may need to begin a transaction yourself:

	DB::beginTransaction();

You can rollback a transaction via the `rollback` method:

	DB::rollback();

Lastly, you can commit a transaction via the `commit` method:

	DB::commit();

<a name="accessing-connections"></a>
## Bağlantılara Erişme

Birden fazla bağlantı kullandığınız durumlarda, bu bağlantılara `DB::connection` metodu aracılığı ile ulaşabilirsiniz.

	$uyeler = DB::connection('foo')->select(...);

Ayrıca temel PDO örneğine de ulaşabilirsiniz:

	$pdo = DB::connection()->getPdo();

Bazen veritabanına tekrar bağlanmaya ihtiyacınız olabilir.

	DB::reconnect('foo');

If you need to disconnect from the given database due to exceeding the underyling PDO instance's `max_connections` limit, use the `disconnect` method:

	DB::disconnect('foo');

<a name="query-logging"></a>
## Sorgu Günlükleme

Varsayılan olarak, Laravel güncel istek için çalıştırılabilecek tüm sorgular için bellekte bir günlük tutar. Bununla birlikte, bu bazı durumlarda, örneğin çok sayıda satır eklerken, uygulamanın aşırı bellek kullanmasına neden olabilir. Günlüğü devre dışı bırakmak için `disableQueryLog` metodunu kullanabilirsiniz.

	DB::connection()->disableQueryLog();

To get an array of the executed queries, you may use the `getQueryLog` method:

	$queries = DB::getQueryLog();