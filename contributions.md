# Katkıda Bulunma Kılavuzu

- [Giriş](#introduction)
- [Çekirdek Geliştirme Tartışmaları](#core-development-discussion)
- [Yeni Özellikler](#new-features)
- [Bug'lar](#bugs)
- [Hangi Dal?](#which-branch)
- [Güvenlik Açıkları](#security-vulnerabilities)
- [Kodlama Biçimi](#coding-style)

<a name="introduction"></a>
## Giriş

Laravel açık kaynak bir projedir ve Laravel'in geliştirilmesi için herkes ona katkıda bulunabilir. Beceri düzeyi, cinsiyeti, ırkı, dini ve milliyeti ne olursa olsun katılımcıları bekliyoruz. Farklı, canlı bir topluluğa sahip olmak frameworkün temel değerlerinden biridir!

Aktif işbirliğini teşvik etmek amacıyla, Laravel şu anda bug bildirimlerini değil, sadece çekme isteklerini (pull requests) kabul etmektedir. **"Bug bildirimleri" başarısız kalan bir unit testini içeren bir çekme isteği şeklinde gönderilebilir.** Başarısız kalan bir unit testi, geliştirme ekibine bu bug'ın mevcut olduğunun "kanıtını" gösterir ve geliştirme ekibi bu bug'ı hallettikten sonra da bug'ın düzeltildiğinin güvenilir bir göstergesi olarak hizmet eder.

Laravel kaynak kodu Github'da yönetilmektedir ve Laravel projelerinin her biri için ambarlar vardır:

- [Laravel Framework](https://github.com/laravel/framework)
- [Laravel Application](https://github.com/laravel/laravel)
- [Laravel Documentation](https://github.com/laravel/docs)
- [Laravel Cashier](https://github.com/laravel/cashier)
- [Laravel Envoy](https://github.com/laravel/envoy)
- [Laravel Homestead](https://github.com/laravel/homestead)
- [Laravel Homestead Build Scripts](https://github.com/laravel/settler)
- [Laravel Website](https://github.com/laravel/website)
- [Laravel Art](https://github.com/laravel/art)

<a name="core-development-discussion"></a>
## Çekirdek Geliştirme Tartışmaları

Buglar, yeni özellikler ve mevcut özelliklerin uygulanmasıyla ilgili tartışmalar `#laravel-dev` IRC channel'da (Freenode) gerçekleşmektedir. Laravel'in geliştiricisi Taylor Otwell tipik olarak hafta içinde 8am-5pm (UTC-06:00 or America/Chicago) saatleri arasında bu kanalda bulunmaktadır ve diğer saatlerde zaman zaman kanalda olmaktadır.

`#laravel-dev` IRC channel herkese açıktır. İster katılımcı olarak ister sadece tartışmaları izlemek için olsun herkesi kanala bekliyoruz!

<a name="new-features"></a>
## Yeni Özellikler

Yeni özellikler için çekme istekleri göndermeden önce, lütfen `#laravel-dev` IRC channel (Freenode) aracılığıyla Taylor Otwell ile temasa geçiniz. Özellik, framework için oldukça uygun bulunursa, bir çekme isteği yapabilirsiniz. Eğer özellik red edilirse, vaz geçmeyin! Özelliğinizi bir paket haline getirebilirsiniz ve [Packagist](https://packagist.org/) aracılığıyla dünyaya açabilirsiniz.

Yeni özellikler eklerken unit testlerini eklemeyi unutmayın! Unit testleri, yeni özellikler eklendikçe frameworkün stabilite ve güvenilirliğinden emin olmamıza yardım eder.

<a name="bugs"></a>
## Bug'lar

Bug'lar için çekme istekleri, öncesinde Laravel geliştirme ekibiyle tartışılmaksızın gönderilebilir. Bir bug düzeltmesi gönderirken bug'ın asla tekrar gözükmeyeceğinden bizi emin eden bir unit testi eklemeye çalışınız!

Eğer frameworkte bir bug bulduğunuza inanıyor, fakat onu nasıl düzelteceğinizden emin değilseniz, lütfen başarısız kalan bir unit testi içeren bir çekme isteği gönderin. Başarısız kalan bir unit testi, geliştirme ekibine bu bug'ın mevcut olduğunun "kanıtını" gösterir ve geliştirme ekibi bu bug'ı hallettikten sonra da bug'ın düzeltildiğinin güvenilir bir göstergesi olarak hizmet eder.

Şayet bir bug için başarısız kalan bir unit testinin nasıl yazılacağından emin değilseniz, frameworke dahil edilmiş diğer unit testlerini gözden geçirin. Hala yapamadınızsa, `#laravel` IRC channel (Freenode)'da yardım isteyebilirsiniz.

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
