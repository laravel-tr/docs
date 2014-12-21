# Laravel Homestead

- [Giriş](#introduction)
- [Dahil Edilen Yazılımlar](#included-software)
- [Yükleme & Kurulum](#installation-and-setup)
- [Günlük Kullanım](#daily-usage)
- [Portlar](#ports)

<a name="introduction"></a>
## Giriş

Laravel sizin lokal geliştirme ortamınız da dahil olmak üzere bütün PHP geliştirme deneyimini zevkli bir hale getirmeye çalışmaktadır. [Vagrant](http://vagrantup.com) Sanal Makinelerin yönetilmesi ve hazırlanması için basit, zekice bir yol sağlamaktadır.

Laravel Homestead lokal makinenizde PHP, HHVM, bir web sunucusu ve diğer herhangi bir sunucu yazılımı yüklemenizi gerektirmeksizin size harika bir geliştirme ortamı sağlayan resmi, ambalajlanmış bir Vagrant "box"tur. İşletim sisteminizi karışmasını daha artık dert etmeyin! Vagrant box'ları tamamen kontrol altındadır. Eğer bir şeyler yanlış giderse, onu yok edebilir ve dakikalar içerisinde yeniden oluşturabilirsiniz!

Homestead herhangi bir Windows, Mac veya Linux sisteminde çalışır ve Nginx web sunucusu, PHP 5.6, MySQL, Postgres, Redis, Memcached ve muhteşem Laravel uygulamaları geliştirmek için gerekli diğer tüm güzellikleri içerir.

> **Not:** Eğer Windows kullanıyorsanız, donanım sanallaştırmasını (hardware virtualization) (VT-x) etkinleştirmeniz gerekebilir. Bu genellikle BIOS'iniz aracılığıyla etkinleştirilebilmektedir.

Homestead hali hazırda Vagrant 1.6 kullanılarak inşa ve test edilmiştir.

<a name="included-software"></a>
## Dahil Edilen Yazılımlar

- Ubuntu 14.04
- PHP 5.6
- HHVM
- Nginx
- MySQL
- Postgres
- Node (Bower, Grunt ve Gulp ile)
- Redis
- Memcached
- Beanstalkd
- [Laravel Envoy](/docs/ssh#envoy-task-runner)
- Fabric + HipChat Uzantısı

<a name="installation-and-setup"></a>
## Yükleme & Kurulum

### VirtualBox & Vagrant Yüklemesi

Homestead ortamınızı başlatabilmek için, [VirtualBox](https://www.virtualbox.org/wiki/Downloads) ve [Vagrant](http://www.vagrantup.com/downloads.html) yüklemelisiniz. Bu yazılım paketlerinin her ikisi de tüm popüler işletim sistemleri için kullanımı kolay görsel yükleyiciler sağlar.

### Vagrant Box Eklenmesi

VirtualBox ve Vagrant yüklendikten sonra, terminalinizde aşağıdaki komutu kullanarak Vagrant yüklemenize `laravel/homestead` box'ını eklemelisiniz. Bu kutunun indirilmesi, internet bağlantı hızınıza bağlı olarak birkaç dakika alacaktır:

	vagrant box add laravel/homestead

### Homestead Yüklenmesi

#### Composer + PHP Aracı İle

Kutuyu Vagrant yüklemenize ekledikten sonra, Composer `global` komutunu kullanarak Homestead CLI aracını kurmaya geçebilirsiniz:

	composer global require "laravel/homestead=~2.0"

Terminalinizde `homestead` komutunu çalıştırdığınız zaman `homestead` çalıştırılabilir dosyasının bulunası için PATH'inizde `~/.composer/vendor/bin` dizininin yer aldığından emin olun.

Homestead CLI aracını kurduktan sonra , `Homestead.yaml` konfigürasyon dosyasını oluşturmak için `init` komutunu çalıştırın:

	homestead init

Bu `Homestead.yaml` dosyası `~/.homestead` dizinine koyulacaktır. Eğer bir Mac veya Linux sistemi kullanıyorsanız, `Homestead.yaml` dosyanızı terminalinizde `homestead edit` komutunu çalıştırmak suretiyle düzenleyebilirsiniz:

	homestead edit

#### Elle Git Aracılığıyla (Local PHP Olmadan)

Alternatif olarak, eğer lokal makinenize PHP kurmak istemiyorsanız, elle basitçe ambarı klonlaak suretiyle Homestead kurabilirsiniz. Homestead box sizin Laravel (ve PHP) projelerinizin tümü için barındırma yeri olarak hizmet edeceği için, bu ambarı Laravel projelerinizin tümünü tutacağınız merkezi bir `Homestead` dizinine klonlamayı düşünün:

	git clone https://github.com/laravel/homestead.git Homestead

Once you have installed the Homestead CLI aracını kurduktan sonra, `Homestead.yaml` yapılandırma dosyasını oluşturmak için `bash init.sh` komutunu çalıştırın:

	bash init.sh

Bu `Homestead.yaml` dosyası `~/.homestead` dizinine koyulacaktır.

### SSH Anahtarınızı Ayarlayın

Ondan sonra da, `Homestead.yaml` dosyasını düzenlemelisiniz. Bu dosyada, public SSH anahtarınızın, bunun yanı sıra ana makineniz ile Homestead sanal makineniz arasında paylaşılmasını istediğiniz klasörlerin yolunu ayarlayabilirsiniz.

Bir SSH anahtarınız yok mu? Mac ve Linux'te, genel olarak aşağıdaki komutu kullanarak bir SSH anahtar çifti oluşturabilirsiniz:

	ssh-keygen -t rsa -C "your@homestead"

Windows'ta, [Git](http://git-scm.com/) yükleyebilir ve yukarıdaki komutu vermek için Git'le birlikte bulunan `Git Bash` kabuğunu kullanabilirsiniz. Alternatif olarak, [PuTTY](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html) ve [PuTTYgen](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html) kullanabilirsiniz.

Bir SSH anahtarı oluşturduktan sonra, `Homestead.yaml` dosyanızın `authorize` özelliğinde anahtarın yolunu belirtin.

### Paylaşılan Klasörlerinizi Yapılandırın

`Homestead.yaml` dosyanızın `folders` özelliği Homestead ortamınızda paylaşmak istediğiniz klasörlerin tümünü listeler. Bu klasörler içindeki dosyalar değiştikçe, bunlar lokal makineniz ile Homestead ortamı arasında senkronize tutulacaktır. Ne kadar gerekiyorsa o kadar paylaşılan klasör yapılandırabilirsiniz!

### Nginx Sitelerinizi Yapılandırın

Nginx size tanıdık değil mi? Problem değil. `sites` özelliği, Homestead ortamınızdaki bir klasöre kolaylıkla bir "domain" eşleştirmenize imkan verir. Örnek bir site yapılandırması `Homestead.yaml` dosyasına dahil edilmiştir. Aynı şekilde, Homestead ortamınıza gerektiği kadar çok sayıda site ekleyebilirsiniz. Homestead, üzerinde çalışmakta olduğunuz her Laravel projesi için kullanışlı, sanallaştırılmış bir ortam olarak hizmet edebilir!

`hhvm` opsiyonunu `true` ayarlamak suretiyle herhangi bir Homestead sitesini [HHVM](http://hhvm.com) kullanır hale getirebilirsiniz:

	sites:
	    - map: homestead.app
	      to: /home/vagrant/Code/Laravel/public
	      hhvm: true

### Bash Alias'ları

Homestead kutunuza Bash aliasları eklemek için, basitçe `~/.homestead` dizininin köküne `aliases` dosyası ekleyin.

### Vagrant Box'ı Başlatın

`Homestead.yaml` dosyasını istediğiniz gibi düzenledikten sonra, terminalinizde `homestead up` komutunu çalıştırın. Eğer Homestead'ı elle yüklemiş ve PHP `homestead` aracını kullanmıyorsanız, klonlanmış Homestead Git ambarınızı taşıyan dizinden `vagrant up` komutunu çalıştırın.

Vagrant sanal makineyi boot edecektir ve paylaşılan klasörlerinizi ve Nginx sitelerinizi otomatik olarak yapılandıracaktır! Makineyi yok etmek için, `homestead destroy` komutunu kullanabilirsiniz. Mevcut Homestead komutlarının tam bir listesini görmek için, `homestead list` çalıştırın.

Nginx siteleriniz için "domain"leri makinenizdeki `hosts` dosyasına eklemeyi unutmayın! Bu `hosts` dosyası local domain'lerinize gelen istekleri Homestead ortamınıza yönlendirecektir. Mac ve Linux'te, bu dosya `/etc/hosts` konumundadır. Windows'ta, `C:\Windows\System32\drivers\etc\hosts` konumundadır. Bu dosyaya eklediğiniz satırlar aşağıdaki gibi gözükecektir:

	192.168.10.10  homestead.app

IP adresinin `Homestead.yaml` dosyanızda ayarlanmış bir IP adresi olduğundan emin olun. Domain'i `hosts` dosyanıza ekledikten sonra, siteye tarayıcınız aracılığıyla erişebilirsiniz!

	http://homestead.app

Veritabanlarınıza nasıl bağlanacağınızı öğrenmek için, okumaya devam edin!

<a name="daily-usage"></a>
## Günlük Kullanım

### SSH Aracılığıyla Bağlanma

Homestead ortamınıza SSH aracılığıyla bağlanmak için, terminalinizde `homestead ssh` komutunu veriniz.

### Veritabanlarınıza Bağlanma

Laravel'in geliş halinde hem MySQL hem de Postgres için bir `homestead` veritabanı yapılandırılmıştır. Hatta daha fazla kolaylık için Laravel'in `local` "database" yapılandırma dosyası ön tanımlı olarak bu veritabanını kullanacak şekilde ayarlanmıştır.

Navicat veya Sequel Pro aracılığıyla ana makinenizdeki MySQL veya Postgres veritabanlarınıza bağlanmak için, `127.0.0.1` ve 33060 (MySQL) veya 54320 (Postgres) portuna bağlanmalısınız. Her iki veritabanı için username ve password `homestead` / `secret`'dir.

> **Not:** Standart dışı bu portları sadece ana makinenizdeki veritabanlarına bağlanırken kullanmalısınız. Laravel, Sanal Makine _içerisinde_ çalıştığı için, Laravel veritabanı yapılandırma dosyanızda ön tanımlı 3306 ve 5432 portlarını kullanacaksınız.

### İlave Sitelerin Eklenmesi

Homestead ortamınızı hazırladıktan ve çalıştırdıktan sonra, Laravel uygulamalarınız için başka Nginx siteleri eklemek isteyebilirsiniz. Tek bir Homestead ortamında istediğiniz kadar çok sayıda Laravel yüklemesi çalıştırabilirsiniz. Bunu yapmanın iki yolu vardır. İlk olarak, basitçe bu siteleri `Homestead.yaml` dosyanıza ekleyebilir ve ondan sonra da `vagrant provision` çalıştırabilirsiniz.

Alternatif olarak, Homestead ortamınızda bulunan `serve` skriptini kullanabilirsiniz. Bu `serve` skriptini kullanmak için, Homestead ortamınıza SSH ile girin ve aşağıdaki komutu çalıştırın:

	serve domain.app /home/vagrant/Code/path/to/public/directory

> **Not:** `serve` komutunu çalıştırdıktan sonra, ana makinenizdeki `hosts` dosyanıza yeni siteyi eklemeyi unutmayın!

<a name="ports"></a>
## Portlar

Aşağıdaki portlar Homestead ortamınıza yönlendirilir:

- **SSH:** 2222 -> 22'ye yönlendirilir
- **HTTP:** 8000 -> 80'e yönlendirilir
- **MySQL:** 33060 -> 3306'ya yönlendirilir
- **Postgres:** 54320 -> 5432'ye yönlendirilir
