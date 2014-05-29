# Sürüm Notları

- [Laravel 4.2](#laravel-4.2)
- [Laravel 4.1](#laravel-4.1)

<a name="laravel-4.2"></a>
## Laravel 4.2

Bu sürümün tam değişiklik listesi bir 4.2 yüklemesinden `php artisan changes` komutunu vererek veya [Github'daki değişiklik dosyasına](https://github.com/laravel/framework/blob/4.2/src/Illuminate/Foundation/changes.json) bakarak görülebilir. Bu notlar sadece bu sürümdeki önemli geliştirmeleri ve değişiklikleri kapsamaktadır.

> **Not:** 4.2 salınım döngüsü süresince, çeşitli Laravel 4.1 nokta salımlarına birçok küçük hata düzeltmeleri ve geliştirmeler katılmıştır. Bu yüzden, Laravel 4.1 için değişiklik listesini de kontrol ettiğinizden emin olun!

### PHP 5.4 Gerekliliği

Laravel 4.2 PHP 5.4 veya daha üstünü gerektirir. Bu yükseltilmiş PHP  gerekliliği bize [Laravel Cashier](/docs/billing) benzeri araçlar için daha anlamlı ve etkileyici interface'ler için trait'ler gibi yeni PHP özelliklerini kullanmamıza imkan verir. PHP 5.4 aynı zamanda PHP 5.3'e göre önemli bir hız ve performans iyileştireleri de getirmektedir.

### Laravel Forge

Laravel Forge, web tabanlı yeni bir uygulama olup, aralarında Linode, DigitalOcean, Rackspace ve Amazon EC2'nin yer aldığı istediğiniz bir bulut üzerinde PHP sunucuları oluşturmak ve yönetmek için kolay bir yol sağlar. Otomatize Nginx yapılandırması, SSH key erişimi, Cron job otomasyonu, NewRelic & Papertrail aracılığıyla sunucu takibi, "Yayımlamak için Bas (Push To Deploy)", Laravel kuyruk işçisi yapılandırması ve pek çok şeyi desteklemek suretiyle, Forge sizin tüm Laravel uygulamalarınızın başlatılmasının en basit ve en uygun fiyatlı yolunu sağlar.

Varsayılan Laravel 4.2 yüklemenizin `app/config/database.php` yapılandırma dosyası, yepyeni uygulamalarınızı platforma daha uygun yayımlamanıza imkan verecek şekilde artık ön tanımlı olarak Forge kullanımı için yapılandırılmıştır.

Laravel Forge hakkında daha fazla bilgi [resmi Forge web sitesinde](https://forge.laravel.com) bulunabilir.

### Laravel Homestead

Laravel Homestead sağlam ve güçlü Laravel ve PHP uygulamaları geliştirilmesi için resmi bir Vagrant ortamıdır. Box'ların ihtiyaçlarının büyük çoğunluğu box dağıtım için paketlenmeden önce halledilir, böyleye box'un boot edilmesi son derece hızlıdır. Homestead'da Nginx 1.6, PHP 5.5.12, MySQL, Postgres, Redis, Memcached, Beanstalk, Node, Gulp, Grunt ve Bower yer almaktadır. Homestead birden çok Laravel uygulamasının tek bir box'ta yönetilmesi için basit bir `Homestead.yaml` yapılandırma dosyası içerir.

Varsayılan Laravel 4.2 yüklemesi şimdi artık box'un Homestead veritabanını kullanması, böylece Laravel'in ilk yükleme ve yapılandırmasının daha kolay olması için yapılandırılmış bir `app/config/local/database.php` yapılandırma dosyası taşımaktadır.

Ayrıca resmi dokümantasyon [Homestead](/docs/homestead) dokümantasyonunu içerecek şekilde güncellenmiştir.

### Laravel Cashier

Laravel Cashier, Stripe ile abonelik faturalaması yönetilmesi için basit, ifade edici bir kitaplıktır. Laravel 4.2'nin başlamasıyla birlikte biz Cashier dokümantasyonunu ana Laravel dokümantasyonuna dahil ediyoruz, ancak bu bileşenin yüklenmesi hala isteğe bağlıdır. Cashier'in bu salınımı çok sayıda hata düzeltmeleri, birden çok para birimi desteği ve en son Stripe API ile uyumluluk getirmektedir.

### Daemon Kuyruk İşçileri

Artisan `queue:work` komutu "deamon modunda" bir işçi başlatmak için şimdi bir `--daemon` seçeneğini destekliyor, bunun anlamı bu işçinin framework yeniden boot edilmesi hiç olmaksızın işleri işlemeye devam edeceğidir. Bu, hafifçe daha karmaşık bir uygulama yayımlama süreci maliyetiyle birlikte, CPU kullanımında önemli bir azalmayla sonuçlanır.

Daemon kuyruk işçileriyle ilgili daha fazla bilgi [queue documentation](/docs/queues#daemon-queue-workers) dokümantasyonunda bulunabilir.

### Mail API Sürücüleri

Laravel 4.2, `Mail` fonksiyonları için yeni Mailgun ve Mandrill API sürecilerini getirdi. Bu birçok uygulama için, e-mailler göndermenin SMTP seçeneğinden daha hızlı ve daha güvenilir bir yöntemini sağlar. Bu yeni sürücüler Guzzle 4 HTTP kitaplığını kullanmaktadır.

### Belirsiz Silme Trait'leri

PHP 5.4 trait'ler sayesinde "soft delete"ler ve diğer "global scope"ler için çok daha temiz bir mimari sunulmuştur. Bu yeni mimari benzer global trait'lerin daha kolay inşa edilmesini ve frameworkün kendisi içerisinde daha temiz bir "ilgilerin ayrılığı" sağlar.

Yeni `SoftDeletingTrait` hakkında daha fazla bilgi [Eloquent](/docs/eloquent#soft-deleting) dokümantasyonunda bulunabilir.

### Uygun Auth & Remindable Trait'leri

Varsayılan Laravel 4.2 yüklemesi authentication ve password reminder user interface'lerinin gerekli özellikleri içermesi için artık basit trait'ler kullanmaktadır. Bu, Laravel'le gelen default `User` model dosyasının çok daha temiz olmasını sağlamaktadır.

### "Basit Sayfalandırma"

Sayfalandırma view'inizde basit "Sonraki" ve "Önceki" linkleri kullanıyorken daha verimli sorgulara imkan vermek amacıyla Sorgu oluşturucusu ve Eloquent'e yeni bir `simplePaginate` metodu eklenmiştir.

### Migration Teyidi

Üretim ortamında, yıkıcı migrasyon işlemleri artık teyit isteyeceklerdir. `--force` seçeneği kullanılarak, komutlar herhangi bir teyit istemeksizin çalıştırılmaya zorlanabilir.

<a name="laravel-4.1"></a>
## Laravel 4.1

### Değişikliklerin Tam Listesi

Bu sürümün tam değişiklik listesi bir 4.1 yüklemesinden `php artisan changes` komutunu vererek veya [Github'daki değişiklik dosyasına](https://github.com/laravel/framework/blob/4.1/src/Illuminate/Foundation/changes.json) bakarak görülebilir. Bu notlar sadece bu sürümdeki önemli geliştirmeleri ve değişiklikleri kapsamaktadır.

### Yeni SSH Bileşeni

Bu sürümle birlikte tamamen yeni bir `SSH` bileşeni getirilmiştir. Bu özellik sizin uzak suncuculara kolaylıkla SSH iletişimi kurmanıza ve komut çalıştırmanıza imkan verir. Daha fazla öğrenmek için [SSH bileşeni dokümantasyonuna](/docs/ssh) bakın.

Yeni `php artisan tail` komutu yeni SSH bileşenini kullanmaktadır. Daha fazla bilgi için, `tail` [komut dokümantasyonuna](/docs/ssh#tailing-remote-logs) bakın.

### Tinker'de Boris

Eğer sisteminiz destekliyorsa `php artisan tinker` komutu şimdi [Boris REPL](https://github.com/d11wtq/boris) kullanmaktadır. Bu özelliği kullanmak için `readline` ve `pcntl` PHP uzantıları başlatılmış olmalıdır. Bu uzantılara sahip değilseniz, 4.0'daki kabuk kullanılacaktır.

### Eloquent Geliştirmeleri

Eloquent'e yeni bir `hasManyThrough` ilişkisi eklenmiştir. Bunun nasıl kullanılacağını öğrenmek için [Eloquent dokümantasyonuna](/docs/eloquent#has-many-through) bakın.

[Modelleri ilişki sınırlandırmalarına dayalı getirmeye](/docs/eloquent#querying-relations) imkan vermek amacıyla yeni bir `whereHas` metodu kullanıma girmiştir.

### Veritabanı Okuma / Yazma Bağlantıları

Sorgu oluşturucu ve Eloquent de dahil olmak üzere veritabanı katmanı boyunca artık okuma / yazma bağlantılarının otomatik olarak ayrı ayrı ele alınması mümkün bulunmaktadır. Daha fazla bilgi için [dokümantasyonuna](/docs/database#read-write-connections) bakın.

### Kuyruk (Queue) Önceliği

Kuyruk öncelikleri şimdi `queue:listen` komutuna virgülle ayrılmış bir liste geçilmesi şeklinde desteklenmektedir.

### Gerçekleştirilememiş Kuyruk İşinin İşlenmesi

Kuyruk araçları şimdi `queue:listen` üzerinde yeni `--tries` anahtarı kullanılması halinde, başarısız kalmış işlerin otomatik işlenmesini içermektedir. Başarısız kalmış işlerin işlenmesiyle ilgili daha fazla bilgi [kuyruklar dokümantasyonunda](/docs/queues#failed-jobs) bulunabilir.

### Cache Tagları

Cache "section"larının yerini "tag"lar almıştır. Cache tagları bir cache öğesine birden çok "tag" atamanıza ve tek bir tag'a atanmış tüm öğeleri boşaltmanıza (flush) imkan verir. Cache taglarının kullanılması üzerine daha fazla bilgi [cache dokümantasyonunda](/docs/cache#cache-tags) bulunabilir.

### Esnek Şifre Hatırlatıcıları

Şifre hatırlatıcı motoru şifreler geçerlilik denetiminden geçirilirken, session'a durum mesajları flaşlanırken v.b., geliştiriciye daha büyük esneklik sağlayacak şekilde değiştirilmiştir. Gelişmiş şifre hatırlatıcı motorunun kullanımı hakkında daha fazla bilgi için [dokümantasyonuna](/docs/security#password-reminders-and-reset) bakın.

### Gelişmiş Rotalama Motoru

Laravel 4.1 tamamen yeniden yazılmış bir rotalama katmanına sahiptir. API aynıdır; ancak, rotaların kayda geçirilmesi 4.0 ile karşılaştırıldığında tam % 100 daha hızlıdır. Bütün motor büyük ölçüde basitleştirilmiştir ve rota ifadelerinin derlenmesinde Symfony Routing Katmanına bağımlılık en aza indirilmiştir.

### Gelişmiş Session Motoru

Bu yeni sürümde biz aynı zamanda tamamen yeni bir session motorunu da kullanıma sokuyoruz. Rotalama geliştirmelerine benzer şekilde, yeni session katmanı da daha yalın ve daha hızlıdır. Artık Symfony'nin (ve dolayısıyla PHP'nin) session işleme araçlarını kullanmıyoruz ve daha basit ve sürdürülmesi daha kolay olan özel bir çözüm kullanıyoruz.

### Doctrine DBAL

Eğer migrasyonlarınızda `renameColumn` fonksiyonunu kullanıyorsanız, `composer.json` dosyanıza `doctrine/dbal` bağımlılığını eklemeniz gerekecek. Bu paket artık ön tanımlı olarak Laravel'e dahil edilmemektedir.