# Dizi (Array)

Dizi, aynı türden öğelerin bir araya getirildiği bir koleksiyondur. `array::ArrayTrait` özelliğini dahil ederek dizi metotları oluşturabilir ve bunları kullanabilirsiniz.

Diziler, sınırlı değişiklik seçenekleri sunarlar. Aslında, bir dizinin değerleri değiştirilemez. Bir dizi, belleğin belirli bir bölümüne yazıldığında, üzerinde değişiklik yapma imkanı yoktur. Bir diziye yalnızca sonuna eleman ekleyebilir ve `pop_front` kullanarak baştaki elemanları çıkarabilirsiniz.

## Dizi Oluşturmak ##

`ArrayTrait::new()` metodu, boş bir dizi oluşturmak için kullanılır. İşte 3 eleman ekleyerek oluşturduğumuz bir dizi örneği:
```
use array::ArrayTrait;

fn main() {
    let mut a = ArrayTrait::new();
    a.append(0);
    a.append(1);
    a.append(2);
}
```

Boş bir dizi oluştururken, içine atanacak verilerin türünü aşağıdaki gibi belirleyebilirsiniz:
```
let mut arr = ArrayTrait::<u128>::new();
```
```
let mut arr:Array<u128> = ArrayTrait::new();
```

Elemanları direk tanımlayarak bir dizi oluşturabilirsiniz:
```
let first = array![1, 2, 3];
```

## Bir Diziyi Güncelleme ##

### Eleman Eklemek ###

Bir dizinin sonuna eleman eklemek için `append()` metodunu kullanabilirsiniz:
```
a.append(0);
```

### Eleman Çıkarmak ###

Bir dizinin başındaki elemanları yalnızca `pop_front()` metodu kullanılarak çıkarabilirsiniz. Bu metot, çıkarılan elemanı içeren bir Option veya dizi boşsa `Option:Option::None` döndürür.
```
use option::OptionTrait;
use array::ArrayTrait;
use debug::PrintTrait;

fn main() {
    let mut a = ArrayTrait::new();
    a.append(10);
    a.append(1);
    a.append(2);

    let first_value = a.pop_front().unwrap();
    first_value.print(); // print '10'
}
```
Yukarıdaki kodu çalıştırdığımızda, eklenen ilk elemanı çıkardığımızda terminale `10` yazdırılacaktır.

## Bir Diziden Elemanları Okumak ##

Dizi elemanlarına erişmek için, farklı türler döndüren `get()` veya `at()` dizi metotlarını kullanabilirsiniz. `arr.at(index) `ve `arr[index]` metotları eşdeğerdir.

`get` fonksiyonu bir `Option<Box<@T>>` döndürür, yani dizi içinde eleman varsa, belirtilen indeksteki elemana anlık görüntü sağlayan bir Box türüne (Cairo'nun akıllı pointer yapısı) bir seçenek döndürür. Eleman yoksa `None` döndürür.

Öte yandan, `at` fonksiyonu, bir kutuda saklanan değeri çıkarmak için `unbox()` işlemcisini kullanarak belirtilen indeksteki elemana doğrudan bir anlık görüntü döndürür. İndeks sınırların dışındaysa, bir panik hatası oluşur.
```
use array::ArrayTrait;
fn main() {
    let mut a = ArrayTrait::new();
    a.append(0);
    a.append(1);

    let first = *a.at(0);
    let second = *a.at(1);
}
```
Bu örnekte, `first` adlı değişken `0` değerini alır ve `second` adlı değişken `1 `değerini alır.
`get()` metoduyla ilgili bir örnek:
```
use array::ArrayTrait;
use box::BoxTrait;
fn main() -> u128 {
    let mut arr = ArrayTrait::<u128>::new();
    arr.append(100);
    let index_to_access =
        1; // Farklı sonuçlar görmek için bu değeri değiştirin
    match arr.get(index_to_access) {
        Option::Some(x) => {
            *x.unbox()
        },
        Option::None(_) => {
            let mut data = ArrayTrait::new();
            data.append('out of bounds');
            panic(data)
        }
    }
}
```

## Boyutla İlgili Metotlar ##

Bir dizideki elemanların sayısını bulmak için `len()` metodu kullanılır. Dönüş değeri `usize` tipindedir.

Bir dizinin boş olup olmadığını öğrenmek istiyorsanız `is_empty()` metodu kullanabilirsiniz. Dizi boşsa `true`, değilse `false` değerini döndürür.

## Spans ##

Span, bir dizi anlık görüntüsünü temsil eden bir yapıdır. Dizi tarafından sağlanan tüm metotlar, `append()` metodu hariç, **Span** ile de kullanılabilir. 

Bir **Span** oluşturmak için, `Arrayspan()` metodu çağrılmalıdır.
```
use array::ArrayTrait;
use array::SpanTrait;

// Receives a Span
fn sum_starting_two(data: Span<u32>) -> u32 {
    // data.append(5_u32); <- this fails
    *data[0] + *data[1]
}

fn main() -> u32 {
    let mut data: Array<u32> = ArrayTrait::new();
    data.append(1_u32);
    data.append(2_u32);
    data.append(3_u32);
    data.append(4_u32);
    data.get(0);
    sum_starting_two(data.span()) // Using a Span
}
```
