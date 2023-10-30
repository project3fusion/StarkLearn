# Yapılar (Struct) #

Yapıları tanımlamak için `struct` anahtar kelimesini kullanırız ve ardından yapıya bir isim veririz. Bu ismin ardından, süslü parantezler içerisinde, 'alan' olarak adlandırdığımız veri parçalarının isimlerini ve türlerini belirtiriz.
```
#[derive(Copy, Drop)]
struct User {
    active: bool,
    username: felt252,
    email: felt252,
    sign_in_count: u64,
}```

Bir yapıyı tanımladıktan sonra, her bir alan için belirli değerler belirleyerek bu yapının bir örneğini oluştururuz. Yapının ismini belirtiriz ve ardından anahtar-değer çiftlerini içeren süslü parantezler ekleriz.
```
#[derive(Copy, Drop)]
struct User {
    active: bool,
    username: felt252,
    email: felt252,
    sign_in_count: u64,
}
fn main() {
    let user1 = User {
        active: true, username: 'someusername123', email: 'someone@example.com', sign_in_count: 1
    };
}```

Bir yapıdan belirli bir değere ulaşmak için nokta gösterimini kullanırız. Örneğin, bu kullanıcının e-posta adresine erişmek için `user1.email` kullanırız. Nokta gösterimini kullanarak belirli bir alandaki değeri değiştirebiliriz.
```
fn main() {
    let mut user1 = User {
        active: true, username: 'someusername123', email: 'someone@example.com', sign_in_count: 1
    };
    user1.email = 'anotheremail@example.com';
}```

Herhangi bir ifadede olduğu gibi, fonksiyon gövdesindeki son ifade olarak struct'ın yeni bir örneğini oluşturarak bu yeni örneği dolaylı olarak döndürebiliriz. Aşağıda verilen `email` ve `username` adıyla bir Kullanici örneği döndüren `build_user `fonksiyonunu gösterir. `active` alanı `true` değerini alır ve `sign_in_count 1` değerini alır.
```
fn build_user(email: felt252, username: felt252) -> User {
    User { active: true, username: username, email: email, sign_in_count: 1,  }
}```

Fonksiyon parametrelerini struct alanlarıyla aynı adla adlandırmak mantıklıdır, ancak e-posta ve kullanıcı adı alan adlarını ve değişkenlerini tekrarlamak zorunda kalmak biraz sıkıcıdır. Yukarıdaki örnek bu durumlar için daha kullanışlıdır.

## Struct İle Basit Bir Alan Hesaplama Örneği ##

Structları daha iyi anlamak için, bir dikdörtgenin alanını hesaplayan bir program yazalım.
```
use debug::PrintTrait;

struct Rectangle {
    width: u64,
    height: u64,
}

fn main() {
    let rectangle = Rectangle { width: 30, height: 10,  };
    let area2 = area(rectangle);
    area2.print(); // alanı yazdır
}

fn area(rectangle: Rectangle) -> u64 {
    rectangle.width * rectangle.height
}
```

Burada bir yapı tanımladık ve adını `Rectangle` koyduk. Süslü parantezlerin içinde, `u64` türünde `width` ve `height` adında iki değişken tanımladık. Daha sonra `main` fonksiyonunda, genişliği 30 ve yüksekliği 10 olan bir `rectangle` örneği oluşturduk. `Rectangle` parametresini alan bir alan hesaplama fonksiyonu olan `area` fonksiyonunu oluşturduk. Ve son olarak `main` içerisinde `Rectangle` parametresini alan `area` fonksiyonunu `area2 `adında bir değişkene atayıp `print()`  ile terminalde yazdırdık.

Eğer aşağıdaki gibi `rectangle`'ı terminalde yazdırmaya çalışsaydık hata alırdık çünkü `print()` özelliği birçok veri türü için uygulanmıştır, ancak `Rectangle` yapısı için uygulanmamıştır.
```
use debug::PrintTrait;

struct Rectangle {
    width: u64,
    height: u64,
}

fn main() {
    let rectangle = Rectangle { width: 30, height: 10,  };
    rectangle.print();
}
```

Kodu çalıştırdığımız zaman aşağıdaki gibi bir hata alırız.
```
error: Method `print` not found on type "../src::Rectangle". Did you import the correct trait and impl?
 --> lib.cairo:16:15
    rectangle.print();
              ^***^

Error: Compilation failed.
```

 Bu sorunu aşağıdaki gibi  `Rectangle` üzerinde `PrintTrait` özelliğini uygulayarak düzeltebiliriz.
```
use debug::PrintTrait;

struct Rectangle {
    width: u64,
    height: u64,
}

fn main() {
    let rectangle = Rectangle { width: 30, height: 10,  };
    rectangle.print();
}

impl RectanglePrintImpl of PrintTrait<Rectangle> {
    fn print(self: Rectangle) {
        self.width.print();
        self.height.print();
    }
}
```
