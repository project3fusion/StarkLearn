# Değişkenler (Variables ve Mutability)

Cairo'da değişken tanımladığınızda, bu değişkeni değiştiremezsiniz çünkü Cairo'da değişkenler, diğer dillerin aksine değiştirilemez olarak tanımlanır. Basit bir değişkeni değiştirip terminale yazdırmak için variables adında bir scarb projesi başlatalım  ve aşağıdaki kodu kullanalım:
```
use debug::PrintTrait;
fn main() {
    let x = 15;
    x.print();
    x = 20;
    x.print();
}
```
Bu kodu `scarb cairo-run` komutuyla çalıştırdığımızda aşağıdaki hatayı alırız:
```
error: Cannot assign to an immutable variable.
 --> variables.cairo:5:5
    x = 20;
    ^****^
​
Error: failed to compile: variables.cairo
```
Bu hata, `x` değerinin değiştirilemez (immutable) olduğunu ve ikinci bir değer atayamayacağınızı gösterir. Cairo'da değişkenleri değiştirilebilir (mutable) olarak tanımlamak için değişkenin soluna `mut` anahtar kelimesini ekleyebilirsiniz. Aşağıdaki kodu kullanarak bu değişikliği yapalım:
```
use debug::PrintTrait;
fn main() {
    let mut x = 15;
    x.print();
    x = 20;
    x.print();
}
```
Bu şekilde değişkenin değerini değiştirebilir ve istediğimiz sonucu elde edebiliriz.

## Constants
Sabitler de değişkenler gibi değiştirilemez, ancak değişkenlerin aksine sabitlerde "mut" anahtar kelimesi kullanılamaz ve `let` yerine `const `ile tanımlanırlar. İşte bir sabit değişken örneği:
```
const ONE_HOUR_IN_SECONDS: u32 = 21000;
```
Cairo'da sabitler, büyük harflerle ve iki kelime arasına alt çizgi koyularak isimlendirilir.

## Shadowing
Gölgeleme, aynı isimle tanımlanan iki değişkenin durumunu ifade eder; burada son tanımlanan değişken, ilk tanımlananı gölgede bırakır. Bu durumu aşağıdaki örnekle açıklayalım:
```
use debug::PrintTrait;
fn main() {
    let x = 5;
    let x = x + 1;
    {
        let x = x * 2;
        
        x.print()
    }
    
    x.print();
}
```
Bu kodda, ilk olarak `x` değişkenine `5` değeri atanır. Ardından, `let x = x + 1` ifadesiyle, orijinal `x` değerini alıp `1` ekleyerek yeni bir `x` değişkeni oluşturulur ve bu yeni `x` değeri `6` olur. Daha sonra, süslü parantezlerle oluşturulan iç kapsamda, `let x = x * 2` ifadesiyle bir kez daha `x` değişkeni gölgelenir ve önceki değeri `2` ile çarparak yeni bir `x` değişkeni oluşturulur ve bu yeni `x` değeri `12` olur. Bu iç kapsam sona erdiğinde, iç gölgelendirme sona erer ve `x` değeri tekrar `6` olur.

Gölgeleme durumunda `mut` anahtar kelimesi kullanılmaz. Çünkü `mut `anahtar kelimesi, aynı değişkene farklı bir değer atanabilmesini sağlar. Ancak gölgeleme durumunda, `let` anahtar kelimesiyle aynı isimli yeni bir değişken oluşturulur ve önceki değişken gölgede kalır.
Daha önce de belirtildiği gibi, değişken gölgeleme ve değiştirilebilir değişkenler alt seviyede eşdeğerdir. Bir değişkeni gölgelediğinizde, türünü değiştirdiğinizde derleyicinin şikayet etmez. Örneğin, programımızın `u64` ve `felt252` tipleri arasında bir tip dönüşümü gerçekleştirdiğini varsayalım.
```
use debug::PrintTrait;
use traits::Into;
fn main() {
    let x = 2;
    x.print();
    let x: felt252 = x.into(); 
    x.print()
}
}
```
İlk `x` değişkeni `u64` tipine sahipken, ikinci `x` değişkeni `felt252` tipine sahiptir. Böylece gölgeleme bizi `x_u64` ve `x_felt252` gibi farklı isimler bulmaktan kurtarır; bunun yerine daha basit olan `x `ismini tekrar kullanabiliriz.

Ancak, bunun için `mut` kullanmaya çalışırsak, burada gösterildiği gibi, bir derleme zamanı hatası alırız:
```
use debug::PrintTrait;
use traits::Into;
fn main() {
    let mut x: u32 = 2;
    x.print();
    x = 100_felt252;
    x.print()
}
```
Hata, bir u64 (orijinal tür) beklediğimizi ancak farklı bir türe sahip olduğumuzu söylüyor:
```
$ scarb cairo-run
error: Unexpected argument type. Expected: "core::integer::u32", found: "core::felt252".
 --> lib.cairo:9:9
    x = 100_felt252;
        ^*********^

Error: failed to compile: src/lib.cairo
```
