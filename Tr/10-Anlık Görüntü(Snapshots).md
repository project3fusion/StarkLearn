# Anlık Görüntü(Snapshots) #

Cairo'daki anlık görüntü, bir değerin belirli bir zaman dilimindeki sabit görünümünü ifade eder. Önceki bölümde, Cairo'nun sahiplik sistemini ve bir değeri nasıl taşıdıktan sonra kullanmamızı engellediğini, ayrıca bir diziye değer eklerken aynı bellek hücresine iki kez yazma riskini nasıl azalttığını ele almıştık. Ancak, bu her zaman kullanışlı olmayabilir. Anlık görüntülerin yardımıyla, çağıran işlevin değer sahipliğini nasıl koruyabileceğimizi inceleyelim.

Bir dizinin anlık görüntüsünü parametre olarak alan bir `calculate_length` fonksiyonu tanımlamayı düşünelim. Bu örnekte, `calculate_length` fonksiyonu, parametre olarak verilen dizinin uzunluğunu döndürür. Diziyi, değişmez bir görünüm olan anlık görüntü olarak aktardığımız için, `calculate_length` fonksiyonunun diziyi değiştirmeyeceğini ve dizinin sahipliğinin ana fonksiyonda kalacağını biliyoruz.
```
use array::ArrayTrait;
use debug::PrintTrait;

fn main() {
    let mut arr1 = ArrayTrait::<u128>::new();
    let first_snapshot = @arr1; // 'arr1' dizisinin anlık görüntüsünü alın
    arr1.append(1); // 'arr1' dizisine bir değer ekleyin
    let first_length = calculate_length(
        first_snapshot
    ); // Anlık görüntü alındığında dizinin uzunluğunu hesaplayın
    let second_length = calculate_length(@arr1); // Anlık görüntü alındığında dizinin uzunluğunu hesaplayın
    first_length.print();
    second_length.print();
}

fn calculate_length(arr: @Array<u128>) -> usize {
    arr.len()
}
```
Programı çalıştırdığımızda aşağıdaki çıktıyı alırız:
```
[DEBUG]                                       (raw: 0)

[DEBUG]                                      (raw: 1)

Run completed successfully, returning []
```
`arr1` sözdizimi, `arr1` dizisinin anlık görüntüsünü oluşturmamızı sağlar. Anlık görüntü, bir değerin değişmez bir görünümü olduğundan, işaret ettiği değer anlık görüntü aracılığıyla değiştirilemez ve anlık görüntü kullanılmayı bıraktığında işaret ettiği değer düşmez.
Fonksiyonun imzasında `arr` parametresinin türünün bir anlık görüntü olduğunu belirtmek için `@` kullanılır.
```
fn calculate_length(
    array_snapshot: @Array<u128>
) -> usize { // 'array_snapshot' bir dizinin anlık görüntüsüdür
    array_snapshot.len()
} // Burada, 'array_snapshot' kapsam dışına çıkar ve bırakılır. 
//Ancak, bu sadece orijinal 'arr' dizisinin içerdiklerinin bir görünümü olduğundan,
// orijinal 'arr' hala kullanılabilir.
```
`array_snapshot` değişkeninin geçerli olduğu kapsam, herhangi bir fonksiyon parametresinin kapsamı ile aynıdır, ancak `array_snapshot` kullanılmayı bıraktığında anlık görüntünün temel değeri düşmez. Fonksiyonlar parametre olarak gerçek değerler yerine anlık görüntülere sahip olduğunda, orijinal değerin sahipliğini geri vermek için değerleri döndürmemiz gerekmez, çünkü ona hiç sahip olmadık.

Anlık görüntüler, değer türü kopyalanabilir olduğu sürece desnap operatörü `*` kullanılarak normal değerlere geri dönüştürülebilir. Aşağıdaki örnekte, bir dikdörtgenin alanını hesaplamak istiyoruz, ancak `calculate_area` fonksiyonunda dikdörtgenin sahipliğini almak istemiyoruz, çünkü fonksiyon çağrısından sonra dikdörtgeni tekrar kullanmak isteyebiliriz. Fonksiyonumuz dikdörtgen örneğini değiştirmediğinden, dikdörtgenin anlık görüntüsünü fonksiyona aktarabilir ve ardından desnap operatörünü `*` kullanarak anlık görüntüleri tekrar değerlere dönüştürebiliriz.

```
use debug::PrintTrait;

#[derive(Copy, Drop)]
struct Rectangle {
    height: u64,
    width: u64,
}

fn main() {
    let rec = Rectangle { height: 3, width: 10 };
    let area = calculate_area(@rec);
    area.print();
}

fn calculate_area(rec: @Rectangle) -> u64 {
    // rec' bir dikdörtgenin anlık görüntüsü olduğundan, alanları da alan türlerinin anlık görüntüleridir.
    // Anlık görüntüleri '*' desnap operatörünü kullanarak tekrar değerlere dönüştürmemiz gerekir.
    // Bu yalnızca türün kopyalanabilir olması durumunda mümkündür; u64 için durum böyledir.
    // Burada '*' hem yükseklik ve genişliği çarpmak hem de anlık görüntülerin haritasını çıkarmak için kullanılır.
    *rec.height * *rec.width
}
```
## Değiştirilebilir Referanslar ##

Not: Cairo'da, bir parametre ancak değişken mut ile bildirilmişse ref değiştiricisi kullanılarak mutable referans olarak tanımlanabilir.

`flip` fonksiyonunda `Rectangle `örneğinin yükseklik ve genişlik alanlarının değerini değiştirmek için değiştirilebilir bir referans kullanırız.
```
use debug::PrintTrait;
#[derive(Copy, Drop)]
struct Rectangle {
    height: u64,
    width: u64,
}

fn main() {
    let mut rec = Rectangle { height: 3, width: 10 };
    flip(ref rec);
    rec.height.print();
    rec.width.print();
}

fn flip(ref rec: Rectangle) {
    let temp = rec.height;
    rec.height = rec.width;
    rec.width = temp;
}
```
İlk olarak `rec` değişkenini `mut` olarak değiştiriyoruz. Ardından `rec` değişkeninin mutable referansını `ref rec` ile `flipe` aktarıyoruz ve fonksiyon imzasını `ref rec: Rectangle` şeklinde mutable referansı kabul edecek şekilde güncelliyoruz. Bu, `flip` fonksiyonunun parametre olarak aktarılan `Rectangle` örneğinin değerini değiştireceğini çok açık hale getirir.

Programın aşağıdaki gibi bir çıktı verir:
```
[DEBUG]                         (raw: 10)

[DEBUG]                            (raw: 3)
```
