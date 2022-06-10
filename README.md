# SOLID

## Single Responsibility

- Modül tek bir amaca hizmet eder.
- Her yazılım modülünün değiştirmek için bir ve yalnızca bir nedeni vardır.

### Örnek

- Bir user modelimiz olduğunu düşünelim...

```ts
class User {
    private id: number;
    private name: string;
    private street: string;
    private city: string;
    private username: string;

   // Getters, setters

    public void changeAddress(street: string, city: string) {
        //logic
    }

    public void login(username: string) {
        //logic
    }

    public void logout(username: string) {
        // logic
    }
}

```

- Şuanki isterlerde 3 tane işlem yapılması isteniyor.
  - `changeAddress`
  - `login`
  - `logout`
- Burada yazdığımız `Single Responsibility`'e uygun değil. Login işlemlerini farklı bir yerde, address değişimini farklı bir yerde yapmamız gerekiyor.
- LoginService ve AddressService adına 2 tane service yapalım.

```ts
class LoginService {
  login(username: string) {
    // log-in logic
  }
  
  logout(username: string) {
    // log-out logic
  }
}
class AddressService {
  changeAddress(user: User) {
    // logic
  }
}
```
- Buradaki LoginService güzel ama AddressService o kadar da iyi değil. Çünkü AddressService içine ihtiyacı olmayan user modeli ile çalışıyor. User modelini değiştirelim.
```ts

class Address {
    private id: number;
    private street: string;
    private city: string;
}

class User {
    private id: number;
    private city: string;
    private username: string;

    private address: Address;

}

class AddressService {
  changeAddress(address: Address) {
    //logic
  }
}

```


## Open–Closed Principle

- Software entities ... should be open for extension, but closed for modification.
- Değişime kapalı, geliştirmeye açık olması

## Liskov Substitution Principle

- Türeyen sınıf yani alt sınıflar ana(üst) sınıfın tüm özelliklerini ve metotlarını aynı işlevi gösterecek şekilde kullanabilme ve kendine ait yeni özellikler barındırabilmelidir.

### Örnek

- Bir `Square` ve bir tane `Rectangle` class'ımız var
- Her Square de bir Rectangle'dır ve `Rectangle` classından miras almıştır.

```ts
class Rectangle {
  constructor(width: number, height: number) {
    this.width = width;
    this.height = height;
  }
  getArea() {
    return this.width * this.height;
  }
  setWidth(width: number) {
    this.width = width;
  }
  setHeight(height: number) {
    this.height = height;
  }
}

class Square extends Rectangle {
  constructor(length: number) {
    super(length, length);
  }
}
```

- Bu örnek Liskov'a **UYMUYOR**

```ts
const kare = new Square(10);
kare.setWidth(20);
kare.setHeight(10);
// kare olmaktan çıktı

expect(kare.getArea()).equal(100);
```

- Liskov'a uyması için ne yapmalıyız

```ts
interface Shape {
  getArea(): number;
}

class Rectangle implements Shape {
  constructor(width: number, height: number) {
    this.width = width;
    this.height = height;
  }
  getArea() {
    return this.width * this.height;
  }
  setWidth(width: number) {
    this.width = width;
  }
  setHeight(height: number) {
    this.height = height;
  }
}

class Square implements Shape {
  constructor(length: number) {
    this.length = length;
  }
  setLength(length: number) {
    this.length = length;
  }
  getArea() {
    return this.length * this.length;
  }
}
```

- Artık Square'in setWidth setHeight gibi bir methodu yok.

## Interface Segregation Principle

- Nesneler asla ihtiyacı olmayan property/metot vs içeren interfaceleri implement etmeye zorlanmamalıdır

### Örnek

- Aşağıdaki gibi bir `Phone` abstract class'ı yapalım.

```ts
abstract class Phone {

  call(number) {}

  takePhoto() {}

  connectToWifi() {}

  ...
}

```

```ts
class Nokia3310 extends Phone {
  call(number) {
    // Implementation
  }

  takePhoto() {
    // Bu method'u tanımlamamız gerekiyor ama içinde ne yapacağız ??
  }

  connectToWifi() {
    // Bu method'u tanımlamamız gerekiyor ama içinde ne yapacağız ??
  }
}
```

- Burada yaptığımız genel yanlış şudur. Her telefonda kamera ve/veya wifi olmamasıdır. Bu durum burada fazlalıktır. Bu durum Interface Segregation kuralana aykırıdır.

## Dependency Inversion Principle

- Sınıflar arası bağımlılıklar olabildiğince az olmalıdır özellikle üst seviye sınıflar alt seviye sınıflara bağımlı olmamalıdır.
