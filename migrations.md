# Migrasyon (Migration) ve Veri Ekme (Seeding)

- [Giriş](#introduction)
- [Migrasyonların Oluşturulması](#creating-migrations)
- [Migrasyonların Çalıştırılması](#running-migrations)
- [Migrasyonların Geriye Döndürülmesi](#rolling-back-migrations)
- [Veritabanına Veri Ekme](#database-seeding)

<a name="introduction"></a>
## Giriş

Migrasyonlar veritabanı için bir sürüm kontrol türüdür. Bir ekibin veritabanı şemasını değiştirmesine ve son şema durumuna güncel kalmalarına imkan verir. Migrasyonlar uygulama şemasını kolayca yönetmek amacıyla tipik olarak [Şema (Schema) Kurucu](/docs/schema) ile eşleştirilirler.

<a name="creating-migrations"></a>
## Migrasyonların Oluşturulması

Bir migrasyon oluşturmak için, Artisan KSA'da (Artisan Komut Satırı Arayüzü) `migrate:make` komutunu kullanabilirsiniz:

#### Bir Migrasyon Oluşturulması

	php artisan migrate:make kullanicilar_tablosunu_olustur

Migrasyon `app/database/migrations` dizininize konumlandırılır ve Laravel'in migrasyonların sırasını belirlemesine imkan veren bir zaman damgası içerir.

Migrasyonu oluştururken bir patika `--path` seçeneği de belirtebilirsiniz. Patika, kurulum kök dizinine göreceli olmalıdır:

	php artisan migrate:make falancaMigrasyon --path=app/migrations

Tablo ismini ve yeni bir tablonun oluşturulacağını da, tablo `--table` ve oluştur `--create` seçeneklerini kullanarak belirtebilirsiniz:

	php artisan migrate:make kullanicilar_tablosunu_olustur --table=users

	php artisan migrate:make kullanicilar_tablosuna_oy_verenler_ekle --create=users

<a name="running-migrations"></a>
## Migrasyonların Çalıştırılması

#### Bekleyen Migrasyonların Hepsinin Birden Çalıştırılması

	php artisan migrate

#### Bir Patikadaki Migrasyonların Çalıştırılması

	php artisan migrate --path=app/falancaDizin/migrations

#### Bir Paketin Tüm Bekleyen Migrasyonlarının Çalıştırılması

	php artisan migrate --package=vendor/package

> **Not:** Migrasyonlar çalıştırırken, "class not found" (sınıf bulunamadı) hatası veririse, `composer update` komutunu çalıştırarak deneyiniz.

<a name="rolling-back-migrations"></a>
## Migrasyonların Geriye Döndürülmesi

#### Son Migrasyon İşleminin Geriye Döndürülmesi

	php artisan migrate:rollback

#### Tüm Migrasyon İşlemlerinin Geriye Döndürülmesi

	php artisan migrate:reset

#### Tüm Migrasyon İşlemlerinin Geriye Döndürülmesi ve Hepsinin Tekrardan Çalıştırılması

	php artisan migrate:refresh		//Veri ekmeler dahil edilmeden

	php artisan migrate:refresh --seed	//Veri ekmeler dahil edilerek

<a name="database-seeding"></a>
## Veritabanına Veri Ekme

Veri Ekme (seeding), migrasyon ile oluşturulacak veritabanı tablosunda gerekli olacak ilk veri kayıtlarının (seed data) oluşturulması işlemidir(:çevirenin notu). Laravel, veritabanınızın deneme verisi ile veri ekme için kolaylık sağlayacak olan veri ekme (seed) sınıflarını bulundurur. Bütün veri ekme sınıfları `app/database/seeds` dizininde konumlandırılır. Veri ekme sınıflarına istediğiniz isimleri verebilirsiniz. Fakat isimlendirirken anlaşılacak belli bir düzene (convention) uyulması lehinizedir, örneğin `KullanicilarTablosunaVeriEkme`, vb. Ön tanımlı olarak, sizin için bir DatabaseSeeder sınıfı tanımlanmıştır. Veri ekme sırasını denetlemenize imkan verecek olan, bu sınıfın 'çağır' `call` metodunu kullanarak diğer veri ekme sınıflarınızı çalıştırabilirsiniz.

#### Veritabanı Veri Ekme Sınıfı Örneği

	class DatabaseSeeder extends Seeder {

		public function run()
		{
			$this->call('KullanicilarTablosunaVeriEkme');

			$this->command->info('Kullanıcı tablosuna veri ekildi!');
		}

	}

	class KullanicilarTablosunaVeriEkme extends Seeder {

		public function run()
		{
			DB::table('kullanicilar')->delete();

			User::create(array('email' => 'falanca@filanca.com'));
		}

	}

Veritabanına veri ekmek için, Artisan KSA'da `db:seed` (veri ek) komutunu kullanabilirsiniz:

	php artisan db:seed

By default, the `db:seed` command runs the `DatabaseSeeder` class, which may be used to call other seed classes. However, you may use the `--class` option to specify a specific seeder class to run individually:

	php artisan db:seed --class=UserTableSeeder

Veritabanına `migrate:refresh` (yenile) komutunu kullanarak da veri ekebilirsiniz, bu komut aynı zamanda bütün migrasyonları geriye döndürüp, hepsini tekrardan çalıştıracaktır:

	php artisan migrate:refresh --seed