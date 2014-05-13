# Artisan CLI

- [Giriş](#introduction)
- [Kullanım](#usage)

<a name="introduction"></a>
## Giriş

Artisan, Laravel içerisinde gelen CLI'ın (Command-line Interface) adıdır. Artisan size uygulamanızı geliştirirken birçok yardımcı komut sağlar. Artisan, güçlü Symfony Console bileşeni üzerinden geliştirilmiştir.

<a name="usage"></a>
## Kullanım

#### Tüm Kullanılabilir Komutların Listelenmesi

Tüm Artisan komutlarının bir listesini görmek için `list` komutunu kullanabilirsiniz:

	php artisan list

#### Bir Komut için Yardım Ekranının Görüntülenmesi

Tüm komutların özel bir "yardım" ekranı vardır ve komut hakkındaki argüman sırası ile ayarlar gibi bilgilerin açıklanmasını sağlar. Bir yardım ekranını görüntülemek için komut adından önce `help` yazın:

	php artisan help migrate

#### Yapılandırma Ortamının Belirtilmesi

`--env` anahtarını kullanarak bir komut çalıştırılırken kullanılacak olan yapılandırma ortamını belirtebilirsiniz:

	php artisan migrate --env=local

#### Güncel Laravel Sürümünüzün Gösterilmesi

Ayrıca Laravel yüklemenizin güncel sürümünü de `--version` seçeneğini kullanarak görebilirsiniz:

	php artisan --version