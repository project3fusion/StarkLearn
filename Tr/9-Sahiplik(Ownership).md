# Sahiplik (Ownership) #

Sahiplik, Cairo programlama dilinde bellek yönetimi ve kaynak güvenliği için kullanılan bir kavramdır. Her değerin yalnızca bir sahibi olabilir ve sahip, değeri kullanırken onu değiştirebilir, başka bir değişkene aktarabilir veya bellekten serbest bırakabilir. Sahiplik kuralı, bellek sızıntılarını önler ve geçersiz bellek erişimlerini engeller. Cairo, sahiplik kuralını derleme zamanında kontrol eder ve hatalı sahiplik işlemlerini tespit eder. Bu sayede güvenli ve hızlı bir dil olmayı sağlar.

## Sahiplik Kuralları ##

Cairo'da 3 temel sahiplik kuralı vardır:

1- Cairo'da her değerin yalnızca bir sahibi vardır.
2- Aynı anda yalnızca bir sahip olabilir.
3- Sahip kapsam dışına çıktığında değerini kaybeder.

##  Değişken Kapsamı ##

Kapsam, bir program içinde bir öğenin geçerli olduğu aralığı belirtir. Örnek olarak aşağıdaki değişkeni ele alalım:
```
let s = 'hello';
```

Bu durumda, `s` değişkeni bildirildiği noktadan kapsamın sonuna kadar geçerlidir. Örneğin:
```
// s burada geçerli değildir, henüz beyan edilmemiştir
{                      
        let s = 'hello';   // s bu noktadan itibaren geçerlidir

        
    } // bu kapsam artık sona ermiştir ve s artık geçerli değildir
```

## Array ile Sahiplik ##

Derleme zamanında boyutu bilinmeyen ve kopyalanamayan Array türünün nasıl davrandığını inceleyelim. Aşağıdaki kodu ele alalım:
```
use array::ArrayTrait;
fn foo(arr: Array<u128>) {}

fn bar(arr: Array<u128>) {}

fn main() {
    let mut arr = ArrayTrait::<u128>::new();
    foo(arr);
    bar(arr);
}
```
Bu durumda, `foo` ve `bar` fonksiyonlarına aynı dizi örneğini geçirmeye çalışıyoruz. Bu, her iki fonksiyon çağrısında kullanılan parametrenin aynı dizi örneği olduğu anlamına gelir. Eğer `foo` fonksiyonuna bir değer ekler ve ardından `bar` fonksiyonuna aynı diziyle başka bir değer eklemeye çalışırsanız, aynı bellek hücresine iki kez yazmaya çalışırsınız ve bu mülkiyet kurallarını ihlal eder ve hatalara neden olur.

Yukarıdaki kodu çalıştırmayı denerseniz aşağıdaki gibi bir derleme zamanı hatası alırsınız:
```
error: Variable was previously moved. Trait has no implementation in context: core::traits::Copy::<core::array::Array::<core::integer::u128>>
 --> array.cairo:6:9
    let mut arr = ArrayTrait::<u128>::new();
        ^*****^
```
## Copy ve Clone ##

Cairo'nun sahiplik kuralları nedeniyle, verilerin nasıl çoğaltılabileceğini belirtmek gerekir. Bunun için `Clone` ve `Copy` özellikleri kullanılır. Clone, bir tür örneğinin nasıl klonlanabileceğini, yani verilerinin çoğaltılacağını ve yeni bir sahip atanabileceğini belirtir. Clone tarafından belirtilen davranış Clone tarafından karmaşık olabilir. Bunun yerine Copy, Clone gerektirir ve bir türün bit düzeyinde kopyalanabileceğini belirtir.

Örnek olarak aşağıdaki kodu ele alalım:
```
use array::ArrayTrait;
use clone::Clone;
use array::ArrayTCloneImpl;

fn foo(arr: Array<u128>) {
    // foo dizinin sahipliğini alır.
    // bu fonksiyon return edildiğinde arr düşer.
}

fn main() {
    // arr'ın yaratıcısı olarak, ana fonksiyon dizinin sahibidir
    let arr = ArrayTrait::<u128>::new();

    foo(arr.clone()); // arr'ın bir klonunun sahipliğini fonksiyon çağrısına taşır

    foo(arr); // derlenir çünkü daha önce dizi çoğaltılmıştır
    
    // foo(arr); <- main artık diziye sahip olmadığı için derleme başarısız olur
}
```

