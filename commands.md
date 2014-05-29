# Artisan'ın Geliştirilmesi

- [Giriş](#introduction)
- [Komut Oluşturulması](#building-a-command)
- [Komutların Kayıt Ettirilmesi](#registering-commands)
- [Diğer Komutların Çağırılması](#calling-other-commands)

<a name="introduction"></a>
## Giriş

Artisan'da mevcut olan komutlara ilaveten,  uygulamanız ile çalışacak olan kendi özel komutlarınızı inşa edebilirsiniz. Bu özel komutlarınızı  `app/commands` dizininde depolayabilirsiniz. Komutlarınızı kendi istediğiniz başka bir dizinde de depolayabilirsiniz. Bunun için, bu komutlarınızın `composer.json` ayarlarınız bazında "autoload" edilebiliyor olması gerekmektedir.

<a name="building-a-command"></a>
## Komut Oluşturulması

### Sınıfının Oluşturulmması

Yeni bir komut oluşturmak için, `command:make` Artisan komutunu kullanabilirsiniz. Bu komut, başlamanızda size yardımcı olmak için yeni bir komut taslağı oluşturacaktır.

#### Yeni Bir Komut Sınıfının Oluşturulması

	php artisan command:make FalancaKomut

Ön tanımlı olarak, oluşturulan komutlar `app/commands` dizininde depolanırlar. Fakat, siz başka bir dizin veya bir 'namespace' de belirleyebilirsiniz.

	php artisan command:make FalancaKomut --path=app/classes --namespace=Siniflar

Bir komut oluşturulurken terminal komut adı atamak için `--command` seçeneği kullanılabilir:

	php artisan command:make AssignUsers --command=users:assign

### Komutun Yazılışı

Komut oluşturulduktan sonra, komutu `list` ekranında görüntülerken kullanılacak olan, sınıf ismi `name` ve tanımı `description` özellikleri doldurulmalıdır.

Komut çalıştırıldığında `fire` (ateşle) metodu çağırılmaktadır. Bu metoda, istenecek herhangi bir komut mantığı yerleştirebilinir.

### Argümanlar & Seçenekler

Komutunuzun alacağı argüman veya seçenekleri tanımlayabileceğiniz yerler `getArguments` ve `getOptions` metodlarıdır. Bu metodların her ikisi de birer komut dizisi verirler. Bu komut dizileri, bir 'dizi seçenekleri listesi' ile tarif edilirler.

Argümanları (`arguments`) belirlerlerken, dizi tanımı değerleri şunları belirler: (ismi, modu, tanımı, ön değeri)

	array($name, $mode, $description, $defaultValue)

Modu argümanı `mode` şunlardan herhangi biri olabilir: `InputArgument::REQUIRED` (mecburi) veya `InputArgument::OPTIONAL` (isteğe bağlı).

Seçenekleri (`options`) belirlerken, dizi tanımı değerleri şunları belirler: (ismi, kısayolu, modu, tanımı, ön değeri)

	array($name, $shortcut, $mode, $description, $defaultValue)

Seçenekler için, modu argümanı `mode` şunlardan biri olabilir: `InputOption::VALUE_REQUIRED` (Girdi Seçeneği: mecburi), `InputOption::VALUE_OPTIONAL` (isteğe bağlı), `InputOption::VALUE_IS_ARRAY` (dizi), `InputOption::VALUE_NONE` (yok).

`VALUE_IS_ARRAY` (Girdi Seçeneği: dizi) modu, komut çağırılırken, anahtarın birden çok kez kullanılabilir oldugunu belirtir:

	php artisan falan --option=filan --option=fesmekan

`VALUE_NONE` (Girdi Seçeneği: yok) modu, seçeneğin sadece bir "anahtar" olarak kullanıldığını belirtir.

	php artisan falan --option

### Girdilerin Çağırılması

Komutunuz çalışırken, uygulamanızın kabul edeceği argüman ve seçenek değerlerine ulaşabilmeniz gerekecektir. Bunu yapabilmek için argüman `argument` ve seçenek `option` metodlarını kullanabilirsiniz:

#### Bir Komut Argüman Değerinin Çağırılması

	$value = $this->argument('ismi');

#### Tüm Argümanların Birden Çağırılması

	$arguments = $this->argument();

#### Bir Komut Seçeneği Değerinin Çağırılması

	$value = $this->option('ismi');

#### Tüm Seçeneklerin Birden Çağırılması

	$options = $this->option();

### Çıktı Yazılışı

Çıktının konsola gönderilmesi için, `info` (bilgi), `comment` (not), `question` (soru) ve `error` (hata) metodlarını kullanabilirsiniz. Bu metodların her biri, kendi amaçlarına uygun olan ANSI renklerini kullanacaktır.

#### Konsola Bilgi Gönderilmesi

	$this->info('Bunu ekranda göster');

#### Konsola Bir Hata Mesajı Gönderilmesi

	$this->error('Bir hata oluştu!');

### Soruların Soruluşu

Kullanıcıdan bir girdi talep etmek için, `ask` (sor) ve `confirm` (onayla) metodlarını kullanabilirsiniz.

#### Kullanıcıya Girdi Bilgisinin Soruluşu

	$name = $this->ask('İsminiz nedir?');

#### Kullanıcıya Gizli Şifre Bilgisinin Soruluşu

	$password = $this->secret('Lütfen şifrenizi giriniz!');

#### Kullanıcıya Onayının Soruluşu

	if ($this->confirm('Devam etmek istiyor musunuz? [evet|hayır]'))
	{
		//
	}

İsterseniz `confirm` (onayla) metoduna, `true` (evet) ve `false` (hayır) seçeneklerinden birini varsayılan ön değer olarak belirleyebilirsiniz :

	$this->confirm($soru, true);

<a name="registering-commands"></a>
## Komutların Kayıt Ettirilmesi

#### Bir Artisan Komutunun Kayıt Ettirilişi

Komutunuzun inşa edilmesi tamamlandığında, kullanılmaya hazır olabilmesi için, Artisan'da kayıt ettirmeniz gerekir. Bu, genelde `app/start/artisan.php` dosyası içerisinde yapılır. Bu dosya içerisinde, kayıt ettirmek için `Artisan::add` metodunu kullanabilirsiniz.

	Artisan::add(new FalancaKomut);

#### IoC Container'da Olan Bir Komutun Kayıt Ettirilişi

Eğer komutunuz [IoC container](/docs/ioc) uygulamasında kayıtlı ise, Artisan'da da kullanılabilir olması için `Artisan::resolve` metodunu kullanabilirsiniz.

	Artisan::resolve('binding.ismi');

#### Komutların Bir Hizmet Sağlayıcısında Kayda Geçirilmesi

Eğer komutları bir hizmet sağlayıcısı içerisinden kayda geçirmeniz gerekirse, sağlayıcının `boot` metodundan `commands` metodunu çağırmalı ve çağırırken ilgili komutun [IoC konteyneri](/docs/ioc) bağlamasını geçmelisiniz:

	public function boot()
	{
		$this->commands('command.binding');
	}

<a name="calling-other-commands"></a>
## Diğer Komutların Çağırılması

Bazı durumlarda, komutunuzun içerisinden başka bir komutu çağırmak isteyebilirsiniz. Bunu, `call` metodunu kullanarak yapabilirsiniz:

	$this->call('command.ismi', array('argument' => 'falan', '--option' => 'filan'));
