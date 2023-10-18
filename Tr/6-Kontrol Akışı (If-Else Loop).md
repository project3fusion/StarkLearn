# Kontrol Akışı (If-Else/Loop) #
Bir programlama dilinin temel özelliklerinden biri, belirli bir duruma bağlı olarak kod parçalarını çalıştırabilme ve bir durum doğru olduğunda belirli kodları tekrarlamadır. Cairo kodunun yürütme akışını yönetmek için en yaygın kullanılan yapılar `if` ifadeleri ve döngülerdir.

## IF ifadesi ##
Cairo programlama dilinde `if` ifadesi, belirli bir durumun doğru olup olmadığını denetlemek için kullanılır. Eğer durum doğruysa, `if` bloğu içindeki kod çalıştırılır. Eğer durum yanlışsa, `else` bloğu çalıştırılır. Eğer `else` bloğu yoksa, `if` bloğu atlanır ve program devam eder.
```
use debug::PrintTrait;

fn main() {
    let sayi = 3;

    if sayi == 5 {
        'kosul dogrudur'.print();
    } else {
        'kosul yanlistir'.print();
    }
}
```
Bu kodda, durumun bir boolean olması gerektiğini belirtmek önemlidir. Eğer durum bir boolean değilse, aşağıdaki gibi bir hata alınır.
```
thread 'main' panicked at 'Failed to specialize: `enum_match<felt252>`. Error: Could not specialize libfunc `enum_match` with generic_args: [Type(ConcreteTypeId { id: 1, debug_name: None })]. Error: Provided generic argument is unsupported.', crates/cairo-lang-sierra-generator/src/utils.rs:256:9
```

`&&` (ve), `||` (veya) gibi boolean operatörlerini kullanarak birden fazla durumu birleştirebilirsiniz.
```
use debug::PrintTrait;

fn main() {
    let version: u8 = 2;
    let is_awesome = true;

    if is_awesome && version > 0 {
        'Lets code!'.print();
    } else {
        'Great things are coming'.print();
    }
}
```
## Birden Fazla Durumu `else if` İle İşleme ##

`else if` ifadesi, bir durumu kontrol eder ve eğer durum doğruysa belirli bir kod bloğunu çalıştırır. Eğer durum yanlışsa, başka bir durumu kontrol eder ve eğer bu durum doğruysa başka bir kod bloğunu çalıştırır. Bu işlem, istediğimiz kadar `else if` bloğu ekleyerek genişletilebilir.
```
use debug::PrintTrait;

fn main() {
    let number = 3;

    if number == 12 {
        'number is 12'.print();
    } else if number == 3 {
        'number is 3'.print();
    } else if number - 2 == 1 {
        'number minus 2 is 1'.print();
    } else {
        'number not found'.print();
    }
}
```
Bu programın dört farklı çıktısı olabilir. Program çalıştırıldığında, her `if` ifadesini sırasıyla kontrol eder ve ilk doğru durumu bulduğunda ilgili kod bloğunu çalıştırır.
```
[DEBUG]    number is 3
```

## `let` ile `if` İfadesini Kullanmak ##

Bir sorgunun sonucunu bir değişkene atamak için let ifadesini kullanabiliriz.
```
use debug::PrintTrait;

fn main() {
    let condition = true;
    let number = if condition {
        5
    } else {
        6
    };

    if number == 5 {
        'condition was true'.print();
    }
}
```
`number` değişkeni, `if` ifadesinin sonucuna bağlı olarak bir değere atanır. Bu programı `scarb cairo-run` komutu ile çalıştırdığımızda aşağıdaki çıktıyı alırız.
```
[DEBUG]    condition was true
```

## Döngüler ##

`loop` anahtar kelimesi, Cairo'ya bir kod bloğunu sürekli olarak (gaz bitene kadar) veya siz durdurana kadar tekrar tekrar çalıştırmasını söyler. Cairo'nun şu an için sadece `loop` döngü türü bulunmaktadır.
```
use debug::PrintTrait;
fn main() {
    let mut i: usize = 0;
    loop {
        if i > 10 {
            break;
        }
        'again!'.print();
    }
}
```

Bu programı çalıştırdığımızda, programı manuel olarak durdurana kadar sürekli olarak `again!` yazdırılır, çünkü durdurma durumu asla gerçekleşmez.

Programı çalıştırmak için `scarb cairo-run --available-gas 200000` komutunu kullanın.

Not: Cairo, bir gaz sayacı ekleyerek programın sonsuz döngülerde çalışmasını engeller. Gaz sayacı, bir programda gerçekleştirilebilecek hesaplama miktarını sınırlar. `--available-gas` bayrağına bir değer ayarlayarak, program için kullanılabilir maksimum gaz miktarını belirleyebiliriz. Gaz, bir talimatın hesaplama maliyetini ifade eden bir ölçü birimidir. Gaz sayacı bittiğinde, program durur.
`break` anahtar kelimesi, döngünün ne zaman duracağını belirtir.
```
use debug::PrintTrait;
fn main() {
    let mut i: usize = 0;
    loop {
        if i > 10 {
            break;
        }
        'again'.print();
        i += 1;
    }
}
```

`continue` anahtar kelimesi, programa döngünün bir sonraki iterasyonuna geçmesini ve bu iterasyondaki kodun geri kalanını atlamasını söyler. `i` 5'e eşit olduğunda `print` ifadesini atlamak için döngümüze bir `continue` ifadesi ekleyelim.
```
use debug::PrintTrait;
fn main() {
    let mut i: usize = 0;
    loop {
        if i > 10 {
            break;
        }
        if i == 5 {
            i += 1;
            continue;
        }
        i.print();
        i += 1;
    }
}
```
Bu programı çalıştırdığında `i` değeri 5'e eşit olduğunda terminalde bir çıktı vermez.

## Döngülerden Değer Döndürme ##

Bir döngünün kullanımlarından biri, başarılı olup olmadığını kontrol etmek gibi başarısız olabileceğini bildiğiniz bir işlemi yeniden denemektir. Ayrıca, bu işlemin sonucunu döngüden kodunuzun geri kalanına aktarmanız gerekebilir. Bunu yapmak için, döngüyü durdurmak için kullandığınız` break` ifadesinden sonra döndürmek istediğiniz değeri ekleyebilirsiniz; bu değer döngüden döndürülür, böylece onu kullanabilirsiniz:

```
fn main() {
    let mut sayaç = 0;

    let sonuç = loop {
        if sayaç == 10 {
            break sayaç;
        }
        sayaç += 1;
    };

    println!("Sonuç: {}", sonuç);
}
```
Döngüden önce, sayaç adında bir değişken tanımlıyoruz ve onu `0`'a başlatıyoruz.Döngünün her yinelemesinde, sayaç değişkeninin 10'a eşit olup olmadığını kontrol ediyoruz ve sayaç değişkenine 1 ekliyoruz. Koşul karşılandığında, `sayaç` değeri ile break anahtar kelimesini kullanıyoruz. Son olarak, sonuçtaki değeri yazdırıyoruz, bu durumda bu değer 10'dur.
