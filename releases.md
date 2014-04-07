# Sürüm Notları

- [Laravel 4.1](#laravel-4.1)

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