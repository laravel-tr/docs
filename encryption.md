# Şifreleme

- [Önsöz](#introduction)
- [Temel Kullanım](#basic-usage)

<a name="introduction"></a>
## Önsöz

Laravel Mcrypt PHP eklentisi aracılığı ile güçlü AES şifreleme olanakları sağlar.

<a name="basic-usage"></a>
## Temel Kullanım

#### Bir Değer Şifreleme

	$encrypted = Crypt::encrypt('secret');

> **Not:** `config/app.php` dosyasındaki `key` alanına 16, 24 ya da 32 karakterli rastgele bir dize değerininin tanımlanmış olduğundan emin olun. Aksi durumda, şifreli değerleriniz güvenli olmayacaktır.

#### Bir Değerin Deşifresi

	$decrypted = Crypt::decrypt($encryptedValue);

#### Şifre Anahtarı & Mod Ayarı

Ayrıca şifreleyicinin kullandığı şifre anahtarını ve modununu değiştirebilirsiniz:

	Crypt::setMode('ctr');

	Crypt::setCipher($cipher);
