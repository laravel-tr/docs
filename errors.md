# Hatalar ve Günlüğe Ekleme

- [Yapılandırma](#configuration)
- [Hataların İşlenmesi](#handling-errors)
- [HTTP İstisnaları](#http-exceptions)
- [Günlüğe Ekleme](#logging)

<a name="configuration"></a>
## Yapılandırma

The logging facilities for your application are configured in the `Illuminate\Foundation\Bootstrap\ConfigureLogging` bootstrapper class. This class utilizees the `log` configuration option from your `config/app.php` configuration file.

By default, the logger is configured to use daily log files; however, you may customize this behavior as needed. Since Laravel uses the popular [Monolog](https://github.com/Seldaek/monolog) logging library, you can take advantage of the variety of handlers that Monolog offers.

Örneğin, tek bir büyük dosya yerine günlük log dosyaları kullanmak istiyorsanız, `config/app.php` dosyanızda aşağıdaki değişikliği yapabilirsiniz:

	'log' => 'single'

Out of the box, Laravel supported `single`, `daily`, and `syslog` logging modes. However, you are free to customize the logging for your application as you wish by overriding the `ConfigureLogging` bootstrapper class.

### Hata Ayrıntısı

The amount of error detail your application displays through the browser is controlled by the `app.debug` configuration option in your `config/app.php` configuration file. By default, this configuration option is set to respect the `APP_DEBUG` environment variable, which is stored in your `.env` file.

For local development, you should set the `APP_DEBUG` environment variable to `true`. **In your production environment, this value should always be `false`.**

<a name="handling-errors"></a>
## Hataların İşlenmesi

All exceptions are handled by the `App\Exceptions\Handler` class. This class contains two methods: `report` and `render`.

The `report` method is used to log exceptions or send them to an external service like [BugSnag](https://bugsnag.com). By default, the `report` method simply passes the exception to the base implementation on the parent class where the exception is logged. However, you are free to log exceptions however you wish. If you need to report different types of exceptions in different ways, you may use the PHP `instanceof` comparison operator:

	/**
	 * Report or log an exception.
	 *
	 * This is a great spot to send exceptions to Sentry, Bugsnag, etc.
	 *
	 * @param  \Exception  $e
	 * @return void
	 */
	public function report(Exception $e)
	{
		if ($e instanceof CustomException)
		{
			//
		}

		return parent::report($e);
	}

The `render` method is responsible for converting the exception into an HTTP response that should be sent back to the browser. By default, the exception is passed to the base class which generates a response for you. However, you are free to check the exception type or return your own custom response.

The `dontReport` property of the exception handler contains an array of exception types that will not be logged. By default, exceptions resulting from 404 errors are not written to your log files. You may add other exception types to this array as needed.

<a name="http-exceptions"></a>
## HTTP İstisnaları

Bazı istisnalar, sunucu tarafından HTTP hata kodları olarak tanımlanır. Örneğin, Bu bir "sayfa bulunamadı"" hatası (404), bir yetkisizlik hatası (401), hatta genel 500 hatası olabilir. Böyle bir cevap döndürmek için aşağıdaki biçimi kullanın:

	abort(404);

İsteğe bağlı olarak, bir cevap verebilirsiniz:

	abort(403, 'Yetkili değilsiniz.');

Bu istisnalar, isteğin yaşam döngüsü boyunca herhangi bir zamanda kullanılabilir.

### Özel 404 Hata Sayfaları

Tüm 404 hataları için özel bir hata sayfası döndürmek için, bir tane `resources/views/errors/404.blade.php` dosyasını oluşturun. Sizin uygulamanız tarafından oluşturulan bütün 404 hatalarında bu sayfa sunulacaktır:

<a name="logging"></a>
## Günlüğe Ekleme

Laravel'in günlüğe ekleme imkanları güçlü [Monolog](http://github.com/seldaek/monolog) üstünde basit bir katman sağlar. Laravel, ön tanımlı olarak uygulamanız için günlük dosyalar oluşturacak şekilde yapılandırılmıştır ve bu dosya `storage/logs` içinde tutulmaktadır. Bu günceye aşağıdakilere benzer şekilde bilgi yazabilirsiniz:

	Log::info('İşte bu yararlı bir bilgidir.');

	Log::warning('Yanlış giden bir şeyler olabilir.');

	Log::error('Gerçekten yanlış giden bir şey var.');

Günlük tutucu, [RFC 5424](http://tools.ietf.org/html/rfc5424)'de tanımlanmış yedi günlük ekleme düzeyi sağlamaktadır: **debug**, **info**, **notice**, **warning**, **error**, **critical** ve **alert**.

Log metodlarına bağlamsal bir veri dizisi de geçilebilir:

	Log::info('Log message', ['context' => 'Other helpful information']);

Monolog, günlüğe ekleme için kullanabileceğiniz bir takım başka işleyicilere de sahiptir. Gerektiğinde, Laravel tarafından kullanılan Monolog olgusuna şu şekilde ulaşabilirsiniz:

	$monolog = Log::getMonolog();

Ayrıca, günlüğe geçirilen tüm mesajları yakalamak için bir olay kaydı da yapabilirsiniz:

#### Bir günlük olay izleyici kaydı yapılması

	Log::listen(function($level, $message, $context)
	{
		//
	});
