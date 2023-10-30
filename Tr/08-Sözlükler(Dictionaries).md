# Sözlükler (Dictionaries)

Cairo'nun ana kitaplığında sözlük benzeri bir tip sunulmaktadır. `Felt252Dict<T>` veri tipi, her anahtarın benzersiz olduğu ve ilgili bir değerle ilişkilendirildiği **anahtar-değer** çiftlerinin bir koleksiyonunu temsil eder. Bu tür veri yapısı farklı programlama dillerinde haritalar, karma tablolar, ilişkilendirilmiş diziler ve daha birçokları gibi farklı şekillerde bilinir.

`Felt252Dict<T>` tipi, verilerinizi belirli bir şekilde düzenlemek istediğinizde ve bir `Array<T>` kullanıp indeksleme yeterli olmadığında kullanışlıdır. Cairo sözlükleri, hiç değişmez belleğin olmadığı durumlarda değiştirilebilir belleğin varlığını kolayca taklit etmeye de olanak tanır.

## Sözlüklerin Temel Kullanımı ##

Diğer dillerde yeni bir sözlük oluştururken hem anahtarın hem de değerin veri tiplerini tanımlamak normaldir. Ancak Cairo'da, anahtar türü `felt252`'ye sınırlandırılmıştır, böylece sadece değer veri tipini, `Felt252Dict<T>`'de `T `olarak temsil edilen, belirtme olanağı vardır.

`Felt252Dict<T>` 'nin temel işlevselliği, tüm temel işlemleri içeren `Felt252DictTrait` adında bir özellikle gerçekleştirilir. Bu işlemler arasında:

`insert(felt252, T) -> ()`: Bir sözlük örneğine değer yazmak için,
`get(felt252) -> T`: Değerleri ondan okumak için.

Bu fonksiyonlar, bize sözlükleri diğer her dilde olduğu gibi kullanma olanağı tanır. Aşağıdaki örnekte, bireyler ve bakiyeleri arasında bir eşleme temsil eden bir sözlük oluşturuyoruz:
```
use dict::Felt252DictTrait;

fn main() {
    let mut bakiyeler: Felt252Dict<u64> = Default::default();

    bakiyeler.insert('Alex', 100);
    bakiyeler.insert('Maria', 200);

    let alex_bakiyesi = bakiyeler.get('Alex');
    assert(alex_bakiyesi == 100, 'Bakiye 100 değil');

    let maria_bakiyesi = bakiyeler.get('Maria');
    assert(maria_bakiyesi == 200, 'Bakiye 200 değil');
}
```
İlk yaptığımız şey, sözlükle etkileşimde bulunmak için ihtiyaç duyduğumuz tüm yöntemleri kapsama alanına getiren `Felt252DictTrait`'i içe aktarmaktır. Daha sonra `Default` özelliğinin varsayılan yöntemini kullanarak `Felt252Dict<u64>` 'nin yeni bir örneğini oluşturduk ve `insert` yöntemi kullanarak her birine kendi bakiyesiyle iki birey ekledik. Son olarak, `get` yöntemi ile kullanıcılarımızın bakiyesini kontrol ettik.

Bu kitap boyunca, Cairo'da bir hafıza hücresine sadece bir kez yazabilirsiniz, ancak `Felt252Dict<T>` tipi bu engeli aşmanın bir yolunu temsil eder. Sözlükler Altında bölümünde bunun nasıl gerçekleştirildiğini daha sonra açıklayacağız.

## Sözlüklerin Alt Yapısı ##

Cairo'nun belirsiz tasarımının kısıtlamalarından biri, bellek sisteminin değiştirilemez olmasıdır. Bu nedenle, değiştirilebilirliği taklit etmek için dil, `Felt252Dict<T>` 'yi bir giriş listesi olarak uygular. Her giriş, bir sözlüğün okuma/güncelleme/yazma amaçları için erişildiği bir zamanı temsil eder. Bir girişte üç alan vardır:

1- Bu **anahtar-değer** çiftinin değerini tanımlayan bir anahtar alanı.
2- Anahtarda hangi önceki değerin tutulduğunu belirten bir `previous_value` alanı.
3- Anahtarda tutulan yeni değeri belirten bir `new_value` alanı.

