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

    changeAddress(street: string, city: string) {
        //logic
    }

    login(username: string) {
        //logic
    }

    logout(username: string) {
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
Artık `AddressService`'i daha da güzel oldu. User'a verisi ile değil de sadece `Address` üzerinden işlem yapıcak...

---


## Open–Closed Principle

- Software entities ... should be open for extension, but closed for modification.
- Değişime kapalı, geliştirmeye açık olması

### Örnek
- Rectangle adında bir class'ımız var.
- AreaServise adında bir service'miz var.
```ts
class Rectangle {
    private length: number; 
    private height: number; 
    // getters/setters ... 
}

class AreaService {

    calculateArea(shapes: Rectangle[]) {
        let area = 0;
        for (const rect of shapes) {
            area += rect.getLength() * rect.getHeight();
        }
        return area;
    }
}

```
- Yapımız bu şekilde hazırlanmış.
- Yeni bir feature ekleme ihtiyacımız oluştu. Uygulamamıza `Circle` adında bir özellik daha eklememiz gerekti.

```ts
class Circle {
    private radius: number; 
    // getters/setters ... 
}
```
`AreaService`'i güncellerken aşağıdaki gibi **YAPMAMALIYIZ**
```ts
class AreaService {

    calculateArea(shapes: any[]) {
        let area = 0;
        for (const shape of shapes) {
            if(rect instanceOf Rectangle) {
                area += shape.getLength() * shape.getHeight();
            } else if(shape instanceOf Circle) {
                area += shape.getRadius() * shape.getRadius() * Math.PI;
            }
        }
        return area;
    }
}

```
- Nasıl olmalı
```ts
interface Shape {
    getArea(): number;
}

class Circle implements Shape {
    private radius: number; 
    // getters/setters ... 

    getArea(){
        return this.radius * this.radius * Math.PI;
    }
}

class Rectangle {
    private length: number; 
    private height: number; 
    // getters/setters ... 
     getArea(){
        return this.length * this.height;
    }
}

class AreaService {

    calculateArea(shapes: Shape[]) {
        let area = 0;
        for (const shape of shapes) {
            area += shape.getArea();
        }
        return area;
    }
}
```
### Artıları
- Yeni bir tür, Circle ekledik
- Rectangle hala eskisi gibi çalışıyor.
- AreaService'e calculateArea'yı çağıran fonksiyonlarda herhangi bir değişikliğe gitmedik.

---
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

### Örnek
- Backend projemizden örnek verelim `httpRequestService`'i inceleyelim
```js
const axios = require("axios");
const debug = require("debug")("httpRequestService");

module.exports = ({logService}) => ({
    async request({method, url, headers, data, log, params}) {
        const response = await axios({method, url, headers, data, params});
        if (log) {
            logService.info(log.type, {
                request: {method, url, headers, data},
                response: {
                    status: response.status,
                    statusText: response.statusText,
                    data: response.data
                }
            });
        }
        return response;
    }
});

```
- Buradaki yapı bana göre güzel :)
- Ama şöyle bir gerçek var. `httpRequestService` logService'e bağımlı durumda. Bu servis'e logService'in yanına slack'e yazma ve email gönderme istenilseydi nasıl yapardık.

- Böyle bir durumda, bu Mesaj servislerinin ortak bir interface'i olması ve bu service'de bunlar array olarak tutulması isteniliyor...

