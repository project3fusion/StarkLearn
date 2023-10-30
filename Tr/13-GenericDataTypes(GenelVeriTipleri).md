# Generic Data Types  (Genel Veri Tipleri) #

Genel veri tipleri (generics) kullanarak, yapılar (structs) ve fonksiyonlar gibi öğe tanımları oluştururuz. Cairo'da, fonksiyonlar, yapılar, enumlar, özellikler (traits), uygulamalar (implementations) ve metodlar tanımlanırken genel tipleri kullanabiliriz!

## Genel Fonksiyonlar ##

Generics kullanan bir fonksiyon tanımlarken, genel veri tiplerini fonksiyon imzasında yerleştiririz. Bu, parametrenin ve dönüş değerinin veri tiplerini belirttiğimiz yere benzer. Örneğin, verilen iki "Array" arasında en büyük olanını döndüren bir fonksiyon oluşturmak istediğimizi düşünelim. Bu işlemi farklı türlerdeki listeler için gerçekleştirmemiz gerekiyorsa, fonksiyonu her seferinde yeniden tanımlamamız gerekir. Generics kullanarak bu fonksiyonu bir kez uygulayabilir ve diğer görevlere geçebiliriz.
```
use array::ArrayTrait;
fn largest_list<T, impl TDrop: Drop<T>>(l1: Array<T>, l2: Array<T>) -> Array<T> {
    if l1.len() > l2.len() {
        l1
    } else {
        l2
    }
}

fn main() {
    let mut l1 = ArrayTrait::new();
    let mut l2 = ArrayTrait::new();

    l1.append(1);
    l1.append(2);

    l2.append(3);
    l2.append(4);
    l2.append(5);

    // T'nin somut tipini belirtmeye gerek yok çünkü
    // bu, derleyici tarafından çıkarılır
    let l3 = largest_list(l1, l2);
}
```
`largest_list` fonksiyonu, aynı türde iki listeyi karşılaştırır ve daha fazla öğeye sahip olanı döndürür. 
`largest_list` fonksiyonu, oraya yerleştirilen herhangi bir genel tipin bırakılabilir olması gerektiğini tanımına ekler. Ana fonksiyon değişmez; derleyici hangi somut tipin kullanıldığını ve Bırakma özelliğini (Drop trait) uygulayıp uygulamadığını akıllıca belirleyebilir.

## Genel Veri Tipleri İçin Kısıtlamalar ##

Genel veri tiplerini tanımlarken, bu tipler hakkında bilgi sahibi olmak yararlıdır.  Daha önce, `largest_list`'in genel argümanlarına TDrop uygulamasını ekleyerek bunun bir örneğini gördük. TDrop, derleyici gereksinimlerini karşılamak için eklendi, ancak fonksiyon mantığımızı yararlandırmak için de kısıtlamalar ekleyebiliriz.

Örneğin, T adında bir genel tipin bazı öğelerinin listesi verildiğinde, bunlar arasında en küçük öğeyi bulmak istiyoruz. Başlangıçta, T türündeki bir öğenin karşılaştırılabilir olması için `PartialOrd` özelliğini uygulaması gerektiğini biliyoruz. Sonuç fonksiyon şöyledir:

```
use array::ArrayTrait;

// T listesi verildiğinde en küçük olanını alın.
// PartialOrd özelliği, T için karşılaştırma işlemlerini uygular
fn smallest_element<T, impl TPartialOrd: PartialOrd<T>>(list: @Array<T>) -> T {
    // Bu, iterasyon boyunca en küçük öğeyi temsil eder
    // Burada desnap (*) operatörünü kullanmaya dikkat edin
    let mut smallest = *list[0];
    let mut index = 1;
    loop {
        if index >= list.len() {
            break smallest;
        }
        if *list[index] < smallest {
            smallest = *list[index];
        }
        index = index + 1;
    }
}
```
smallest_element fonksiyonunda, listeye indeksleme yaptığımızda, sonuç olarak indekslenmiş öğenin bir anlık görüntüsünü alıyoruz. Eğer PartialOrd özelliği @T için uygulanmamışsa, öğeyi * operatörü ile desnap yapmamız gerekiyor. * operasyonu, @T'den T'ye bir kopya oluşturur. Bu, T'nin Copy özelliğini uygulaması gerektiği anlamına gelir. Bir @T türünden T türüne bir öğe kopyalandığında, bırakılması gereken T türünde değişkenler vardır. Bu da T'nin Drop özelliğini de uygulaması gerektiği anlamına gelir. Bu iki özellik, fonksiyonun doğru çalışabilmesi için eklenmelidir. smallest_element fonksiyonunu güncelledikten sonra sonuç kod şöyledir:

