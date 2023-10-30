# Fonksiyonlar

Fonksiyonlar, Cairo programlama dilinde yaygın olarak kullanılır. Birçok programın başlangıç noktası olan `main` fonksiyonunu daha önce görmüş olmalısınız. Bir fonksiyonun `fn` anahtar kelimesi ile tanımlandığını ve fonksiyonun temel yapısına önceki bölümlerden aşina olduğunuzu varsayıyoruz.

Cairo'da fonksiyonlar, **snake_case** formatında adlandırılır. İşte bir örnek:
```
use debug::PrintTrait;

fn another_function() {
    'Another function.'.print();
}

fn main() {
    'Hello, world!'.print();
    another_function();
}
```
Yukarıdaki kodda, `another_function` fonksiyonunu `main` fonksiyonundan önce tanımladık; ancak bunu daha sonra da tanımlayabilirdik. Cairo, fonksiyonlarınızın nerede tanımlandığını önemsemez, yeter ki çağrıldıkları yerden erişilebilir olsunlar. 

Bu programı `scarb cairo-run` komutu ile çalıştırdığımızda aşağıdaki çıktıyı alırız:
```
[DEBUG] Hello, world!                (raw: 5735816763073854953388147237921)
[DEBUG] Another function.            (raw: 22265147635379277118623944509513687592494)
```

## Parametreler

Parametreler, fonksiyonun çalışması için gerekli olan değerleri almak için kullanılan değişkenlerdir. Bir fonksiyonun parantez içinde belirtilen parametre listesi, fonksiyonun çağrıldığında bu değerlerin geçirilmesini sağlar. İşte `another_function` fonksiyonuna parametre ekleyerek bir örnek:
```
use debug::PrintTrait;

fn main() {
    another_function(5, 6);
}

fn another_function(x: felt252, y: felt252) {
    x.print();
    y.print();
}
```

Bu örnekte, `another_function` adlı bir fonksiyon oluşturduk ve bu fonksiyona iki parametre ekledik. İlk parametre `x`, ikincisi `y` olarak adlandırıldı ve veri türleri belirtilmediği için varsayılan olarak `felt252` olarak tanımlanacak. Fonksiyon daha sonra `x` ve `y` değerlerini terminale yazdırır.
Kodu çalıştırmak için terminalde `scarb cairo-run` komutunu kullanabilirsiniz.

## Adlandırılmış Parametreler

Adlandırılmış parametreler, bir fonksiyonu çağırırken bağımsız değişkenlerin adlarını belirtmenize olanak sağlar. Adlandırılmış parametreler kullanmak istiyorsanız, parametrenin adını ve iletmek istediğiniz değeri belirtmeniz gerekir. Fonksiyonda belirtilen sırayla aynı sıraya sahip olmaları gerektiğini unutmayın

```
fn foo(x: u8, y: u8) {
    // ...
}

fn main() {
    let first_arg = 3;
    let second_arg = 4;
    // parametre_adı: değeri
    foo(x: first_arg, y: second_arg);
    // foo(y: second_arg, x: first_arg); <- bu şekil bir hata üretecektir
}
```

 ## Statements and Expressions

Fonksiyon gövdeleri bir dizi Statement ile oluşur ve isteğe bağlı olarak bir Expression ile sonlanır. Şu ana kadar ele aldığımız fonksiyonlar sonlanan bir Expression içermemiştir, ancak bir Statement'ın parçası olarak bir Expression görmüşsünüzdür. Cairo, bir Expression tabanlı dildir, bu nedenle bu farklılığı anlamak önemlidir.

**Statements**: Bir eylem gerçekleştiren ve değer döndürmeyen talimatlardır. Örneğin:
```
let y = 6;
```
Bu bir Statement'tır.

**Expressions**: Bir değere çözümlenen kod parçalarıdır. 

Cairo'da, bir Statement (örneğin let y = 6;) değer döndürmez, bu nedenle:
```
let x = (let y = 6);
```
gibi bir kod hata üretir. Ancak Expressions, değer döndürebilir ve bir Statement'ın parçası olabilir:
```
let y = {
    let x = 3;
    x + 1
};
```
Bu Expression, bu durumda 4'e çözümlenir.

Eğer Expression'ın sonuna bir noktalı virgül eklerseniz, onu bir Statement yaparsınız ve artık bir değer döndürmez. Bu, fonksiyon dönüş değerleriyle ilgilenirken dikkate almanız gereken bir konsepttir.

## Return Değerine Sahip Fonksiyonlar

Cairo'da dönüş değerine sahip fonksiyonlar, bir değeri hesaplar ve bu değeri çağıran yere döndürür. Fonksiyonun sonunda `return` ifadesi kullanılarak döndürülecek değer belirtilir. Bu tür fonksiyonlar, hesaplamaları tamamladıktan sonra sonucu döndürmek için kullanılır ve çağıran tarafından bu değer kullanılabilir. Dönüş değerlerini adlandırmıyoruz, ancak türlerini bir oktan (`->`) sonra bildirmeliyiz. Aşağıda bununla ilgili bir örnek verilmiştir:
```
use debug::PrintTrait;

fn main() {
    let x = plus_one(5);

    x.print();
}

fn plus_one(x: u32) -> u32 {
    x + 1
}
```

Bu kodu çalıştırdığınızda `[DEBUG] (raw: 6)` çıktısını alırsınız. Ancak `x + 1` ifadesinin sonuna noktalı virgül eklerseniz, bu bir ifade olmaktan çıkar ve aşağıdaki gibi bir hata alırsınız:
```
error: Unexpected return type. Expected: "core::integer::u32", found: "()".
```
