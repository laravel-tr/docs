# Sayfalama

- [Yapılandırma](#configuration)
- [Kullanım](#usage)
- [Sayfalama Linklerine Ekleme Yapmak](#appending-to-pagination-links)
- [Converting To JSON](#converting-to-json)
- [Custom Presenters](#custom-presenters)

<a name="configuration"></a>
## Yapılandırma

Diğer frameworkler'de, sayfalama oldukça sıkıntılı olabilir. Laravel bu işi çocuk oyuncağı gibi yapar. `app/config/view.php` dosyasında bir tek yapılandırma seçeneği bulunmaktadır. `pagination` seçeneği sayfalama bağlantıları (links) oluşturmak için kullanılması gereken görünümü (view) belirtir. Varsayılan olarak, Laravel iki görünüm içerir.

`pagination::slider` görünümü mevcut sayfaya dayalı olarak akıllı bir bağlantı aralığı gösterirken, `pagination::simple` görünümü sadece "önceki" ve "sonraki" butonlarını gösterecektir. **Her iki görünüm de Twitter Bootstrap ile uyumludur**

<a name="usage"></a>
## Kullanım

Öğeleri sayfalamak için çeşitli yollar vardır. En basiti sorgu oluşturucusunda veya bir Eloquent modelinde `paginate` metodunu kullanmaktır.

#### Veritabanı Sonuçlarının Sayfalandırılması

	$uyeler = DB::table('uyeler')->paginate(15);

[Eloquent](/docs/eloquent) modellerini de sayfalandırabilirsiniz:

#### Bir Eloquent Modelinin Sayfalandırılması

	$uyeler = User::paginate(15);

	$uyeler = User::where('oylar', '>', 100)->paginate(15);

`paginate` metodundan geçen argüman sayfa başı görüntülemek istediğiniz öğelerin sayısıdır. Bir kez sonuçları aldıktan sonra görünümde görüntüleyebilir ve `links` metodunu kullanarak sayfalama bağlantıları oluşturabilirsiniz:

	<div class="container">
		<?php foreach ($uyeler as $uye): ?>
			<?php echo $uye->isim; ?>
		<?php endforeach; ?>
	</div>

	<?php echo $uyeler->links(); ?>

Sayfalama sistemi oluşturmak işte bu kadar! Unutmayın, mevcut sayfa için çatıya bilgi vermedik. Laravel bunu sizin için otomatik olarak belirledi.

If you would like to specify a custom view to use for pagination, you may pass a view to the `links` method:

	<?php echo $users->links('view.name'); ?>

Ayrıca aşağıdaki metodlarla ek olarak sayfalama bilgisine erişebilirsiniz:

- `getCurrentPage`
- `getLastPage`
- `getPerPage`
- `getTotal`
- `getFrom`
- `getTo`
- `count`

Bazen bir sayfalama olgusunu kendiniz bir öğeler dizisi geçerek oluşturmak isteyebilirsiniz. Bunu yapmak için `Paginator::make` methodunu kullanınız:

#### Elle Sayfalandırıcı Oluşturmak

	$sayfalandirici = Paginator::make($ogeler, $toplamOgeler, $sayfaBasi);

#### Sayfalama URI'ını Özelleştirmek

Sayfalama tarafından kullanılan `setBaseUrl` methodunu da özelleştirebilirsiniz:

	$uyeler = Uye::paginate();

	$uyeler->setBaseUrl('ozel/url');

Yukarıdaki örnek böyle bir URL oluşturacaktır: http://ornek.com/ozel/url?page=2

<a name="appending-to-pagination-links"></a>
## Sayfalama Linklerine Ekleme Yapmak

Sayfalandırıcı üzerinde `appends` methodunu kullanarak sayfalama linklerinize sorgu katarı (query string) ekleyebilirsiniz:

	<?php echo $uyeler->appends(array('sira' => 'oylar'))->links(); ?>

Bu kod, sayfalama linkine "&sira=oylar" ekleyecek ve şöyle bir URL üretecektir:

	http://ornek.com/birsey?sayfa=2&sira=oylar

If you wish to append a "hash fragment" to the paginator's URLs, you may use the `fragment` method:

	<?php echo $users->fragment('foo')->links(); ?>

This method call will generate URLs that look something like this:

	http://example.com/something?page=2#foo

<a name="converting-to-json"></a>
## Converting To JSON

The `Paginator` class implements the `Illuminate\Support\Contracts\JsonableInterface` contract and exposes the `toJson` method. You can may also convert a `Paginator` instance to JSON by returning it from a route. The JSON'd form of the instance will include some "meta" information such as `total`, `current_page`, `last_page`, `from`, and `to`. The instance's data will be available via the `data` key in the JSON array.

<a name="custom-presenters"></a>
## Custom Presenters

The default pagination presenter is Bootstrap compatible out of the box; however, you may customize this with a presenter of your choice.

### Extending The Abstract Presenter

Extend the `Illuminate\Pagination\Presenter` class and implement its abstract methods. An example presenter for Zurb Foundation might look like this:

    class ZurbPresenter extends Illuminate\Pagination\Presenter {

        public function getActivePageWrapper($text)
        {
            return '<li class="current">'.$text.'</li>';
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

### Using The Custom Presenter

First, create a view in your `app/views` directory that will server as your custom presenter. Then, replace `pagination` option in the `app/config/view.php` configuration file with the new view's name. Finally, the following code would be placed in your custom presenter view:

    <ul class="pagination">
        <?php echo with(new ZurbPrensenter($paginator))->render(); ?>
    </ul>