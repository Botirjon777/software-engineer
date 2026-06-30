# OOP va SOLID

Object-Oriented Programming (OOP) — bu dasturlashda kodni real dunyodagi "obyektlar" ko'rinishida modellashtiruvchi paradigma. Ushbu hujjat OOP ning to'rt ustunini, composition vs inheritance falsafasini, coupling/cohesion tushunchalarini va SOLID prinsiplarini intervyu darajasida, TypeScript misollari bilan tushuntiradi.

OOP ni yaxshi bilish — har qanday katta loyihada (Angular, NestJS, enterprise backend) toza arxitektura qurishning poydevoridir. SOLID esa "yaxshi OOP kod" ni "yomon OOP kod" dan ajratuvchi besh asosiy prinsipdir.

## Mundarija

- [OOP nima va nega kerak](#oop-nima-va-nega-kerak)
- [Class vs Object](#class-vs-object)
- [4 ustun: Encapsulation](#1-encapsulation-inkapsulyatsiya)
- [4 ustun: Abstraction](#2-abstraction-abstraksiya)
- [4 ustun: Inheritance](#3-inheritance-merosxorlik)
- [4 ustun: Polymorphism](#4-polymorphism-polimorfizm)
- [Composition vs Inheritance](#composition-vs-inheritance)
- [Coupling va Cohesion](#coupling-va-cohesion)
- [SOLID prinsiplari](#solid-prinsiplari)
- [DRY, KISS, YAGNI](#dry-kiss-yagni)
- [OOP vs Functional](#oop-vs-functional)
- [Savol-Javoblar](#savol-javoblar)
- [Masalalar](#masalalar)

---

## OOP nima va nega kerak

**💡 Tushuncha:** OOP — bu ma'lumot (state) va shu ma'lumot ustida ishlovchi xatti-harakatlarni (behavior) bitta birlik — **object** ichida birlashtiruvchi paradigma.

OOP dan oldin protsedurali dasturlashda ma'lumotlar va funksiyalar alohida yashar edi. Loyiha o'sgani sayin "qaysi funksiya qaysi ma'lumotni o'zgartiradi" ni kuzatish qiyinlashardi. OOP buni hal qiladi:

- **Modullik (modularity):** har bir object o'z mas'uliyatiga ega, mustaqil.
- **Qayta ishlatish (reusability):** inheritance va composition orqali kod takrorlanmaydi.
- **Kengaytirish (maintainability):** o'zgarishlar lokal qoladi, butun tizimga tarqalmaydi.
- **Abstraksiya:** murakkablik object ortida yashiriladi.

```ts
// Protsedurali yondashuv — ma'lumot va xatti-harakat ajralgan
const user = { name: "Ali", balance: 100 };
function withdraw(u: { balance: number }, amount: number) {
  u.balance -= amount; // har kim o'zgartira oladi, nazorat yo'q
}

// OOP yondashuv — birlashtirilgan va himoyalangan
class Account {
  private balance = 100;
  withdraw(amount: number): void {
    if (amount > this.balance) throw new Error("Mablag' yetarli emas");
    this.balance -= amount;
  }
}
```

---

## Class vs Object

**💡 Tushuncha:** **Class** — bu chizma (blueprint), **object** — shu chizmadan yasalgan aniq nusxa (instance).

Uy qurilish chizmasi — class. Shu chizma asosida qurilgan aniq uylar — objectlar. Bitta classdan cheksiz ko'p object yaratish mumkin, har biri o'z state ga ega.

```ts
class Car {
  constructor(public brand: string, private speed = 0) {}
  accelerate(): void {
    this.speed += 10;
  }
}

const tesla = new Car("Tesla"); // object (instance) #1
const bmw = new Car("BMW");      // object (instance) #2
tesla.accelerate(); // faqat tesla ning speed i o'zgaradi, bmw ga ta'sir qilmaydi
```

---

## 4 ustun: Encapsulation (inkapsulyatsiya)

**💡 Tushuncha:** Encapsulation — internal state ni tashqaridan yashirish va unga faqat nazorat qilingan interfeys orqali kirish imkonini berish.

TypeScript da access modifiers:

- `public` — hamma joydan kirish mumkin (default).
- `private` — faqat shu class ichidan.
- `protected` — shu class va undan meros olgan subclasslardan.
- `readonly` — faqat o'qish mumkin, yaratilgandan keyin o'zgarmaydi.

```ts
class BankAccount {
  private _balance = 0;
  readonly owner: string;

  constructor(owner: string) {
    this.owner = owner;
  }

  // getter — nazorat qilingan o'qish
  get balance(): number {
    return this._balance;
  }

  // setter — validatsiya bilan yozish
  deposit(amount: number): void {
    if (amount <= 0) throw new Error("Summa musbat bo'lishi kerak");
    this._balance += amount;
  }
}

const acc = new BankAccount("Ali");
acc.deposit(500);
console.log(acc.balance); // 500
// acc._balance = -999; // ❌ compile error — private
```

**⚠️ Ehtiyot bo'l:** TypeScript ning `private` faqat compile-time da ishlaydi. Runtime da haqiqiy himoya kerak bo'lsa, JavaScript ning `#private` field laridan foydalaning: `#balance = 0`.

---

## 4 ustun: Abstraction (abstraksiya)

**💡 Tushuncha:** Abstraction — "qanday qilib" (implementation) ni yashirib, faqat "nima qiladi" (interface) ni ko'rsatish.

Mashina haydashda dvigatel ichida nima bo'layotganini bilmaysiz — faqat pedal va rul bilan ishlaysiz. Bu abstraksiya.

TypeScript da abstraksiya ikki vosita orqali:

```ts
// 1. abstract class — qisman implementatsiya bo'lishi mumkin
abstract class Shape {
  abstract area(): number; // implementatsiyasiz — subclass majbur
  describe(): string {
    return `Bu shaklning yuzasi: ${this.area()}`;
  }
}

class Circle extends Shape {
  constructor(private r: number) {
    super();
  }
  area(): number {
    return Math.PI * this.r ** 2;
  }
}

// 2. interface — faqat shartnoma (contract), implementatsiya yo'q
interface Repository<T> {
  findById(id: string): T | null;
  save(entity: T): void;
}
```

**⚠️ Ehtiyot bo'l:** `abstract class` ni `new` bilan yaratib bo'lmaydi. Interface esa runtime da umuman mavjud emas — u faqat compile vaqtida tekshiriladi va JS ga kompilyatsiya qilinganda yo'qoladi.

---

## 4 ustun: Inheritance (merosxorlik)

**💡 Tushuncha:** Inheritance — bir class (child/subclass) boshqa class (parent/superclass) ning xususiyat va metodlarini meros olishi. "is-a" munosabati.

```ts
class Animal {
  constructor(protected name: string) {}
  move(): void {
    console.log(`${this.name} harakatlanmoqda`);
  }
}

class Dog extends Animal {
  constructor(name: string, private breed: string) {
    super(name); // parent constructor ni chaqirish — MAJBURIY
  }
  bark(): void {
    console.log(`${this.name} (${this.breed}) vov-vov!`);
  }
}

const rex = new Dog("Rex", "Husky");
rex.move(); // meros olingan
rex.bark(); // o'zining
```

`super` ikki maqsadda: (1) `super(...)` — parent constructor; (2) `super.method()` — parent metodini chaqirish.

**⚠️ Ehtiyot bo'l:** Chuqur inheritance ierarxiyalari (3+ daraja) "fragile base class" muammosiga olib keladi — parent o'zgarsa, butun ierarxiya buziladi. Shuning uchun composition afzalroq (pastda).

---

## 4 ustun: Polymorphism (polimorfizm)

**💡 Tushuncha:** Polymorphism — "ko'p shakllilik". Bir interfeys ostida turli xil obyektlar har xil xatti-harakat ko'rsatishi.

**Override (qayta yozish)** — subclass parent metodini o'z versiyasi bilan almashtiradi:

```ts
class Notification {
  send(): string {
    return "Umumiy xabar";
  }
}
class EmailNotification extends Notification {
  send(): string {
    return "Email yuborildi"; // override
  }
}
class SmsNotification extends Notification {
  send(): string {
    return "SMS yuborildi"; // override
  }
}

// Runtime polymorphism — qaysi send() chaqirilishi runtime da hal bo'ladi
const list: Notification[] = [new EmailNotification(), new SmsNotification()];
list.forEach((n) => console.log(n.send()));
```

**Overload (qayta yuklash)** — bir nom, turli parametrlar. TypeScript da overload signature lar:

```ts
class Calculator {
  add(a: number, b: number): number;
  add(a: string, b: string): string;
  add(a: any, b: any): any {
    return a + b; // bitta implementatsiya
  }
}
```

**Duck typing** — "agar g'oz kabi yursa va g'oz kabi qag'illasa, u g'ozdir". Tip emas, struktura muhim:

```ts
interface Quacker {
  quack(): void;
}
function makeItQuack(thing: Quacker) {
  thing.quack();
}
// Quacker ni implements qilmagan, lekin quack() bor — ishlaydi (structural typing)
makeItQuack({ quack: () => console.log("Qag'-qag'") });
```

**⚠️ Ehtiyot bo'l:** TypeScript "structural typing" ishlatadi (duck typing ga o'xshash). Java/C# esa "nominal typing". Shuning uchun TS da explicit `implements` shart emas — struktura mos kelsa yetarli.

---

## Composition vs Inheritance

**💡 Tushuncha:** Inheritance — "is-a" (Dog is-a Animal). Composition — "has-a" (Car has-a Engine). Zamonaviy tavsiya: **"Favor composition over inheritance"**.

```ts
// ❌ Inheritance bilan — ierarxiya portlaydi
// FlyingCar extends Car? SwimmingCar extends Car? FlyingSwimmingCar?

// ✅ Composition bilan — moslashuvchan
interface Engine {
  start(): void;
}
class ElectricEngine implements Engine {
  start() {
    console.log("Jimgina ishga tushdi");
  }
}
class Car {
  constructor(private engine: Engine) {} // has-a Engine
  drive() {
    this.engine.start();
  }
}

// Engine ni istalgan vaqtda almashtirish mumkin — flexible
const car = new Car(new ElectricEngine());
```

Nima uchun composition afzal:
- **Moslashuvchanlik:** runtime da xatti-harakatni almashtirish mumkin.
- **Past coupling:** classlar bir-biriga zich bog'lanmaydi.
- **Test qilish oson:** dependency larni mock qilish oson.

---

## Coupling va Cohesion

**💡 Tushuncha:** **Coupling** — modullar orasidagi bog'liqlik darajasi (kam bo'lsa yaxshi). **Cohesion** — bitta modul ichidagi elementlarning bir maqsadga xizmat qilishi (yuqori bo'lsa yaxshi).

Maqsad: **Low Coupling, High Cohesion**.

```ts
// ❌ High coupling — OrderService to'g'ridan-to'g'ri konkret klassga bog'langan
class OrderService {
  private db = new MySQLDatabase(); // qattiq bog'lanish
}

// ✅ Low coupling — abstraksiyaga bog'langan
interface Database {
  save(data: unknown): void;
}
class OrderService {
  constructor(private db: Database) {} // istalgan DB ni berish mumkin
}
```

- **High cohesion:** `UserValidator` faqat validatsiya bilan shug'ullanadi, email yuborish bilan emas.
- **Low coupling:** modullar interface lar orqali gaplashadi, konkret implementatsiyalar orqali emas.

---

## SOLID prinsiplari

SOLID — Robert C. Martin (Uncle Bob) tomonidan ommalashtirilgan 5 prinsip. Maqsad: o'qiladigan, kengaytiriladigan, oson o'zgaradigan OOP kod.

### S — Single Responsibility Principle (SRP)

**💡 Tushuncha:** Har bir class faqat bitta sababga ko'ra o'zgarishi kerak — ya'ni faqat bitta mas'uliyatga ega bo'lishi kerak.

```ts
// ❌ Buzilgan — User class uchta ish qiladi
class User {
  save() {}       // database
  sendEmail() {}  // notifications
  toJSON() {}     // serialization
}

// ✅ To'g'ri — har biri o'z mas'uliyati
class User {}
class UserRepository {
  save(user: User) {}
}
class EmailService {
  sendWelcome(user: User) {}
}
```

### O — Open/Closed Principle (OCP)

**💡 Tushuncha:** Class lar kengaytirishga **ochiq**, o'zgartirishga **yopiq** bo'lishi kerak. Yangi xatti-harakat qo'shganda eski kodni o'zgartirmaslik.

```ts
// ❌ Har yangi shakl uchun if qo'shamiz — yopiq emas
class AreaCalculator {
  area(shape: any): number {
    if (shape.type === "circle") return Math.PI * shape.r ** 2;
    if (shape.type === "square") return shape.side ** 2;
    return 0; // har safar bu metodni o'zgartiramiz
  }
}

// ✅ Yangi shakl = yangi class, eski kod tegilmaydi
interface Shape {
  area(): number;
}
class Circle implements Shape {
  constructor(private r: number) {}
  area() {
    return Math.PI * this.r ** 2;
  }
}
class Square implements Shape {
  constructor(private side: number) {}
  area() {
    return this.side ** 2;
  }
}
```

### L — Liskov Substitution Principle (LSP)

**💡 Tushuncha:** Subclass har doim o'z parent class i o'rniga ishlay olishi kerak — dasturni buzmasdan.

```ts
// ❌ LSP buzilishi — Penguin ucha olmaydi, lekin Bird da fly bor
class Bird {
  fly(): void {}
}
class Penguin extends Bird {
  fly(): void {
    throw new Error("Pingvinlar ucha olmaydi!"); // shartnomani buzdi
  }
}

// ✅ To'g'ri abstraksiya
interface Bird {
  eat(): void;
}
interface FlyingBird extends Bird {
  fly(): void;
}
class Sparrow implements FlyingBird {
  eat() {}
  fly() {}
}
class Penguin implements Bird {
  eat() {} // fly yo'q — to'g'ri
}
```

### I — Interface Segregation Principle (ISP)

**💡 Tushuncha:** Mijozlar (client) o'zlari ishlatmaydigan metodlarga bog'liq bo'lmasligi kerak. Katta interfeyslarni kichiklarga bo'ling.

```ts
// ❌ "Fat interface" — Robot uxlamaydi, lekin sleep ni implement qilishga majbur
interface Worker {
  work(): void;
  eat(): void;
  sleep(): void;
}

// ✅ Bo'lingan interfeyslar
interface Workable {
  work(): void;
}
interface Eatable {
  eat(): void;
}
class Robot implements Workable {
  work() {} // faqat kerakligi
}
class Human implements Workable, Eatable {
  work() {}
  eat() {}
}
```

### D — Dependency Inversion Principle (DIP)

**💡 Tushuncha:** Yuqori darajadagi modullar past darajadagi modullarga emas, **abstraksiyalarga** bog'liq bo'lishi kerak. "Details depend on abstractions".

```ts
// ❌ Yuqori daraja (Service) past darajaga (MySQL) bog'langan
class MySQLDatabase {
  save(d: string) {}
}
class UserService {
  private db = new MySQLDatabase(); // qattiq bog'lanish
}

// ✅ Ikkisi ham abstraksiyaga bog'langan
interface Database {
  save(d: string): void;
}
class MySQLDatabase implements Database {
  save(d: string) {}
}
class PostgresDatabase implements Database {
  save(d: string) {}
}
class UserService {
  constructor(private db: Database) {} // dependency injection
}

new UserService(new PostgresDatabase()); // istalganini berish mumkin
```

**⚠️ Ehtiyot bo'l:** DIP va Dependency Injection (DI) bir narsa emas. DIP — prinsip (abstraksiyaga bog'lan). DI — uni amalga oshirish texnikasi (dependency ni tashqaridan ber). DI Container (NestJS, Angular) — bu jarayonni avtomatlashtiruvchi vosita.

---

## DRY, KISS, YAGNI

- **DRY (Don't Repeat Yourself):** bir bilim/logika kodda faqat bir joyda yashashi kerak. Takrorlanishni funksiya/class ga chiqaring.
- **KISS (Keep It Simple, Stupid):** eng oddiy yechimni tanlang. Murakkablik — dushman.
- **YAGNI (You Aren't Gonna Need It):** hozir kerak bo'lmagan funksionallikni "kelajakda kerak bo'lar" deb yozmang.

**⚠️ Ehtiyot bo'l:** DRY ni haddan oshirib yubormang. Ba'zan ikkita o'xshash kod aslida turli sabablarga ko'ra o'zgaradi — ularni majburan birlashtirsangiz, coupling oshadi. "Premature abstraction" — DRY ning teskari tomoni.

---

## OOP vs Functional

**💡 Tushuncha:** OOP — state va behavior ni object da birlashtiradi (mutable state, encapsulation). Functional Programming (FP) — sof funksiyalar, immutability va kompozitsiyaga tayanadi.

| Jihat | OOP | Functional |
|-------|-----|------------|
| Asosiy birlik | Object | Funksiya |
| State | Mutable, encapsulated | Immutable |
| Asosiy g'oya | Inheritance, polymorphism | Pure functions, composition |
| Side effect | Ruxsat etiladi | Minimallashtiriladi |

Amalda TypeScript ikkisini ham qo'llab-quvvatlaydi — gibrid yondashuv ko'p qo'llaniladi (masalan React'da FP, NestJS'da OOP).

---

## Savol-Javoblar

### ❓ OOP ning 4 ustunini sanab bering va qisqacha izohlang.

**✅ Javob:** **Encapsulation** (state ni yashirish va nazorat qilingan kirish), **Abstraction** (implementatsiyani yashirib interface ni ko'rsatish), **Inheritance** (parent class dan xususiyat meros olish, "is-a"), **Polymorphism** (bir interfeys ostida turli xatti-harakat). Bular birgalikda modulli, qayta ishlatiluvchi va kengaytiriluvchi kod yaratadi.

### ❓ Encapsulation va Abstraction orasidagi farq nima?

**✅ Javob:** Encapsulation — bu **mexanizm**: ma'lumotni `private` qilib, faqat metodlar orqali kirish (qanday himoyalash). Abstraction — bu **dizayn darajasi**: keraksiz tafsilotlarni yashirib, faqat muhimini ko'rsatish (nimani ko'rsatish). Encapsulation "qanday yashirish", abstraction "nimani yashirish" haqida.

### ❓ TypeScript da `private` haqiqiy himoyami?

**✅ Javob:** Yo'q, TS ning `private` faqat compile-time da ishlaydi — JS ga kompilyatsiya qilingach yo'qoladi va runtime da kirish mumkin. Haqiqiy runtime himoya kerak bo'lsa ECMAScript ning `#field` (hash) private field laridan foydalaning, ular runtime da ham himoyalangan.

### ❓ `abstract class` va `interface` orasidagi farq?

**✅ Javob:** `abstract class` — qisman implementatsiya bo'lishi mumkin (konkret metodlar, holat), bir class faqat bitta abstract class dan meros oladi, runtime da mavjud. `interface` — faqat shartnoma, implementatsiyasiz, bir class bir nechta interface ni implement qila oladi, runtime da yo'qoladi. Umumiy logika ulashish kerak bo'lsa — abstract class, faqat shartnoma kerak bo'lsa — interface.

### ❓ Override va Overload farqi?

**✅ Javob:** **Override** — subclass parent ning metodini bir xil signature bilan qayta yozadi (runtime polymorphism). **Overload** — bir nom ostida turli parametr ro'yxatlari (compile-time). TypeScript da overload alohida signature lar bilan e'lon qilinadi, lekin bitta implementatsiya bo'ladi.

### ❓ "Favor composition over inheritance" nima degani va nega?

**✅ Javob:** Xatti-harakatni meros olish o'rniga, kerakli komponentlarni object ga "qo'shish" (has-a) afzal. Sababi: inheritance qattiq, chuqur ierarxiyalar mo'rt ("fragile base class"), runtime da o'zgartirib bo'lmaydi. Composition esa moslashuvchan, coupling past, test qilish oson. Inheritance ni faqat haqiqiy "is-a" munosabati bo'lganda ishlating.

### ❓ Coupling va Cohesion nima? Qaysi yaxshi?

**✅ Javob:** Coupling — modullar orasidagi bog'liqlik (**low** yaxshi). Cohesion — modul ichidagi elementlarning bir maqsadga birlashishi (**high** yaxshi). Maqsad: low coupling, high cohesion. Bu o'zgarishlarni lokal qiladi va modullarni mustaqil test qilish imkonini beradi.

### ❓ SRP nima va uni qanday tushunasiz?

**✅ Javob:** Single Responsibility Principle — har bir class faqat bitta o'zgarish sababiga ega bo'lishi kerak. "Mas'uliyat" — bu o'zgarish uchun bir manba/aktor. Masalan, User class ni database, email va serialization uchun uchta alohida class ga ajratish kerak, chunki bu uch jihat turli sabablarga ko'ra o'zgaradi.

### ❓ OCP ni amalda qanday ta'minlaysiz?

**✅ Javob:** Polymorphism va abstraksiya orqali. `if/switch` bilan tip tekshirish o'rniga, har bir variant uchun umumiy interface ni implement qiluvchi alohida class yarating. Yangi xatti-harakat qo'shilganda eski kod o'zgarmaydi — faqat yangi class qo'shiladi (Strategy pattern bu prinsipga asoslangan).

### ❓ LSP buzilishiga misol keltiring.

**✅ Javob:** Klassik misol — `Square extends Rectangle`. Rectangle da `setWidth`/`setHeight` mustaqil, lekin Square da ular o'zaro bog'liq. Rectangle kutgan kod Square bilan noto'g'ri ishlaydi. Yana: `Penguin extends Bird` da `fly()` exception tashlasa — LSP buziladi. Yechim: to'g'ri abstraksiya (FlyingBird ni ajratish).

### ❓ ISP qanday muammoni hal qiladi?

**✅ Javob:** "Fat interface" muammosini — bir katta interfeys mijozlarni ishlatmaydigan metodlarga bog'lab qo'yadi. Masalan, Robot `Worker` interfeysidagi `sleep()` ni implement qilishga majbur bo'ladi. ISP buni kichik, fokuslangan interfeyslarga bo'lib hal qiladi (Workable, Eatable).

### ❓ DIP va Dependency Injection bir narsami?

**✅ Javob:** Yo'q. DIP — **prinsip**: yuqori daraja past darajaga emas, abstraksiyaga bog'lansin. Dependency Injection — bu prinsipni amalga oshiruvchi **texnika**: dependency ni class ichida yaratmasdan, tashqaridan (constructor/setter orqali) berish. DI Container — DI ni avtomatlashtiruvchi framework vositasi (NestJS, Angular).

### ❓ DRY ni haddan oshirib yuborish nima xavf tug'diradi?

**✅ Javob:** "Premature abstraction" — tasodifan o'xshash, lekin aslida turli sabablarga ko'ra o'zgaradigan kodlarni majburan birlashtirish. Bu coupling ni oshiradi: bir use-case o'zgarsa, birlashtirilgan abstraksiya buziladi va boshqa use-case lar zarar ko'radi. Qoida: "duplication is cheaper than the wrong abstraction".

### ❓ Duck typing nima va TypeScript bunga qanday aloqador?

**✅ Javob:** Duck typing — object ning tipi emas, balki strukturasi (qanday metod/xususiyatlari borligi) muhim. TypeScript "structural typing" ishlatadi: agar object kutilgan strukturaga mos kelsa, explicit `implements` shart emas. Bu nominal typing (Java/C#) dan farq qiladi, u yerda tip nomi muhim.

### ❓ Nega `new AbstractClass()` ishlamaydi?

**✅ Javob:** Abstract class to'liq emas — unda implementatsiyasiz `abstract` metodlar bo'lishi mumkin. Uni to'g'ridan-to'g'ri yaratish nomukammal object beradi. Avval konkret subclass orqali barcha abstract metodlarni implement qilish, keyin uni yaratish kerak.

---

## Masalalar

> Yechimlar: [01-oop-solid yechimlari](../solutions/oop-patterns/01-oop-solid.md)

1. **BankAccount encapsulation:** `private` balance li `BankAccount` class yozing. `deposit`, `withdraw` (mablag' yetmasa xato), va getter `balance` qo'shing. Manfiy summalarni rad eting.

2. **Shape abstraction:** `abstract class Shape` yarating (`area()` abstract, `describe()` konkret). `Circle`, `Rectangle`, `Triangle` subclasslarini yozing va massivda yuzalarini hisoblang.

3. **OCP — to'lov tizimi:** `if/switch` siz, har bir to'lov usuli (Card, PayPal, Crypto) uchun `PaymentMethod` interface ni implement qiluvchi alohida class yozing. Yangi usul qo'shganda eski kod o'zgarmasin.

4. **LSP tuzatish:** `Rectangle`/`Square` LSP buzilishini reproduksiya qiling, so'ng to'g'ri abstraksiya bilan tuzating.

5. **ISP refaktoring:** `work()`, `eat()`, `sleep()` li "fat" `Worker` interfeysni kichik interfeyslarga bo'ling. `Robot` va `Human` ni mos ravishda implement qiling.

6. **DIP — logger:** `Logger` interface yarating (`ConsoleLogger`, `FileLogger` implementatsiyalari). `OrderService` ni constructor injection orqali istalgan logger bilan ishlaydigan qiling.

7. **Composition — notifier:** Inheritance o'rniga composition ishlatib, `Notifier` class yozing. U `MessageChannel` (Email, SMS, Push) ni qabul qiladi va runtime da kanalni almashtirish mumkin bo'lsin.

8. **Polymorphism — figuralar maydoni:** `Shape[]` massivini qabul qilib, umumiy yuzani qaytaruvchi funksiya yozing. Yangi shakl qo'shilganda funksiya o'zgarmasligi kerak.

---

← [OOP bo'limiga qaytish](./README.md)
