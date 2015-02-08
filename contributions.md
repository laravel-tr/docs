# Katkıda Bulunma Kılavuzu

- [Bug Bildirimi](#bug-reports)
- [Çekirdek Geliştirme Tartışmaları](#core-development-discussion)
- [Hangi Dal?](#which-branch)
- [Güvenlik Açıkları](#security-vulnerabilities)
- [Kodlama Biçimi](#coding-style)

<a name="bug-reports"></a>
## Hata Bildirimi

Aktif işbirliğini teşvik etmek amacıyla, Laravel şu anda bug bildirimlerini değil, sadece çekme isteklerini (pull requests) kabul etmektedir. "Bug bildirimleri" başarısız kalan bir unit testini içeren bir çekme isteği şeklinde gönderilebilir.

Eğer bir hata bildirimi hazırlarsan, senin sorunun bir başlık ve net bir açıklama içermelidir. Ayrıca mümkün olduğunca ilgili bilgileri ve sorunu gösteren bir kod örneğini içermelidir. Hata bildiriminin amacı kendini - ve diğerleri - hata çoğaltmak ve düzeltme geliştirmek için kolay hale getirmektirtir.

Unutma, hata bildirimleri diğerleri ile aynı problemi çözme konusunda işbirliği yapmayı mümkün kılacak şekilde oluşturulur. Hata bildirimi otomatik olarak herhangi bir faaliyet görmeyi veya başkalarının onu düzeltmeye hemen başlayacğını bekleme. Hata raporu oluşturma size ve başkalarına sorunu çözme konusunda yardım sağlayacaktır.

Laravel kaynak kodu Github'da yönetilmektedir ve Laravel projelerinin her biri için ambarlar vardır:

- [Laravel Framework](https://github.com/laravel/framework)
- [Laravel Application](https://github.com/laravel/laravel)
- [Laravel Documentation](https://github.com/laravel/docs)
- [Laravel Cashier](https://github.com/laravel/cashier)
- [Laravel Envoy](https://github.com/laravel/envoy)
- [Laravel Homestead](https://github.com/laravel/homestead)
- [Laravel Homestead Build Scripts](https://github.com/laravel/settler)
- [Laravel Website](https://github.com/laravel/laravel.com)
- [Laravel Art](https://github.com/laravel/art)

<a name="core-development-discussion"></a>
## Çekirdek Geliştirme Tartışmaları

Buglar, yeni özellikler ve mevcut özelliklerin uygulanmasıyla ilgili tartışmalar `#laravel-dev` IRC channel'da (Freenode) gerçekleşmektedir. Laravel'in geliştiricisi Taylor Otwell tipik olarak hafta içinde 8am-5pm (UTC-06:00 or America/Chicago) saatleri arasında bu kanalda bulunmaktadır ve diğer saatlerde zaman zaman kanalda olmaktadır.

`#laravel-dev` IRC channel herkese açıktır. İster katılımcı olarak ister sadece tartışmaları izlemek için olsun herkesi kanala bekliyoruz!

<a name="which-branch"></a>
## Hangi Dal?

**Tüm** bug düzeltmeleri en son kararlı dala gönderilmelidir. Bug düzeltmeleri, düzeltilen özellikler sadece çıkacak sürümde mevcut olmadığı sürece **asla** `master` dalına gönderilmemelidir.

Güncel Laravel sürümü ile **geriye dönük tam uyumlu** **Minor** özellikler en son kararlı dala gönderilebilir.

**Major** yeni özellikler her zaman için `master` dalına gönderilmelidir; bu dal, çıkacak Laravel sürümünü içerir.

Sizin özelliğin major mu minör mü olduğundan emin değilseniz lütfen `#laravel-dev` IRC channel (Freenode)'da Taylor Otwell'e sorun.

<a name="security-vulnerabilities"></a>
## Güvenlik Açıkları

Laravelde bir güvenlik açığı keşfederseniz, lütfen <a href="mailto:taylorotwell@gmail.com">taylorotwell@gmail.com</a> adresinden Taylor Otwell'e bir e-posta gönderin. Tüm güvenlik açıkları derhal ele alınacaktır.

<a name="coding-style"></a>
## Kodlama Biçimi

Laravel [PSR-0](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-0.md) ve [PSR-1](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-1-basic-coding-standard.md) kodlama standartlarını takip eder. Bu standartlara ek olarak, aşağıdaki kodlama standartlarına uyulmalıdır:

- Sınıf aduzayı beyanları `<?php` ile aynı satırda olmalıdır.
- Bir sınıfın açılışı `{` sınıf ismi ile aynı satırda olmalıdır.
- Fonksiyon ve kontrol yapıları Allman biçimi parantezler kullanmalıdır.
- Sekmelerle girintileyin, boşluklarla hizalayın.