Kopyalama özelliği ile değer türetilmesine bir örnek:
```
use clone::Clone;

#[derive(Copy, Clone, Drop)]
struct Vector2 {
    x: u32,
    y: u32,
}
// u32 derives Copy too

fn main() {
    let v = Vector2 { x: 1, y: 0 };
    let w = v;

    // Şimdi 'w', 'v'nin bir kopyasıdır, 'v' hala erişilebilirdir
}
```
Örnekleri, terminalinizde `scarb cairo-run --available-gas 20000` komutunu kullanarak çalıştırabilirsiniz.

## Destruct Özelliği ##

Sözlükler **Drop** özelliğini uygulamasa da, Destruct özelliğini uygularlar ve bu, kapsamdan çıktıklarında otomatik olarak sıkıştırılmalarına izin verir. Bu, sözlüğü manuel olarak `squash` yöntemini çağırmadan kullanabileceğiniz anlamına gelir.

Aşağıdaki örneği düşünün, burada bir sözlük içeren özel bir tip tanımlıyoruz:

```
use dict::Felt252DictTrait;

struct A {
    dict: Felt252Dict<u128>
}

fn main() {
    A { dict: Felt252DictTrait::new() };
}
```

Bu kodu çalıştırmaya çalışırsanız, derleme zamanı bir hata alırsınız:
```
hata: Değişken bırakılmadı. Bağlamda özellik uygulaması yok: core::traits::Drop::<temp7::temp7::A>. Bağlamda özellik uygulaması yok: core::traits::Destruct::<temp7::temp7::A>.
 --> temp7.cairo:7:5
    A {
    ^*^
```

A kapsamdan çıktığında bırakılamaz çünkü ne Drop özelliğini uygular (bir sözlük içerdiği için derive(Drop) alamaz) ne de Destruct özelliğini. Bunu düzeltmek için, A tipi için Destruct özelliği uygulamasını türetebiliriz:
```
use dict::Felt252DictTrait;
use traits::Default;

#[derive(Destruct)]
struct A {
    dict: Felt252Dict<u128>
}

fn main() {
    A { dict: Default::default() }; // Burada hata yok
}
```
Şimdi, A kapsamdan çıktığında, sözlüğü otomatik olarak sıkıştırılacak ve program derlenecektir.

## Sahiplik ve Fonksiyonlar ##

Bir değişkeni bir fonksiyona geçirmek, değişkeni taşır veya kopyalar. `Array` bölümünde görüldüğü gibi, bir `Array` fonksiyon parametresi olarak geçirildiğinde sahipliği aktarır. Diğer türlerde ne olacağını görelim. Örnek olarak aşağıdaki kodu ele alalım:
```
#[derive(Drop)]
struct MyStruct{}

fn main() {
    let my_struct = MyStruct{};  // my_struct kapsama girer

    takes_ownership(my_struct);     // my_struct'ın değeri fonksiyona taşınır 
                                    // ve bu nedenle artık burada geçerli değildir.

    let x = 5;                 // x kapsama girer

    makes_copy(x);                  // x fonksiyonun içine doğru hareket edecektir,
                                    // ancak u128 Copy'yi uygular, 
                                    // bu nedenle daha sonra x'i kullanın

}                                   // Burada x kapsam dışına çıkar ve atılır.


fn takes_ownership(some_struct: MyStruct) { // some_struct kapsama girer
} // Burada, some_struct kapsam dışına çıkar ve `drop` çağrılır.

fn makes_copy(some_uinteger: u128) { // some_uinteger kapsama girer
} // Burada, some_integer kapsam dışına çıkar ve bırakılır.
```
Eğer `my_struct`'ı `takes_ownership` çağrısından sonra kullanmaya çalışsaydık, Cairo derleme zamanı hatası verirdi. Bu statik kontroller bizi hatalardan korur. Bunları nerede kullanabileceğinizi ve sahiplik kurallarının bunu yapmanızı nerede engellediğini görmek için `main`'e `my_struct` ve `x` kullanan kod eklemeyi deneyin.
