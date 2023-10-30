# Enum #

Enumlar, varyant adı verilen belirli bir veri türünü tanımlamak için kullanılır. Her bir varyant, farklı ve belirli bir anlamı olan değerler koleksiyonunu temsil eder. İşte basit bir enum örneği:
```
#[derive(Drop)]
enum Direction {
    North: (),
    East: (),
    South: (),
    West: (),
}```

Bu örnekte, `Direction` adında dört varyantlı bir enum tanımladık: `North`, `East`, `South` ve `West`. Her varyant, Yön tipinin farklı bir değerini temsil eder ve bir birim tipi `()` ile ilişkilendirilir. Bir değişkeni örneklendirmek için bu sözdizimi kullanabiliriz:
```
let direction = Direction::North(());```

## Enumlar Ve Özel Veri Türleri ##

Enumlar, her bir varyantla ilişkili daha karmaşık verileri saklamak için de kullanılabilir. İşte bir örnek:
```
#[derive(Drop)]
enum Message {
    Quit: (),
    Echo: felt252,
    Move: (u128, u128),
}```

Bu örnekte, `Message` enumunun üç çeşidi vardır: `Quit`, `Echo` ve `Move`, hepsi farklı türlere sahiptir. `Quit` birim tipidir - kendisiyle ilişkili hiçbir veri yoktur. `Echo` `felt252` tipindedir. `Move`, iki `u128` değerinden oluşan bir tuple'dır.

## Enumlar İçin Trait Uygulamaları ##

Cairo'da, özel enum'larınız için özellikler tanımlayabilir ve bunları uygulayabilirsiniz. Bu, enum ile ilişkili yöntemleri ve davranışları tanımlamanıza olanak tanır. İşte bir önceki `Message` enumu için bir özellik tanımlama ve uygulama örneği:
```
trait Processing {
    fn process(self: Message);
}

impl ProcessingImpl of Processing {
    fn process(self: Message) {
        match self {
            Message::Quit(()) => {
                'quitting'.print();
            },
            Message::Echo(value) => {
                value.print();
            },
            Message::Move((x, y)) => {
                'moving'.print();
            },
        }
    }
}```

Bu örnekte, `Message` için `Processing` özelliğini uyguladık. İşte bir `Quit` mesajını işlemek için nasıl kullanılabileceğine dair bir örnek:
```
enum Option<T> {
    Some: T,
    None: (),
}```

`Option` enumu, bir değerin bulunmama olasılığını açıkça göstermenize olanak tanıyarak kodunuzu daha anlamlı ve hakkında daha kolay mantık yürütülebilir hale getirdiği için faydalıdır. `Option` kullanımı ayrıca, başlatılmamış veya beklenmedik `null` değerlerin kullanılmasından kaynaklanan hataların önlenmesine de yardımcı olabilir.

Aşağıda, verilen değere sahip bir dizinin ilk elemanının indeksini veya eleman yoksa `None` değerini döndüren bir fonksiyon örneği bulunmaktadır.

```
use array::ArrayTrait;
use debug::PrintTrait;
use option::OptionTrait;
fn find_value_recursive(arr: @Array<felt252>, value: felt252, index: usize) -> Option<usize> {
    if index >= arr.len() {
        return Option::None(());
    }

    if *arr.at(index) == value {
        return Option::Some(index);
    }

    find_value_recursive(arr, value, index + 1)
}

fn find_value_iterative(arr: @Array<felt252>, value: felt252) -> Option<usize> {
    let length = arr.len();
    let mut index = 0;
    let mut found: Option<usize> = Option::None(());
    loop {
        if index < length {
            if *arr.at(index) == value {
                found = Option::Some(index);
                break;
            }
        } else {
            break;
        }
        index += 1;
    };
    return found;
}

#[test]
#[available_gas(999999)]
fn test_increase_amount() {
    let mut my_array = ArrayTrait::new();
    my_array.append(3);
    my_array.append(7);
    my_array.append(2);
    my_array.append(5);

    let value_to_find = 7;
    let result = find_value_recursive(@my_array, value_to_find, 0);
    let result_i = find_value_iterative(@my_array, value_to_find);

    match result {
        Option::Some(index) => {
            if index == 1 {
                'it worked'.print();
            }
        },
        Option::None(()) => {
            'not found'.print();
        },
    }
    match result_i {
        Option::Some(index) => {
            if index == 1 {
                'it worked'.print();
            }
        },
        Option::None(()) => {
            'not found'.print();
        },
    }
}```

Bu örneği anlamamanız çok normal bir sonraki bölümde match leri öğrendiğinizde daha iyi anlayacaksınız.
# Match #

Cairo, **match **adında güçlü bir yapı sunar. Bu yapı, bir değeri farklı kalıplarla karşılaştırmanızı ve eşleşen kalıba göre farklı kodlar çalıştırmanızı sağlar.

Match yapısını madeni paralarla ilgili bir örnekle gösterelim. Bir ABD madeni parasını alan ve hangi madeni para olduğunu belirleyip sent cinsinden değerini veren bir fonksiyon yazalım. Madeni paraları temsil etmek için `Coin` adında bir **enum **kullanalım.
```
enum Coin {
    Penny: (),
    Nickel: (),
    Dime: (),
    Quarter: (),
}

fn value_in_cents(coin: Coin) -> felt252 {
    match coin {
        Coin::Penny(_) => 1,
        Coin::Nickel(_) => 5,
        Coin::Dime(_) => 10,
        Coin::Quarter(_) => 25,
    }
}```

