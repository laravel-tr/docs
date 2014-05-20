# İstek Yaşam Döngüsü

- [Genel Bakış](#overview)
- [İstek Yaşam Döngüsü](#request-lifecycle)
- [Start Dosyaları](#start-files)
- [Application Olayları (Events)](#application-events)

<a name="overview"></a>
## Genel Bakış

"Gerçek dünyada" bir alet kullanırken, aletin nasıl kullanıldığını bilirseniz kendinizi daha güvende hissedersiniz. Uygulama geliştirme farklı değildir. Geliştirme aletlerinizin nasıl fonksiyon gördüklerini bilirseniz bunları kullanırken kendinizi daha rahat ve güvende hissedersiniz. Bu dokümanın amacı Laravel frameworkünün nasıl "çalıştığının" iyi, yüksek düzeyli bir özetini vermektir. Tüm frameworkün daha iyi tanınmasıyla, her şey daha az "büyülü" hissedilecek ve uygulamanızı inşa ederken daha güvenli olacaksınız. İstek yaşam döngüsünün yüksek düzeyli bir özetine ek olarak "start" dosyalarını ve application olaylarını da anlatacağız.

Terimlerin hepsini hemencecik anlamadıysanız, inancınızı kaybetmeyin! Sadece ne olup bittiğini temel olarak kavrayama çalışın. Dokümantasyonun diğer kesimlerini inceledikçe bilginiz giderek büyüyecektir.

<a name="request-lifecycle"></a>
## İstek Yaşam Döngüsü

Uygulamanıza gelen tüm istekler `public/index.php` skripti aracılığı ile yönetilir. Apache kullanırken, Laravel'le gelen `.htaccess` dosyası tüm isteklerin `index.php`'ye geçirilmesi işini halletmektedir. Bu noktadan itibaren, Laravel istekleri işleme ve istemciye bir cevap döndürme sürecine başlar. Laravel bootstrap süreci hakkında genel bir fikir elde edilmesi yararlı olacaktır, bu nedenle şimdi onu anlatacağız!

Laravel'in bootstrap sürecini öğrenirken kavranması gereken en önemli kavram **Servis Sağlayıcılarıdır**. Kendinizin `app/config/app.php` yapılandırma dosyasını açıp, buradaki `providers` dizisine bakarak servis sağlayıcılarının bir listesini görebilirsiniz. Bu sağlayıcılar Laravel için başlıca bootstrapping mekanizması olarak hizmet ederler. Fakat, servis sağlayıcıları konusuna daha fazla girmeden önce `index.php` dosyasına geri dönelim. Bir istek sizin `index.php` dosyasına girdikten sonra, `bootstrap/start.php` dosyası yüklenecektir. Bu dosya, aynı zamanda bir [IoC konteyneri](/docs/ioc) olarak hizmet eden yeni bir Laravel `Application` nesnesi oluşturur.

`Application` nesnesi oluşturulduktan sonra, birkaç proje path'i ayarlanacak ve [ortam tespiti](/docs/configuration#environment-configuration) yapılacaktır. Ondan sonra, dahili bir Laravel bootstrap skripti çağrılacaktır. Bu dosya Laravel kaynağının derinliklerinde yaşar ve yapılandırma dosyalarınıza dayalı olarak zaman dilimi (timezone), hata bildirimi ve benzeri birkaç ayarı daha ayarlar. Fakat, oldukça önemsiz yapılandırma seçeneklerinin ayarlanmasına ek olarak, aynı zamanda çok önemli bir şey daha yapar: uygulamanız için yapılandırılmış servis sağlayıcılarının hepsini kayda geçirir.

Basit servis sağlayıcılarında sadece bir metod vardır: `register`. Bu `register` metodu, Application nesnesi tarafından Application'un kendi `register` metodu aracılığıyla bir servis sağlayıcısı kayda geçirildiği zaman çağrılır. Bu metod içerisinde, servis sağlayıcıları bir şeyleri [IoC konteynerinde](/docs/ioc) kayda geçirirler. Esasında, her servis sağlayıcı konteynere bir veya daha fazla [closure](http://us3.php.net/manual/en/functions.anonymous.php) (anonim fonksiyon) bağlar ki, bu closure'lar uygulamanız içinde bağlanmış hizmetlere erişmenize imkan verirler. Yani, örnek olarak `QueueServiceProvider` servis sağlayıcısı [Kuyruk (Queue)](/docs/queues) ile ilgili çeşitli sınıfları çözümleyen closure'leri kayda geçirmektedir. Pek tabii, servis sağlayıcıları sadece bir şeyleri IoC konteynerinde kayda geçirmekte değil, her türlü bootstrapping işi için kullanılabilirler. Bir servis sağlayıcı olay dinleyicilerini, view composer'lerini, Artisan komutlarını ve başkalarını kayda geçirebilirler.

Servis sağlayıcılarının hepsi kayda geçirildikten sonra, `app/start` dosyalarınız yüklenecektir. Son olarak, `app/routes.php` dosyanız yüklenecektir. `routes.php` dosyanız yüklendikten sonra, Request nesnesi bir rotaya sevk edilebilmesi için "application"a gönderilir.

Özetleyecek olursak:

1. İstek `public/index.php` dosyasına girer.
2. `bootstrap/start.php` dosyası "Application"ı oluşturur ve ortamı tespit eder.
3. Dahili `framework/start.php` dosyası ayarları yapılandırır ve servis sağlayıcılarını yükler.
4. Application `app/start` dosyaları yüklenir.
5. Application `app/routes.php` dosyası yüklenir.
6. Request nesnesi Application'a gönderilir, o da Response nesnesi döndürür.
7. Response nesnesi istemciye geri gönderilir.

Artık bir Laravel uygulamasına gelen bir isteğin nasıl işlendiği konusunda iyi bir fikre sahip olduğunuza göre, "start" dosyalarına daha yakından bakabiliriz!

<a name="start-files"></a>
## Start Dosyaları

Uygulamanızın start dosyaları `app/start` dizininde bulunmaktadır. Varsayılan olarak bunlardan üçü uygulamanızın içine dahil edilmiştir. Bunlar `global.php`, `local.php` ve `artisan.php`'dir. `artisan.php` hakkında daha fazla bilgiye sahip olmak için [Artisan komut satırı](/docs/commands#registering-commands) dökümanlarına bakınız.

Bunlardan `global.php` start dosyası [Günlüklerin](/docs/errors) kayda geçirilmesi ve `app/filters.php` dosyanızın dahil edilmesi gibi ön tanımlı birkaç temel öğe içerir. Ancak, bu dosyaya istediğiniz her şeyi ekleyebilirsiniz. Bu dosya ortam ne olursa olsun uygulamanıza gelen _her_ istekte otomatik olarak dahil edilecektir. Öte yandan `local.php` dosyası yalnızca uygulamanız `local` ortamda çalışırken çağrılır. Ortamlar hakkında daha fazla bilgi için [Yapılandırma](/docs/configuration) belgelerine bakınız.

`local`'e ilaveten başka ortamlarınız da varsa, pek tabii bu ortamlar için de start dosyaları oluşturabilirsiniz. Uygulamanız o ortamda çalıştığı zaman bunlar otomatik olarak dahil edileceklerdir. Dolayısıyla, örneğin eğer `bootstrap/start.php` dosyanızda yapılandırılmış olan bir `development` ortamına sahipseniz, bir `app/start/development.php` dosyası oluşturabilirsiniz; herhangi bir istek uygulamaya bu ortamda girdiği zaman bu dosya dahil edilecektir.

### Start Dosyalarına Koyulacak Şeyler

Start dosyaları her türlü "bootstrapping" kodun koyulacağı basit bir yer olarak hizmet ederler. Örneğin, bir View composer'ı kayda geçirebilir, günlükleme tercihlerinizi yapılandırabilir, bazı PHP ayarlarını ve benzerlerini yapabilirsiniz. Bu tamamen size kalmış. Tabii ki, tüm önyükleme kodunuzun start dosyalarına atılması bir karışıklık ve dağınıklık oluşturabilir. Büyük uygulamalar için veya eğer start dosyalarınızın karışmaya başladığını hissederseniz, bootstrapping kodunuzun bir kısmını [servis sağlayıcılarına](/docs/ioc#service-providers) taşımayı düşünün.

<a name="application-events"></a>
## Application Olayları (Events)

#### Uygulama Olaylarının Kayda Geçirilmesi

Bunlara ek olarak `before`, `after`, `close`, `finish` ve `shutdown` uygulama olaylarını kayda geçirmek suretiyle istek öncesinde ve sonrasında bazı işlemler de yapabilirsiniz:

	App::before(function($request)
	{
		//İstek öncesi olayları
	});

	App::after(function($request, $response)
	{
		//İstek sonrası olayları
	});

Bu olay dinleyicileri, uygulamanıza yapılan her istek öncesinde `(before)` ve sonrasında `(after)` çalışacaktır. Bu olaylar cevapların global filtrelenmesi veya global modifikasyonu için yardımcı olabilirler. Bunları `start` dosyalarınızın birinde veya bir [servis sağlayıcısında](/docs/ioc#service-providers) kayda geçirebilirsiniz.

Bir dinleyiciyi `matched` eventinde de kayda geçirebilirsiniz; bu, gelen bir istek bir rotayla eşleştirildiğinde, ancak rota daha henüz çalıştırılmadan önce ateşlenecektir:

	Route::matched(function($route, $request)
	{
		//
	});

`finish` eventi bir cevap sizin uygulamanızdan istemciye geri gönderildikten sonra çağrılır. Burası uygulamanızın gerektirdiği bir son dakika işlemini yapmak için iyi bir yerdir. `shutdown` eventi ise tüm `finish` olay işleyicileri işlemlerini bitirdikten hemen sonra çağrılır ve skript sona ermeden önce herhangi bir iş yapmak için son fırsattır. Büyük ihtimalle, bu olaylardan birini kullanma ihtiyacınız olmayacaktır.