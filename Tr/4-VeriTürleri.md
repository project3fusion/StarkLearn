# Veri Türleri
Cairo programlama dili, statik bir yapıya sahip olup, derleme sürecinde tüm değişkenlerin türlerinin belirlenmesini gerektirir. Cairo'da veri türleri, belirli gruplara ayrılmıştır.

## Skaler Türler
Skaler tür, tek bir değeri temsil eder. Cairo'da üç ana skaler tür bulunmaktadır: felt, integer ve boolean. Şimdi bu türlerin işleyişlerine bir göz atalım.

### Felt252
Cairo'da, bir değişkenin veya argümanın türü belirtilmediğinde, varsayılan olarak alan elemanı (`felt252`) türü kullanılır. Alan elemanı, `0` ile `P` arasında bir tamsayıdır. Toplama, çıkarma veya çarpma işlemlerinde sonuç P'ye göre mod alınır. Bölme işlemi, standart bölme işleminden farklıdır ve `(x / y) * y == x` denklemi her zaman geçerlidir. Ancak,`y`'nin `x`'i tam olarak bölemediği durumlarda beklenmeyen sonuçlar elde edilebilir. Örneğin, 1 / 2 ifadesi `(P+1)/2` olarak kabul edilir. Alan elemanları, değerlerin belirlenen aralığın dışına çıktığında modüler aritmetik kullanarak otomatik olarak sarmalanma özelliğine sahiptir.

```
fn main() {
    // max value of felt252
    let x: felt252 = 3618502788666131213697322783095070105623107215331596699973092056135872020480;
    let y: felt252 = 1;
    assert(x + y == 0, 'P == 0 (mod P)');
}
```
Felt252 türü, varsayılan veri türü olduğu için, değişkenin türünü belirtmeye gerek yoktur. Tür belirtmeyen değişkenler, otomatik olarak felt252 türü olarak tanımlanır.
```
fn main() {
    // max value of felt252
    let x = 3618502788666131213697322783095070105623107215331596699973092056135872020480;
    let y = 1;
    assert(x + y == 0, 'P == 0 (mod P)');
}
```
### Integer

Felt252 türü, tüm türlerin temelini oluşturan bir tür olsa da, programcıların mümkün olduğunda integer türlerini kullanmaları önerilir. Çünkü integer türleri, kodun potansiyel güvenlik açıklarına karşı ekstra koruma sağlar.

Integer tür bildirimi, programcının tamsayıyı depolamak için kullanabileceği bit sayısını belirtir. Cairo'da yerleşik tamsayı türleri aşağıdaki tabloda gösterilmiştir. Bir tamsayı değerinin türünü belirtmek için bu türlerden herhangi birini kullanabilirsiniz.

Her bir türün açık bir boyutu vardır. Değişkenler işaretsiz olduğu için negatif bir sayı içeremezler. Bu  programın çökmesine neden olur:
```
fn sayi(x: u8, y: u8) -> u8 {
    x - y
}

fn main() {
    sayi(1, 3);
}
```
şu şekil bir hata çıktısı alırsınız
```
Run panicked with [608642109794502019480482122260311927 ('u8_sub Overflow'), ].
```
## Sayısal İşlemler

Cairo, tüm tamsayı türleri için temel matematiksel işlemleri destekler: toplama, çıkarma, çarpma, bölme ve kalan.
```
fn main() {
    // addition
    let sum = 5_u128 + 10_u128;

    // subtraction
    let difference = 95_u128 - 4_u128;

    // multiplication
    let product = 4_u128 * 30_u128;

    // division
    let quotient = 56_u128 / 32_u128; 
    let quotient = 64_u128 / 32_u128;

    // remainder
    let remainder = 43_u128 % 5_u128;-
}
```
Cairo'nun desteklediği tüm operatörlerin listesine aşağıdaki linkten ulaşabilirsiniz.
https://book.cairo-lang.org/appendix-02-operators-and-symbols.html#operators

### Boolean

Cairo'da, diğer birçok programlama dilinde olduğu gibi, Boolean türü iki olası değere sahiptir: `true` ve `false`. Cairo'da Boolean türü `bool` olarak belirtilir.
```
fn main() {
    let t = true;

    let f: bool = false; 
}
```

### String

Cairo'da string türü, `felt252`'de saklanan karakterlerin bir koleksiyonudur. String bir ifade en fazla 31 karakter uzunluğunda olabilir.
```
use debug::PrintTrait;
fn main() {
    let x = 'Cairo is awesome';
    x.print();
    let c = 'A';
    c.print();
}
```
bu kodu çalıştırmak için terminalinize `scarb cairo-run` komutunu girebilirisiniz.

### Tür Dönüşümleri

Cairo'da, bir türü başka bir türe dönüştürmek için `TryInto` ve `Into` özellikleri tarafından sağlanan `try_into` ve `into` yöntemlerini kullanabilirsiniz. `try_into` yöntemi, hedef türün kaynak değere uygun olmayabileceği durumlarda güvenli tür dönüşümüne izin verir. `try_into`, yeni değere erişmek için `Option` türünü döndürür ve bu değeri açmak için `unwrap()` yöntemi kullanılır.
```
use traits::TryInto;
use traits::Into;
use option::OptionTrait;

fn main() {
    let my_felt252 = 10;
    // felt252 bir u8'e sığmayabileceğinden, Option<T> türünü açmanız gerekir
    let my_u8: u8 = my_felt252.try_into().unwrap();
    let my_u16: u16 = my_u8.into();
    let my_u32: u32 = my_u16.into();
    let my_u64: u64 = my_u32.into();
    let my_u128: u128 = my_u64.into();
    // felt252, bir u256'dan daha küçük olduğu için into() yöntemini kullanabiliriz
    let my_u256: u256 = my_felt252.into();
    let my_usize: usize = my_felt252.try_into().unwrap();
    let my_other_felt252: felt252 = my_u8.into();
    let my_third_felt252: felt252 = my_u16.into();
}
```

### Tuple Türü

Bir **tuple**, çeşitli türlere sahip bir dizi değeri tek bir bileşik tipte gruplandırmanın genel bir yoludur. **Tuples **sabit bir uzunluğa sahiptir: bir kez ilan edildikten sonra, boyut olarak büyüyemez veya küçülemezler.

Parantez içine virgülle ayrılmış bir değer listesi yazarak bir tuple oluşturabiliriz. Demetteki farklı değerlerin türlerini farklı olabilir.
```
fn main() {
    // Tuple'daki her konumu ve türünü açıkça bildirebilirsiniz.
    let tup: (u32, u64, bool) = (8, 65, false);
    // Veya türleri belirtmeden de bildirilebilir.
    let tup = (false, 2, 70);

    // Bir tuple tek bir öğe olarak kabul edildiğinden, 
    // tek tek öğeleri almak için kalıp eşleştirmeyi kullanabilirsiniz
    let (x, y, z) = tup; 
    // Tuple'ı bildirirken bunu da yapabilirsiniz.
    let (x, y): (felt252, felt252) = (2, 3);
}
```
İşte kenar uzunluklarını tuple olarak tanımladığımız bir dikdörtgen prizmasının hacmini hesaplayan bir örnek:
```
use debug::PrintTrait;

fn volume(sides: (u64, u64, u64)) -> u64 {
    let (x, y, z) = sides;
    x * y * z
}

fn main() {
    let dikdortgen_prizma = (20, 30, 40); // En x Boy x Derinlik
    let v = volume(dikdortgen_prizma);
    v.print(); 
}
```
Kodu çalıştırmak için terminalde `scarb cairo-run` komutunu kullanabilirsiniz.
