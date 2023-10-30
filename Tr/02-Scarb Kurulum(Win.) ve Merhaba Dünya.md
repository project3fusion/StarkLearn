# Scarb Kurulum(Win.) ve Merhaba Dünya #

Cairo'nun başarılı bir şekilde kurulumunu tamamladık. Şimdi, scarb ile programlama dünyasının geleneksel ilk adımı olan Merhaba Dunya kodunu terminalde yazdırma zamanı.
Scarb, Cargo'nun (Rust'ın yapı sistemi ve paket yöneticisi) ilham aldığı Cairo'nun paket yöneticisidir.

'Hello, world!' projesini Scarb ile derlersek, yalnızca kodu derleyen Scarb bölümü kullanılır, çünkü program herhangi bir harici bağımlılık gerektirmez.

Scarb ile Yeni Bir Proje Oluşturma:
Projeler dizinine geri dönün. Sonra şunu çalıştırın:
```rs
scarb new hello_scarb
```
Bu, hello_scarb adında yeni bir dizin ve proje oluşturur. Scarb, aynı adlı bir dizinde dosyalar oluşturur.

`hello_scarb` dizinine gitmek için `cd hello_scarb `komutunu kullanın. Scarb'ın `Scarb.toml` adında bir dosya ve içerisinde `lib.cairo` adında bir dosya bulunan bir `src` dizini oluşturduğunu göreceksiniz.

Bu dosya, Scarb'ın yapılandırma formatı olan TOML formatındadır. İlk satır, `[package]`, bir paketi yapılandırdığınızı belirten bir bölüm başlığıdır.

Scarb'ın oluşturduğu diğer dosya `src/lib.cairo`'dur. Tüm içeriğini silin ve sebebini sonra açıklayacağımız şu içeriği ekleyin:

```rs
mod hello_scarb;
```
Daha sonra `src/hello_scarb.cairo` adında yeni bir dosya oluşturun ve içine şu kodu ekleyin:

Filename: `src/hello_scarb.cairo`

```rs
use debug::PrintTrait;
fn main() {
'Merhaba Dunya Scarb'.print();
}
```
Scarb, kaynak dosyalarınızın **src **dizini içinde olmasını gerektirir.

### Scarb Projesini Oluşturma: ###
**hello_scarb** dizininden projenizi şu komutla derleyin:
```rs
scarb cairo-run
```

Cairo'yu doğru bir şekilde yüklediyseniz, şu çıktıyı görmelisiniz:

```rs
[DEBUG] Hello, Scarb!
```

## Cairo Programının Anatomisi ##
Bu Merhaba dunya programını detaylı bir şekilde inceleyelim.
```rs
fn main() {
​
}
```

Bu satırlar `main `adında bir fonksiyon tanımlar. Main fonksiyonu Cairo programında her zaman ilk olarak çalışan kod parçasıdır.
Fonksiyon gövdesi` { }` içine alınmıştır. Cairo, tüm fonksiyon gövdeleri süslü parantezlerle çevrilidir.
Main fonksiyon öncesinde, `use debug::PrintTrait;` kullanarak başka bir modüldeki öğeyi içe aktararak kullanabiliriz. Bunu yaparak kodu terminale yazdırmayı sağlayan `print()` yöntemini kullanabiliriz. 

## Özel Scriptleri Tanımlama: ##
**Scarb.toml** dosyasında Scarb scriptleri tanımlayabiliriz. Scarb.toml dosyanıza şu satırı ekleyin:

```rs
[scripts]
lib = "cairo-run --single-file src/lib.cairo"
```

Projeyi çalıştırmak için şu komutu kullanabilirsiniz:
```rs
scarb run run-lib
```
