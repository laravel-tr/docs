# Katkıda Bulunma Kılavuzu

- [Giriş](#introduction)
- [Çekirdek Geliştirme Tartışmaları](#core-development-discussion)
- [Yeni Özellikler](#new-features)
- [Bug'lar](#bugs)
- [Liferaft Uygulamalarının Oluşturulması](#creating-liferaft-applications)
- [Liferaft Uygulamalarının Çekip Alınması](#grabbing-liferaft-applications)
- [Hangi Dal?](#which-branch)
- [Güvenlik Açıkları](#security-vulnerabilities)
- [Kodlama Biçimi](#coding-style)

<a name="introduction"></a>
## Giriş

Laravel açık kaynak bir projedir ve Laravel'in geliştirilmesi için herkes ona katkıda bulunabilir. Beceri düzeyi, cinsiyeti, ırkı, dini ve milliyeti ne olursa olsun katılımcıları bekliyoruz. Farklı, canlı bir topluluğa sahip olmak frameworkün temel değerlerinden biridir!

Aktif işbirliğini teşvik etmek amacıyla, Laravel şu anda bug bildirimlerini değil, sadece çekme isteklerini (pull requests) kabul etmektedir. "Bug bildirimleri" başarısız kalan bir unit testini içeren bir çekme isteği şeklinde gönderilebilir. Alternatif olarak, bir sandbox Laravel uygulaması içindeki bir bug gösterimi [ana Laravel ambarına](https://github.com/laravel/laravel) bir çekme isteği olarak gönderilebilir. Başarısız kalan bir unit testi veya bir sandbox uygulaması, geliştirme ekibine bu bug'ın mevcut olduğunun "kanıtını" gösterir ve geliştirme ekibi bu bug'ı hallettikten sonra da bug'ın düzeltildiğinin güvenilir bir göstergesi olarak hizmet eder.

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

### Unit Test Yoluyla

Bug'lar için çekme istekleri, öncesinde Laravel geliştirme ekibiyle tartışılmaksızın gönderilebilir. Bir bug düzeltmesi gönderirken bug'ın asla tekrar gözükmeyeceğinden bizi emin eden bir unit testi eklemeye çalışınız!

Eğer frameworkte bir bug bulduğunuza inanıyor, fakat onu nasıl düzelteceğinizden emin değilseniz, lütfen başarısız kalan bir unit testi içeren bir çekme isteği gönderin. Başarısız kalan bir unit testi, geliştirme ekibine bu bug'ın mevcut olduğunun "kanıtını" gösterir ve geliştirme ekibi bu bug'ı hallettikten sonra da bug'ın düzeltildiğinin güvenilir bir göstergesi olarak hizmet eder.

Şayet bir bug için başarısız kalan bir unit testinin nasıl yazılacağından emin değilseniz, frameworke dahil edilmiş diğer unit testlerini gözden geçirin. Hala yapamadınızsa, `#laravel` IRC channel (Freenode)'da yardım isteyebilirsiniz.

### Laravel Liferaft Yoluyla

Sorununuz için bir unit testi yazamıyorsanız, Laravel Liferaft size sorunu yeniden oluşturan bir demo uygulaması oluşturma imkanı verir. Liferaft, Laravel ambarının fork edilmesi ve ambara çekme isteklerinin gönderilmesini otomatize de edebilir. Liferaft uygulamanız gönderildikten sonra, bir Laravel geliştiricisi sizin uygulamanızı [Homestead](/docs/homestead) üzerinde çalıştırabilir ve sorununuzu gözden geçirebilir.

<a name="creating-liferaft-applications"></a>
## Liferaft Uygulamalarının Oluşturulması

Laravel Liferaft Laravele katkıda bulunmak için yeni ve yenilikçi bir yol sağlar. İlk olarak Composer aracılığıyla Liferaft CLI aracını yüklemeniz gerekecek:

### Liferaft Yüklenmesi

	composer global require "laravel/liferaft=~1.0"

Terminalinizden `liferaft` komutu çalıştırıldığı zaman `liferaft` çalıştırılabilir dosyasının bulunması için PATH'inizde `~/.composer/vendor/bin` dizini olduğundan emin olun.

### GitHub İle Giriş Yapılması

Liferaft ile çalışmaya başlamadan önce, bir GitHub kişisel erişim tokenine kayıt olmanız gerekir. [GitHub settings panelinizden](https://github.com/settings/applications) bir kişisel erişim tokeni üretebilirsiniz. GitHub tarafından seçilmiş durumdaki default kapsamlar yeterli olacaktır; bununla birlikte, eğer isterseniz, Liferaft'ın sizin eski sandbox uygulamalarınızı silebilmesi için `delete_repo` kapsamı imtiyazını alabilirsiniz.

	liferaft auth my-github-token

### Yeni Bir Liferaft Uygulaması Oluşturulması

Yeni bir Liferaft uygulaması oluşturmak için, `new` komutunu kullanmanız yeterlidir:

	liferaft new my-bug-fix

Bu komut birkaç şey yapacaktır. Birincisi, [Laravel GitHub repository'yi](https://github.com/laravel/laravel) sizin GitHub hesabına fork edecektir. Daha sonra bu fork edilen ambarı sizin makinenize klonlayacak ve Composer bağımlılıklarını yükleyecektir. Ambar yüklendikten sonra, Liferaft uygulaması içerisinde sorunu yeniden oluşturmaya başlayabilirsiniz!

### Sorununuzun Yeniden Oluşturulması

Bir Liferaft uygulaması oluşturulmasından sonra, basitçe sorununuzu yeniden oluşturun. Rotalarınızı tanımlayabilir, Eloquent modelleri oluşturabilir ve hatta veritabanı migrasyonları üretebilirsiniz! Tek gereklilik, uygulamanızın yeni bir [Laravel Homestead](/docs/homestead) sanal makinesi üzerinde çalışabiliyor olmasıdır. Bu, Laravel geliştiricilerine sizin uygulamanızı kendi makinelerinde kolaylıkla çalıştırabilme imkanı verir.

Sorununuzu Liferaft uygulaması içerisinde yeniden oluşturduktan sonra, gözden geçirme için onu tekrar Laravel ambarına geri göndermeye hazırsınız demektir!

### Uygulamanızın Gözden Geçirme İçin Gönderilmesi

Sorununuzu yeniden oluşturduktan sonra, onu gözden geçirme için göndermenin tam zamanıdır! Bununla birlikte, öncelikle Liferaft uygulamanız içerisinde üretilmiş olan `liferaft.md` dosyasını tamamlamanız gerekir. Bu dosyanın ilk satırı çekme isteğinizin başlığı olacaktır. İçeriğin geri kalan kısı çekme isteğinin gövdesinde yer alacaktır. Tabii ki, GitHub Flavored Markdown desteklenmektedir.

Bu `liferaft.md` dosyasını doldurduktan sonra, değişikliklerinizin tamamını GitHub ambarınıza gönderin. Sonra da, uygulamanızın dizininden Liferaft `throw` komutunu çalıştırın:

	liferaft throw

Bu komut Laravel GitHub ambarı için bir çekme isteği oluşturacaktır. Bir Laravel geliştiricisi sizin uygulamanızı kolaylıkla çekip alabilecek ve kendi Homestead ortamlarında çalıştırabilecektir!

<a name="grabbing-liferaft-applications"></a>
## Liferaft Uygulamalarının Çekip Alınması

Laravele katkıda bulanmak mı istiyorsunuz? Liferaft Liferaft uygulamalarını yüklemek ve onları kendi [Homestead ortamınızda](/docs/homestead) görme işini sancısız bir hale getirir.

İlk olarak, kolaylık olması için, [laravel/laravel](https://github.com/laravel/laravel) ambarını kendi makinenizde bir `liferaft` dizinine klonlayın:

	git clone https://github.com/laravel/laravel.git liferaft

Ondan sonra, `develop` dalını yoklayın, böylece hem stabil hem de gelecek Laravel sürümlerini hedef alan Liferaft uygulamalarını yükleyebileceksiniz:

	git checkout -b develop origin/develop

Sonra da, sizin ambar dizininden Liferaft `grab` komutunu çalıştırabilirsiniz. Örneğin, #3000 çekme isteği ile ilişkili Liferaft uygulamasını yüklemek istiyorsanız, aşağıdaki komutu çalıştırmalısınız:

	liferaft grab 3000

Bu `grab` komutu sizin Liferaft dizininizde yeni bir dal oluşturacak ve belirtilen çekme isteği için değişiklikleri çekecektir. Liferaft uygulaması yüklendikten sonra, [Homestead](/docs/homestead) sanal makineniz aracılığıyla dizini hizmete sokmanız (serve etmeniz) yeterlidir! Sorunu debug ettikten sonra, doğru düzeltme ile [laravel/framework](https://github.com/laravel/framework) ambarına bir çekme isteği göndermeyi unutmayın!

Fazladan bir saatiniz var ve rastgele bir sorunu çözmek istiyorsunuz? Bir çekme isteği ID'si olmaksızın, `grab` komutunu tek başına çalıştırın:

	liferaft grab

<a name="which-branch"></a>
## Hangi Dal?

> **Not:** Bu kesim esas olarak [laravel/framework](https://github.com/laravel/framework) ambarına çekme istekleri gönderilmesi için geçerlidir, Liferaft uygulamaları için değildir.

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