```
use array::ArrayTrait;

fn smallest_element<T, impl TPartialOrd: PartialOrd<T>, impl TCopy: Copy<T>, impl TDrop: Drop<T>>(
    list: @Array<T>
) -> T {
    let mut smallest = *list[0];
    let mut index = 1;
    loop {
        if index >= list.len() {
            break smallest;
        }
        if *list[index] < smallest {
            smallest = *list[index];
        }
        index = index + 1;
    }
}
```
Bu fonksiyon, PartialOrd, Copy ve Drop özelliklerini uygulayan genel bir T tipini kullanır, bir Array<T>'nin bir anlık görüntüsünü parametre olarak alır ve en küçük öğenin bir kopyasını döndürür.

## Yapılar (Structs) ##

Fonksiyon tanımlarına benzer şekilde, bir veya daha fazla alan için genel tip parametresini kullanarak yapıları tanımlayabiliriz. Aşağıdaki kod örneği, `T` tipinde bir `balance` alanına sahip olan `Wallet<T>` tanımını göstermektedir.
```
#[derive(Drop)]
struct Wallet<T> {
    balance: T
}

fn main() {
    let w = Wallet { balance: 3 };
}
```

Yukarıdaki kod, `Wallet` tipi için `Drop `özelliğini otomatik olarak türetir. Bu, aşağıdaki kodu yazmakla eşdeğerdir:
```
struct Wallet<T> {
    balance: T
}

impl WalletDrop<T, impl TDrop: Drop<T>> for Drop<Wallet<T>>;

fn main() {
    let w = Wallet { balance: 3 };
}
```
Burda temel olarak, `Wallet<T>` yapısının `T` de bırakılabilir olduğu sürece bırakılabilir olduğunu söylüyoruz.

Wallet'a bir adres alanı eklemek istiyorsak ve bu alanın `T`'den farklı ama aynı zamanda genel olmasını istiyorsak, sadece köşeli parantezler arasına başka bir genel tip ekleyebiliriz:
```
#[derive(Drop)]
struct Wallet<T, U> {
    balance: T,
    address: U,
}

fn main() {
    let w = Wallet { balance: 3, address: 14 };
}
```
`Wallet` yapı tanımına yeni bir genel tip olan `U` ekledik ve bu tipi yeni alan üyesi olan `addresse` atadık. `Drop` özelliği için `derive` özelliğinin `U` için de çalıştığını unutmayın.

##  Enumlar ##

Yapılarla (structs) yaptığımız gibi, enumları da varyantlarında genel veri türlerini tutacak şekilde tanımlayabiliriz. Örneğin, Cairo çekirdek kütüphanesi tarafından sağlanan `Option<T>`  enumu:
```
enum Option<T> {
    Some(T),
    None,
}
```
`Option<T>` enumu, `T` tipi üzerinde geneldir ve iki varyanta sahiptir: `Some`, `T` tipinde bir değeri tutar ve `None` herhangi bir değer tutmaz. `Option<T>` enumunu kullanarak, bir değerin isteğe bağlı olma kavramını soyut bir şekilde ifade edebiliriz ve değerin genel bir `T` tipine sahip olması sayesinde bu soyutlamayı herhangi bir tip ile kullanabiliriz.

Enumlar, çekirdek kütüphanenin sağladığı `Result<T, E>` enumunun tanımı gibi, birden fazla genel tipi de kullanabilir:
```
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```
`Result<T, E>` enumu, `T` ve `E` olmak üzere iki genel tipe sahiptir ve iki varyanta sahiptir: `Ok`, `T` tipinde bir değeri tutar ve `Err` `E` tipinde bir değeri tutar. Bu tanım, bir işlemin başarılı olabileceği (T tipinde bir değer dönerek) veya başarısız olabileceği (E tipinde bir değer dönerek) her yeri için `Result` enumunu kullanmayı çok uygun hale getirir.

