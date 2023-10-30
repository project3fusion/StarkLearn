# Özellikler (Traits) #

Özellikler, uygulanabilecek işlevsellik taslaklarını belirtir. Bu taslak spesifikasyonu, parametreler ve dönüş değeri için tip ek açıklamaları içeren bir dizi fonksiyon imzasını içerir. Bu, belirli bir işlevselliği uygulamak için bir standart belirler.

## Bir Özellik Tanımlama ##

Bir özellik tanımlamak için, `PascalCase`'de özelliğin adından sonra özellik anahtar kelimesini kullanırız ve ardından süslü parantezler içerisinde fonksiyon imzalarını yerleştiririz.

Örneğin, birçok yapıyı temsil eden şekillerimiz var diyelim. Uygulamamızın bu şekiller üzerinde geometri işlemleri gerçekleştirebilmesini istiyoruz. Bu nedenle, bir şekil üzerinde geometri işlemlerini uygulama taslağını içeren `ShapeGeometry `adında bir özellik tanımlıyoruz:
```rs
trait ShapeGeometry {
fn boundary(self: Rectangle) -> u64;
fn area(self: Rectangle) -> u64;
}
Burada, ShapeGeometry özelliğimiz boundary ve area adlı iki metot için imzalar deklare eder. Uygulandığında, her iki fonksiyon da özelliğin belirttiği gibi parametreleri kabul etmeli ve bir u64 döndürmelidir.

Bir Özelliği Uygulama
Bir özellik, uygulamanızın adı ile "of" kelimesini takiben uygulanacak özelliğin adını kullanarak "impl" anahtar kelimesi ile uygulanabilir. ShapeGeometry özelliğini uygulayan bir örnek aşağıda verilmiştir.

impl RectangleGeometry of ShapeGeometry {
fn boundary(self: Rectangle) -> u64 {
2 * (self.height + self.width)
}
fn area(self: Rectangle) -> u64 {
self.height * self.width
}
}
```
Yukarıdaki kodda, RectangleGeometry, ShapeGeometry özelliğini uygular ve boundary ve area metodlarının ne yapması gerektiğini tanımlar. Fonksiyon parametreleri ve dönüş değeri tipleri, özellik spesifikasyonuyla aynıdır.

## Trait Bildirimi Olmadan Trait Uygulama ##

Bir özellik tanımlamadan doğrudan onun uygulamasını yazabilirsiniz. Bu, uygulama üzerinde #[generate_trait] özniteliği kullanılarak sağlanır. Bu özellik, derleyicinin uygulamaya karşılık gelen özelliği otomatik olarak oluşturmasını sağlar. Özelliğin adını belirtirken "Trait" son ekini eklemeyi unutmayın, çünkü derleyici uygulama adına "Trait" son ekini ekleyerek özelliği oluşturacaktır.
```rs
struct Rectangle {
    height: u64,
    width: u64,
}

#[generate_trait]
impl RectangleGeometry of RectangleGeometryTrait {
    fn boundary(self: Rectangle) -> u64 {
        2 * (self.height + self.width)
    }
    fn area(self: Rectangle) -> u64 {
        self.height * self.width
    }
}
```
Yukarıdaki kodda özelliği manuel olarak tanımlamanıza gerek kalmaz. Derleyici otomatik olarak tanımını yapacak ve yeni fonksiyonlar eklendikçe onu güncelleyecektir.

## Self Parametresi ##

Yukarıdaki örnekte, "self" özel bir parametredir. "Self" isimli bir parametre kullanıldığında, uygulanan fonksiyonlar türün örneklerine metotlar olarak bağlanır.
```rs
fn main() {
    let rect = Rectangle { height: 5, width: 10 };
    let area1 = rect.area();
    let area2 = RectangleGeometry::area(rect);
    let area3 = ShapeGeometry::area(rect);
    area1.print();
    area2.print();
    area3.print();
}
```
## Genel Özellikler ##

Birden fazla türün bir işlevselliği standart bir şekilde uygulamasını istediğimizde bir özellik yazarız. Ancak yukarıdaki örnekte imzalar sabittir. Birden fazla tür için bu imzaları kullanabilmek adına özelliklerde genel türleri kullanırız.
```rs
use debug::PrintTrait;

#[derive(Copy, Drop)]
struct Rectangle {
    height: u64,
    width: u64,
}

#[derive(Copy, Drop)]
struct Circle {
    radius: u64
}

trait ShapeGeometry<T> {
    fn boundary(self: T) -> u64;
    fn area(self: T) -> u64;
}

impl RectangleGeometry of ShapeGeometry<Rectangle> {
    fn boundary(self: Rectangle) -> u64 {
        2 * (self.height + self.width)
    }
    fn area(self: Rectangle) -> u64 {
        self.height * self.width
    }
}

impl CircleGeometry of ShapeGeometry<Circle> {
    fn boundary(self: Circle) -> u64 {
        (2 * 314 * self.radius) / 100
    }
    fn area(self: Circle) -> u64 {
        (314 * self.radius * self.radius) / 100
    }
}

fn main() {
    let rect = Rectangle { height: 5, width: 7 };
    rect.area().print();
    rect.boundary().print();

    let circ = Circle { radius: 5 };
    circ.area().print();
    circ.boundary().print();
}
```
Bu kodlar, farklı yapılar için nasıl özellikler ve uygulamalar oluşturulduğunu gösterir.