Eğer `Felt252Dict<T>` 'yi yüksek seviye yapıları kullanarak uygulamaya çalışırsak, onu `Array<Entry<T>>` olarak tanımlarız ve her `Entry<T>` 'nin ne **anahtar-değer** çiftini temsil ettiği ve sahip olduğu önceki ve yeni değerler hakkında bilgi sahibi oluruz. `Entry<T> `'nin tanımı şöyledir:
```
struct Entry<T> {
    key: felt252,
    previous_value: T,
    new_value: T,
}
```
Her `Felt252Dict<T>` ile etkileşim için yeni bir `Entry<T>` kaydedilecektir.
Eğer `Felt252Dict<T>` 'yi uygulamanın alternatif yollarını düşünürseniz, onları mutlaka bulursunuz, muhtemelen previous_value alanına tamamen ihtiyaç duymadan. Ancak, Cairo normal bir dil olmadığı için bu işe yaramaz. Cairo'nun amacı, STARK kanıt sistemi ile, hesaplama bütünlüğünün kanıtlarını oluşturmaktır. Bu, programın doğru şekilde çalıştığını doğrulamanız ve Cairo'nun sınırlamaları içinde kaldığını doğrulamanız gerektiği anlamına gelir. Bu sınır kontrollerinden biri "sözlük sıkıştırmasıdır" ve bu, her giriş için önceki ve yeni değerler hakkında bilgi gerektirir.

## Sözlükleri Sıkıştırma ##

Bir Cairo programının bir `Felt252Dict<T>` kullandığı yürütme tarafından üretilen kanıtın doğru olduğunu doğrulamak için özel bir şemaya ihtiyacı vardır. Bunu "sözlük sıkıştırma" adlı özel bir süreçle gerçekleştirir.

Sözlük sıkıştırma süreci, `Felt252Dict<T>` 'nin doğru şekilde kullanıldığından emin olmak için tasarlanmıştır. Daha spesifik olarak, bir `Felt252Dict<T>` 'de bir **anahtar-değer** çifti için birden fazla giriş varsa, bu çiftlerin doğru sırayla girdiğini doğrulamaya çalışır. Bu, **STARK** kanıt sistemiyle kullanıldığında gereklidir, çünkü herhangi bir `Felt252Dict<T>` 'de iki **anahtar-değer** çifti arasındaki sıra, bir programın doğru şekilde çalıştığına dair kanıtın doğru olup olmadığını etkileyebilir. Bu nedenle, her `Felt252Dict<T>` 'de birden fazla **anahtar-değer** çifti varsa, bu çiftlerin doğru sırayla sıralandığından emin olmak için sözlük sıkıştırma süreci gereklidir.

Sözlük sıkıştırma süreci, `Felt252Dict<T>` 'de bulunan tüm **anahtar-değer** çiftlerini tarar ve her çift için şunları yapar:

1- Bu anahtar-değer çifti için birden fazla giriş varsa, bu çiftlerin doğru sırayla sıralandığından emin olun.
2- Bu anahtar-değer çifti için birden fazla giriş yoksa, bu çiftin sadece bir kez `Felt252Dict<T> `'de bulunduğundan emin olun.

Bu süreç, `Felt252Dict<T>` 'de bir anahtar için birden fazla değer kaydedildiğinde bile, sözlüğün içeriğinin değiştirilmeden kalmasını sağlar. Bu, STARK kanıt sistemi ile kanıtları doğrularken `Felt252Dict<T>` 'nin içeriğinin tam olarak belirtildiği gibi olduğundan emin olmak için gereklidir.

Bununla birlikte, sözlük sıkıştırma süreci `Felt252Dict<T>` 'nin tam olarak nasıl uygulandığına dair bazı sınırlamalar getirir. Özellikle:

Bir `Felt252Dict<T>` 'de aynı anahtar için birden fazla giriş varsa, bu girişlerin doğru sırayla sıralandığından emin olunmalıdır.
Bir `Felt252Dict<T>` 'de bir anahtar için sadece bir giriş varsa, bu girişin sadece bir kez `Felt252Dict<T>` 'de bulunduğundan emin olunmalıdır.

Bu sınırlamalar, `Felt252Dict<T> `'yi kullanırken dikkate alınmalıdır. Özellikle, bir `Felt252Dict<T>` 'de bir anahtar için birden fazla giriş varsa, bu girişlerin doğru sırayla sıralandığından emin olmak için ekstra dikkat göstermek önemlidir. Bu, özellikle `Felt252Dict<T>` 'nin bir fonksiyonun içinde veya bir dizi işlemin ardından değiştirildiği durumlarda geçerlidir.

Sonuç olarak, `Felt252Dict<T>` tipi, Cairo'da değiştirilebilir belleğin varlığını taklit etmek için kullanılan bir veri yapısıdır. Ancak, bu türün nasıl uygulandığı ve nasıl kullanıldığı hakkında bazı önemli sınırlamalar vardır. Bu sınırlamalar, `Felt252Dict<T>` 'yi kullanırken dikkate alınmalıdır. Ayrıca, `Felt252Dict<T>` 'nin nasıl uygulandığı ve nasıl kullanıldığı hakkında daha fazla bilgi için Cairo'nun belgelerine başvurmanız önerilir.
