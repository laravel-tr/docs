# Görünümler (Views) ve Cevaplar (Responses)

- [Temel Cevaplar](#basic-responses)
- [Yönlendirmeler (Redirects)](#redirects)
- [Görünümler (Views)](#views)
- [Görünüm Kompozitörleri](#view-composers)
- [Özel Cevaplar](#special-responses)
- [Cevap Makroları](#response-macros)

<a name="basic-responses"></a>
## Temel Cevaplar

#### Rotalardan String Döndürme

	Route::get('/', function()
	{
		return 'Merhaba dünya!';
	});

#### Özel Cevaplar Oluşturma

Bir cevap (`Response`) olgusu `Symfony\Component\HttpFoundation\Response` sınıfından türer ve HTTP cevapları oluşturmak için çeşitli metodlar sağlar.

	$cevap = Response::make($contents, $statusCode);

	$cevap->header('Content-Type', $deger);

	return $cevap;

Eğer `Response` sınıfının metodlarına erişmeniz gerekiyor, ama cevap içeriği olarak bir view döndürmek istiyorsanız, kolaylık açısından `Response::view` metodunu kullanabilirsiniz:

	return Response::view('hello')->header('Content-Type', $type);

#### Cevaplara Çerez Bağlanması

	$cerez = Cookie::make('isim', 'deger');

	return Response::make($content)->withCookie($cerez);

<a name="redirects"></a>
## Yönlendirmeler (Redirects)

#### Bir Yönlendirme Döndürme

	return Redirect::to('uye/giris');

#### Flaş Veri Eşliğinde Bir Yönlendirme Döndürme

	return Redirect::to('uye/giris')->with('mesaj', 'Giriş başarısız!');

> **Not:** `with` metodu veriyi oturum bilgisine flaşlayacağından, bu veriyi tipik `Session::get` metodu ile alabilirsiniz.

#### İsimli Bir Rotaya Yönlendirme Döndürme

	return Redirect::route('giris');

#### Parametre Geçerek İsimli Bir Rotaya Yönlendirme Döndürme

	return Redirect::route('profil', array(1));

#### İsimli Parametre Kullanarak İsimli Bir Rotaya Yönlendirme Döndürme

	return Redirect::route('profil', array('uye' => 1));

#### Bir Kontrolör Eylemine Yönlendirme Döndürme

	return Redirect::action('HomeController@index');

#### Parametre Geçerek Bir Kontrolör Eylemine Yönlendirme Döndürme

	return Redirect::action('UserController@profil', array(1));

#### İsimli Parametre Kullanarak Bir Kontrolör Eylemine Yönlendirme Döndürme

	return Redirect::action('UserController@profil', array('uye' => 1));

<a name="views"></a>
## Görünümler (Views)

Görünümler tipik olarak uygulamanızın HTML'sini içerirler ve kontrolörünüzün ve etki alanı mantığınızın gösterim mantığınızdan ayrık tutulmasının uygun bir yoludur. Görünümler `app/views` dizininde saklanmaktadır.

Basit bir görünüm şuna benzer:

	<!-- Görünüm app/views/selamlama.php dosyasında bulunsun-->

	<html>
		<body>
			<h1>Merhaba <?php echo $isim; ?></h1>
		</body>
	</html>

Bu görünüm web tarayıcısına şu şekilde döndürülebilir:

	Route::get('/', function()
	{
		return View::make('selamlama', array('isim' => 'Tuana Şeyma'));
	});

`View::make` metodundaki ikinci parametre görünümde kullanılması gereken bir veri dizisidir.

#### Görünümlere Veri Geçilmesi

	// Geleneksel yaklaşım kullanmak
	$view = View::make('selamlama')->with('isim', 'Tuana Şeyma');

	// Sihirli Metodları kullanmak
	$view = View::make('selamlama')->withIsim('Tuana Şeyma');

Yukarıdaki örnekte `$isim` değişkeni görünümden erişilebilir olacak ve `Tuana Şeyma` bilgisini taşıyacaktır.

Dilerseniz, `make` metoduna ikinci parametre olarak veriler dizisi geçebilirsiniz:

	$view = View::make('selamlama', $data);

Bir parça veriyi tüm görünümler arasında paylaşmanız da mümkündür:

	View::share('isim', 'Tuana Şeyma');

#### Bir Görünüme Bir Alt Görünüm Geçirilmesi

Bazen bir görünümü başka bir görünümün içine geçirmek isteyebilirsiniz. Örneğin, `app/views/evlat/view.php`'de saklanan belli bir görünüm olsun ve biz bunu şu şekilde başka bir görünüme geçirebiliriz:

	$view = View::make('selamlama')->nest('evlat', 'evlat.view');

	$view = View::make('selamlama')->nest('evlat', 'evlat.view', $data);

Bundan sonra bu alt görünüm ebeveyn görünümde gösterilebilir:

	<html>
		<body>
			<h1>Merhaba!</h1>
			<?php echo $evlat; ?>
		</body>
	</html>

#### Bir Görünümün Mevcut Olmadığının Belirlenmesi
  
Bir view'in mevcut olup olmadığını yoklamanız gerekirse, `View::exists` metodunu kullanın:
  
	if (View::exists('emails.customer'))
	{
		//
  	}
  
<a name="view-composers"></a>
## Görünüm Kompozitörleri

Görünüm kompozitörleri görünüm oluşturulduğu zaman çağrılan isimsiz fonksiyonlar veya sınıf metodlarıdır. Eğer belli bir görünüm, uygulamanız boyunca her oluşturulduğunda bu görünüme bağlamak istediğiniz bir veri varsa, bir görünüm kompozitörü kodun tek bir yere koyulabilmesi imkanı verebilir. Bu nedenle, görünüm kompozitörleri "görünüm modelleri" veya "sunum yapıcı" gibi iş görürler.

#### Bir Görünüm Kompozitörü Tanımlanması

	View::composer('profil', function($view)
	{
		$view->with('navigasyon', Sayfa::all());
	});

Şimdi `profil` görünümü her oluşturulduğunda, `navigasyon` verisi bu görünüme bağlanacaktır.

Bir görünüm kompozitörüne bir defada birden çok görünüm bağlamanız da mümkündür:

    View::composer(array('profil','pano','bildirim'), function($view)
    {
        $view->with('navigasyon', Sayfa::all());
    });

Bunun yerine sınıf tabanlı bir kompozitör kullanmak isterseniz, ki uygulama [IoC Konteyneri](/docs/ioc) ile çözümlenebilme yararı sağlar, şöyle yapabilirsiniz:

	View::composer('profil', 'ProfileComposer');

Bir görünüm kompozitörü sınıfı şöyle tanımlanmalıdır:

	class ProfileComposer {

		public function compose($view)
		{
			$view->with('adet', Uye::count());
		}

	}

#### Birden Çok Composer Tanımlanması

Bir grup composer'i bir defada kayda geçirmek için `composers` metodunu kullanabilirsiniz:

	View::composers(array(
		'AdminComposer' => array('admin.index', 'admin.profile'),
		'UserComposer' => 'user',
	));

> **Not:** Kompozitör sınıfının nerede saklanacağı konusunda bir gelenek olmadığına dikkat edin. `composer.json` dosyanızdaki yönergeleri kullanarak otomatik yüklenebildikleri sürece, bunları istediğiniz yerde depolayabilirsiniz.

### Görünüm Oluşturucular

Görünüm **oluşturucuları** tam olarak görünüm kompozitörleri gibi çalışırlar; ancak bunlar görünüm oluşturulur oluşturulmaz aktifleştirilirler. Görünüm oluşturucusu kaydetmek için, basitçe `creator` metodunu kullanınız:

	View::creator('profil', function($view)
	{
		$view->with('adet', Uye::count());
	});

<a name="special-responses"></a>
## Özel Cevaplar

#### Bir JSON Cevabı Oluşturma

	return Response::json(array('isim' => 'Tuana Şeyma', 'il' => 'Bursa'));

#### Bir JSONP Cevabı Oluşturma

	return Response::json(array('isim' => 'Tuana Şeyma', 'il' => 'Bursa'))->setCallback(Input::get('callback'));

#### Bir Dosya İndirme Cevabı Oluşturma

	return Response::download($indirilecekDosyaYolu);

	return Response::download($indirilecekDosyaYolu, $isim, $basliklar);

> **Not:** Dosya indirmelerini yöneten Symfony HttpFoundation, indirilecek olan dosyanın bir ASCII dosya ismi olmasını gerektirir.

<a name="response-macros"></a>
## Cevap Makroları

Çeşitli rota ve controllerlerinizde tekrar tekrar kullanabileceğiniz özel bir cevap tanımlamak isterseniz, `Response::macro` metodunu kullanabilirsiniz:

	Response::macro('caps', function($value)
	{
		return Response::make(strtoupper($value));
	});

Bu `macro` fonksiyonu birinci parametre olarak bir isim ve ikinci parametre olarak bir Closure kabul eder. `Response` sınıfı üzerinde makro ismi çağrıldığı zaman makronun Closure fonksiyonu çalıştırılacaktır:

	return Response::caps('falan');

Makrolarınızı `app/start` dosyalarınızın birinde tanımlayabilirsiniz. Alternatif olarak, makrolarınızı ayrı bir dosya içerisinde organize edip, bu dosyayı `start` dosyalarınızın birisinden "include" edebilirsiniz.