`value_in_cents` fonksiyonunun nasıl çalıştığını açıklayalım. Fonksiyona bir `coin` parametresi veriyoruz. Bu parametre, ilk satırda tanımladığımız `Coin` **enum**’unun bir varyantıdır.

Sonra `match` anahtar kelimesini ve **coin **ifadesini yazıyoruz. Bu ifade, karşılaştırılacak değerdir.

Ardından match kollarını yazıyoruz. Bir match kolunun iki bölümü vardır: bir kalıp ve bir kod bloğu. İlk match kolunda `Coin::Penny(_)` kalıbını ve `=>` operatörünü görüyoruz. Bu operatör, kalıbın ve kod bloğunun sınırını belirtir. Bu durumda kod bloğu sadece `1` değeridir. Her **match **kolunu virgülle ayırıyoruz.

Cairo’da, match kollarının sırasının enum ile aynı olması gerekmez.

Match ifadesinin değeri, eşleşen match kolunun kod bloğunun değeridir.


Bir match kolunda birden fazla satır kod yazmak istiyorsak, küme parantezleri kullanabiliriz. Örneğin, aşağıdaki kod, fonksiyon her `Coin::Penny(())` ile çağrıldığında “**Lucky penny!**” yazdırır, ancak yine de **1** değerini döndürür: 
```
fn value_in_cents(coin: Coin) -> felt252 {
    match coin {
        Coin::Penny(_) => {
            ('Lucky penny!').print();
            1
        },
        Coin::Nickel(_) => 5,
        Coin::Dime(_) => 10,
        Coin::Quarter(_) => 25,
    }
}```

## Match İle Option ##

**Option**’ı da match ile kullanabiliriz! Option’ın varyantlarını madeni paralar gibi karşılaştıracağız, ancak mantık aynı kalacaktır. Option’ı `option::OptionTrait` özelliğini içe aktararak kullanabiliriz.

Bir Option alan ve içindeki değere `1` ekleyen bir fonksiyon yazalım. Eğer Option içinde bir değer yoksa, fonksiyon `None` değerini döndürsün ve hiçbir işlem yapmasın.
```
use option::OptionTrait;
use debug::PrintTrait;

fn plus_one(x: Option<u8>) -> Option<u8> {
    match x {
        Option::Some(val) => Option::Some(val + 1),
        Option::None(_) => Option::None(()),
    }
}

fn main() {
    let five: Option<u8> = Option::Some(5);
    let six: Option<u8> = plus_one(five);
    six.unwrap().print();
    let none = plus_one(Option::None(()));
    none.unwrap().print();
}```

Option’ın varyantları şöyle tanımlanmıştır:
```
enum Option<T> {
    Some: T,
    None: (),
}```

`plus_one` fonksiyonunun ilk çağrısını inceleyelim. `plus_one(five)` dediğimizde, fonksiyonun içindeki `x` değişkeni `Some(5)` değerine sahip olur. Sonra bunu match kollarıyla karşılaştırırız:
```
Option::Some(val) => Option::Some(val + 1),```

`Option::Some(val)` kalıbı `Some(5)` değeriyle eşleşir. Bu durumda, `val` değişkeni `5` değerine bağlanır. Kod bloğu, `val + 1 `değerini `Option::Some` ile sarmalar ve `Option::Some(6)` değerini döndürür.
```
Option::None(_) => Option::None(()),```

Bu match koluna bakmaya gerek yoktur, çünkü eşleşen bir kalıp bulduk.

Match ve enum’ları birlikte kullanmak çok yararlıdır. Bu modeli Cairo kodunda sık sık göreceksiniz: bir enum ile eşleştirin, içindeki veriyi bir değişkene bağlayın ve sonra bu veriye göre kod çalıştırın. Bu, çok güçlü ve esnek bir yöntemdir. Bir kere alıştığınızda, başka dillerde de isteyeceksiniz. Cairo kullanıcılarının favorilerinden biridir. 

## Match Kapsamı ##

Match yapısının bir özelliği de, tüm olası durumları kapsaması gerektiğidir. Yani, match kollarının her bir varyant için bir kalıp içermesi ve enum’un tanımlandığı sırada olması gerekir. Aksi takdirde, Cairo bir hata verecektir. Örneğin, `plus_one` fonksiyonunun bu versiyonu hatalıdır:
```
fn plus_one(x: Option<u8>) -> Option<u8> {
    match x {
        Option::Some(val) => Option::Some(val + 1), 
    }
}
```
```
$ cairo-run src/test.cairo
    error: Unsupported match. Currently, matches require one arm per variant,
    in the order of variant definition.
    --> test.cairo:34:5
        match x {
        ^*******^
    Error: failed to compile: ./src/test.cairo```

Cairo, `Option::None` varyantını unuttuğumuzu anlar ve bize bunu söyler. Cairo’daki match’ler ayrıntılıdır: kodun geçerli olması için her ihtimali ele almalıyız. Özellikle **Option** durumunda, Cairo `None` durumunu açıkça işlemememiz durumunda, bizi **null **değerine sahipmiş gibi davranmaktan alıkoyar ve böylece daha önce bahsettiğimiz milyar dolarlık hatayı önler.
