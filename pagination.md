# Sayfalama

- [Yapılandırma](#configuration)
- [Kullanım](#usage)
- [Sayfalama Linklerine Ekleme Yapmak](#appending-to-pagination-links)
- [JSON'a Dönüştürme](#converting-to-json)
- [Özel Sunumcular](#custom-presenters)

<a name="configuration"></a>
## Yapılandırma

Diğer frameworkler'de, sayfalama oldukça sıkıntılı olabilir. Laravel bu işi çocuk oyuncağı gibi yapar. `app/config/view.php` dosyasında bir tek yapılandırma seçeneği bulunmaktadır. Bu dosyadaki `pagination` seçeneği sayfalama bağlantıları (links) oluşturmak için kullanılması gereken görünümü (view) belirtir. Varsayılan olarak, Laravel iki görünüm içerir.

`pagination::slider` görünümü mevcut sayfaya dayalı olarak akıllı bir bağlantı aralığı gösterirken, `pagination::simple` görünümü sadece "önceki" ve "sonraki" butonlarını gösterecektir. **Her iki görünüm de Twitter Bootstrap ile uyumludur**

<a name="usage"></a>
## Kullanım

Öğeleri sayfalamak için çeşitli yollar vardır. En basiti sorgu oluşturucusunda veya bir Eloquent modelinde `paginate` metodunu kullanmaktır.

#### Veritabanı Sonuçlarının Sayfalandırılması

	$uyeler = DB::table('users')->paginate(15);

> **Not:** Şu an için, bir `groupBy` cümlesi kullanan pagination işlemleri Laravel tarafından verimli bir biçimde çalıştırılamamaktadır. Eğer sayfalanmış bir sonuç kümesinde bir `groupBy` kullanmanız gerekiyorsa, veritabanını elle sorgulamanız ve `Paginator::make` kullanmanız önerilir.

#### Bir Eloquent Modelinin Sayfalandırılması

[Eloquent](/docs/eloquent) modellerini de sayfalandırabilirsiniz:

	$uyeler = User::paginate(15);

	$uyeler = User::where('votes', '>', 100)->paginate(15);

`paginate` metoduna geçilen argüman sayfa başına görüntülemek istediğiniz öğelerin sayısıdır. Bir kez sonuçları aldıktan sonra görünümde görüntüleyebilir ve `links` metodunu kullanarak sayfalama bağlantıları oluşturabilirsiniz:

	<div class="container">
		<?php foreach ($uyeler as $uye): ?>
			<?php echo $uye->isim; ?>
		<?php endforeach; ?>
	</div>

	<?php echo $uyeler->links(); ?>

Sayfalama sistemi oluşturmak işte bu kadar! Unutmayın, mevcut sayfa için frameworke bilgi vermedik. Laravel bunu sizin için otomatik olarak belirleyecektir.

Sayfalama için kullanılacak özel bir view belirtmek isterseniz, `links` metoduna bir view geçebilirsiniz:

	<?php echo $uyeler->links('view.ismi'); ?>

Ayrıca aşağıdaki metodlar aracılığıyla diğer sayfalama bilgilerine erişebilirsiniz:

- `getCurrentPage`
- `getLastPage`
- `getPerPage`
- `getTotal`
- `getFrom`
- `getTo`
- `count`


#### "Basit Sayfalandırma"

Eğer sayfalandırma view'inizde sadece "Sonraki" ve "Önceki" linklerini gösteriyorsanız, daha etkin bir sorgulama gerçekleştirmek için `simplePaginate` metodunu kullanma seçeneğine sahipsiniz. Bu, view'inizde tam sayfa numaraları gösterilmesi gerekmediğinde, büyük veri setleri için kullanışlıdır:

	$uyeler = User::where('votes', '>', 100)->simplePaginate(15);

#### Elle Bir Sayfalandırıcı Oluşturmak

Bazen bir sayfalama olgusunu kendiniz bir öğeler dizisi geçerek oluşturmak isteyebilirsiniz. Bunu `Paginator::make` metodunu kullanarak yapabilirsiniz:

	$sayfalandirici = Paginator::make($ogeler, $toplamOgeAdedi, $sayfaBasinaAdet);

#### Sayfalama URI'ını Özelleştirmek

`setBaseUrl` metodu aracılığıyla, sayfalandırıcı tarafından URI'yi de özelleştirebilirsiniz:

	$uyeler = Uye::paginate();

	$uyeler->setBaseUrl('ozel/url');

Yukarıdaki örnek böyle bir URL oluşturacaktır: http://ornek.com/ozel/url?page=2

<a name="appending-to-pagination-links"></a>
## Sayfalama Linklerine Ekleme Yapmak

Sayfalandırıcı üzerinde `appends` metodunu kullanarak sayfalama linklerinize sorgu katarı (query string) ekleyebilirsiniz:

	<?php echo $uyeler->appends(array('sira' => 'oylar'))->links(); ?>

Bu kod, sayfalama linkine "&sira=oylar" ekleyecek ve şöyle bir URL üretecektir:

	http://ornek.com/birsey?page=2&sira=oylar

Eğer sayfalandırıcının URL'sine bir "hash fragmanı" eklemek istiyorsanız, `fragment` metodunu kullanabilirsiniz:

	<?php echo $uyeler->fragment('falan')->links(); ?>

Bu metod bunun gibi gözüken URL'ler üretecektir:

	http://ornek.com/birsey?page=2#falan

<a name="converting-to-json"></a>
## JSON'a Dönüştürme

`Paginator` sınıfı `Illuminate\Support\Contracts\JsonableInterface` sözleşmesini implemente eder ve `toJson` metoduna sahiptir. Bir `Paginator` olgusunu bir rotadan döndürerek de onu JSON'a çevirebilirsiniz. Bu olgunun JSON'lanmış biçimi `total`, `current_page`, `last_page`, `from` ve `to` gibi bazı "meta" bilgilerini de içerecektir. Olgunun verileri JSON dizisindeki `data` anahtarı aracılığı ile erişebilir olacaktır.

<a name="custom-presenters"></a>
## Özel Sunumcular

Laravelle geldiği haliyle ön tanımlı sayfalama sunumcusu Bootstrap uyumludur; ancak siz bunu kendi seçeceğiniz bir sunumcu ile özelleştirebilirsiniz.

### Soyut Sunumcunun Genişletilmesi

`Illuminate\Pagination\Presenter` sınıfını genişletin ve onun soyut (abstract) metodlarını implemente edin. Zurb Foundation için örnek bir sunumcu bunun gibi gözükebilir:

    class ZurbPresenter extends Illuminate\Pagination\Presenter {

        public function getActivePageWrapper($text)
        {
            return '<li class="current"><a href="">'.$text.'</a></li>';
        }

        public function getDisabledTextWrapper($text)
        {
            return '<li class="unavailable">'.$text.'</li>';
        }

        public function getPageLinkWrapper($url, $page)
        {
            return '<li><a href="'.$url.'">'.$page.'</a></li>';
        }

    }

### Özel Sunumcunun Kullanılması

Önce `app/views` dizininizde sizin özel sunumcunuz olarak hizmet edecek bir view oluşturun. Ondan sonra, `app/config/view.php` yapılandırma dosyasındaki `pagination` seçeneğini yeni view'in adıyla değiştirin. Son olarak, özel sunumcu view'inizde aşağıdaki kodu koyacaksınız:

    <ul class="pagination">
        <?php echo with(new ZurbPresenter($paginator))->render(); ?>
    </ul>
