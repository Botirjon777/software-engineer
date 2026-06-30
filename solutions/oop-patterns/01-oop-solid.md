# OOP va SOLID — Masalalar Yechimlari

Bu fayl `01-oop-solid.md` dagi mashqlarning to'liq yechimlarini izohlari bilan o'z ichiga oladi. Har bir yechim ishlaydigan TypeScript kodi va u nima uchun shunday yozilganligi haqida tushuntirishni beradi.

## Mundarija

- [1. BankAccount encapsulation](#1-bankaccount-encapsulation)
- [2. Shape abstraction](#2-shape-abstraction)
- [3. OCP — to'lov tizimi](#3-ocp--tolov-tizimi)
- [4. LSP tuzatish](#4-lsp-tuzatish)
- [5. ISP refaktoring](#5-isp-refaktoring)
- [6. DIP — logger](#6-dip--logger)
- [7. Composition — notifier](#7-composition--notifier)
- [8. Polymorphism — figuralar maydoni](#8-polymorphism--figuralar-maydoni)

---

## 1. BankAccount encapsulation

**Maqsad:** `private` field, validatsiya bilan `deposit`/`withdraw`, va getter orqali nazorat qilingan kirish.

```ts
class BankAccount {
  #balance: number; // haqiqiy runtime private (ECMAScript #)

  constructor(initial = 0) {
    if (initial < 0) throw new Error("Boshlang'ich balans manfiy bo'lmaydi");
    this.#balance = initial;
  }

  get balance(): number {
    return this.#balance; // faqat o'qish — tashqaridan yozib bo'lmaydi
  }

  deposit(amount: number): void {
    if (amount <= 0) throw new Error("Summa musbat bo'lishi kerak");
    this.#balance += amount;
  }

  withdraw(amount: number): void {
    if (amount <= 0) throw new Error("Summa musbat bo'lishi kerak");
    if (amount > this.#balance) throw new Error("Mablag' yetarli emas");
    this.#balance -= amount;
  }
}

const acc = new BankAccount(100);
acc.deposit(50);
acc.withdraw(30);
console.log(acc.balance); // 120
```

**Izoh:** `#balance` — ECMAScript private field, runtime da ham himoyalangan (TS `private` dan kuchliroq). Balans faqat metodlar orqali o'zgaradi, har bir o'zgarish validatsiyadan o'tadi. Bu encapsulation ning mohiyati: state himoyalangan va invariantlar (balans hech qachon manfiy bo'lmaydi) kafolatlangan.

---

## 2. Shape abstraction

**Maqsad:** `abstract` metod va konkret metod aralash bo'lgan abstract base class.

```ts
abstract class Shape {
  abstract area(): number; // subclass majburan implement qiladi

  describe(): string {
    // konkret — barcha subclasslarga umumiy
    return `${this.constructor.name} yuzasi: ${this.area().toFixed(2)}`;
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

class Rectangle extends Shape {
  constructor(private w: number, private h: number) {
    super();
  }
  area(): number {
    return this.w * this.h;
  }
}

class Triangle extends Shape {
  constructor(private base: number, private height: number) {
    super();
  }
  area(): number {
    return (this.base * this.height) / 2;
  }
}

const shapes: Shape[] = [new Circle(5), new Rectangle(4, 6), new Triangle(3, 8)];
shapes.forEach((s) => console.log(s.describe()));
const total = shapes.reduce((sum, s) => sum + s.area(), 0);
console.log(`Umumiy yuza: ${total.toFixed(2)}`);
```

**Izoh:** `Shape` — abstract class. `area()` abstract, ya'ni har bir subclass o'z formulasini berishga majbur. `describe()` esa konkret va meros olinadi (DRY). Massivni `Shape[]` tipida saqlash polymorphism ni ko'rsatadi — har bir `area()` chaqiruvi to'g'ri implementatsiyaga boradi.

---

## 3. OCP — to'lov tizimi

**Maqsad:** `if/switch` siz, yangi to'lov usuli qo'shganda eski kodni o'zgartirmaslik.

```ts
interface PaymentMethod {
  pay(amount: number): string;
}

class CardPayment implements PaymentMethod {
  pay(amount: number): string {
    return `Karta orqali ${amount} so'm to'landi`;
  }
}
class PayPalPayment implements PaymentMethod {
  pay(amount: number): string {
    return `PayPal orqali ${amount} so'm to'landi`;
  }
}
class CryptoPayment implements PaymentMethod {
  pay(amount: number): string {
    return `Crypto orqali ${amount} so'm to'landi`;
  }
}

// Bu class hech qachon o'zgarmaydi — OCP
class Checkout {
  process(method: PaymentMethod, amount: number): string {
    return method.pay(amount);
  }
}

const checkout = new Checkout();
console.log(checkout.process(new CardPayment(), 100000));
console.log(checkout.process(new CryptoPayment(), 250000));
// Yangi usul (masalan, ApplePay) qo'shilsa — faqat yangi class yoziladi,
// Checkout o'zgarmaydi.
```

**Izoh:** `Checkout` to'lov turlariga emas, `PaymentMethod` abstraksiyasiga bog'langan. Yangi usul = yangi class. `Checkout.process()` hech qachon o'zgarmaydi — kengaytirishga ochiq, o'zgartirishga yopiq. Bu aslida Strategy pattern ning OCP ko'rinishi.

---

## 4. LSP tuzatish

**Maqsad:** Klassik `Rectangle`/`Square` LSP buzilishini ko'rsatib, to'g'ri abstraksiya bilan tuzatish.

```ts
// ❌ MUAMMO: Square, Rectangle dan meros olsa — LSP buziladi
class BadRectangle {
  constructor(protected w: number, protected h: number) {}
  setWidth(w: number) {
    this.w = w;
  }
  setHeight(h: number) {
    this.h = h;
  }
  area() {
    return this.w * this.h;
  }
}
class BadSquare extends BadRectangle {
  setWidth(w: number) {
    this.w = w;
    this.h = w; // height ni ham o'zgartirishga majbur — shartnoma buzildi
  }
  setHeight(h: number) {
    this.w = h;
    this.h = h;
  }
}
// Rectangle kutgan kod: setWidth(5), setHeight(4) => area = 20
// BadSquare bilan: area = 16 — KUTILMAGAN NATIJA (LSP buzildi)

// ✅ YECHIM: umumiy abstraksiya, alohida tiplar
interface Shape {
  area(): number;
}
class Rectangle implements Shape {
  constructor(private w: number, private h: number) {}
  area(): number {
    return this.w * this.h;
  }
}
class Square implements Shape {
  constructor(private side: number) {}
  area(): number {
    return this.side ** 2;
  }
}

const shapes: Shape[] = [new Rectangle(5, 4), new Square(5)];
shapes.forEach((s) => console.log(s.area())); // 20, 25 — to'g'ri
```

**Izoh:** Muammo — Square "is-a" Rectangle deb taxmin qilish noto'g'ri, chunki Square setWidth/setHeight invariantlarini buzadi. Yechim: ikkalasini ham mustaqil `Shape` qilish (composition over inheritance ruhida). Endi har biri o'z immutable o'lchamlariga ega va LSP buzilmaydi.

---

## 5. ISP refaktoring

**Maqsad:** "Fat" interfeysni kichik, fokuslangan interfeyslarga bo'lish.

```ts
// ❌ Fat interface — Robot eat/sleep ga majbur bo'lardi
// interface Worker { work(): void; eat(): void; sleep(): void; }

// ✅ Bo'lingan interfeyslar
interface Workable {
  work(): void;
}
interface Eatable {
  eat(): void;
}
interface Sleepable {
  sleep(): void;
}

class Human implements Workable, Eatable, Sleepable {
  work() {
    console.log("Inson ishlamoqda");
  }
  eat() {
    console.log("Inson ovqatlanmoqda");
  }
  sleep() {
    console.log("Inson uxlamoqda");
  }
}

class Robot implements Workable {
  work() {
    console.log("Robot ishlamoqda"); // faqat kerakli interfeys
  }
}
```

**Izoh:** Endi `Robot` faqat `Workable` ni implement qiladi — keraksiz `eat`/`sleep` metodlarini bo'sh yoki exception bilan to'ldirishga majbur emas. Har bir client (kod) faqat o'ziga kerakli interfeysga bog'lanadi. Bu coupling ni pasaytiradi va kodni toza qiladi.

---

## 6. DIP — logger

**Maqsad:** Yuqori darajadagi `OrderService` ni abstraksiyaga bog'lab, dependency injection orqali istalgan logger bilan ishlatish.

```ts
interface Logger {
  log(message: string): void;
}

class ConsoleLogger implements Logger {
  log(message: string): void {
    console.log(`[console] ${message}`);
  }
}
class FileLogger implements Logger {
  log(message: string): void {
    // haqiqiy loyihada faylga yoziladi
    console.log(`[file] ${message}`);
  }
}

class OrderService {
  // OrderService konkret logger ga emas, Logger abstraksiyasiga bog'langan
  constructor(private logger: Logger) {}

  placeOrder(id: string): void {
    this.logger.log(`Buyurtma joylashtirildi: ${id}`);
  }
}

// Dependency tashqaridan beriladi (constructor injection)
const service1 = new OrderService(new ConsoleLogger());
const service2 = new OrderService(new FileLogger());
service1.placeOrder("A-100");
service2.placeOrder("B-200");
```

**Izoh:** `OrderService` (yuqori daraja) `ConsoleLogger`/`FileLogger` (past daraja) ga emas, `Logger` interfeysiga bog'langan — bu DIP. Dependency `new` bilan ichida yaratilmaydi, constructor orqali tashqaridan beriladi — bu Dependency Injection. Natijada logger ni almashtirish va test paytida mock qilish oson.

---

## 7. Composition — notifier

**Maqsad:** Inheritance o'rniga composition; runtime da kanalni almashtirish.

```ts
interface MessageChannel {
  send(message: string): void;
}

class EmailChannel implements MessageChannel {
  send(message: string): void {
    console.log(`Email: ${message}`);
  }
}
class SmsChannel implements MessageChannel {
  send(message: string): void {
    console.log(`SMS: ${message}`);
  }
}
class PushChannel implements MessageChannel {
  send(message: string): void {
    console.log(`Push: ${message}`);
  }
}

class Notifier {
  // has-a MessageChannel (composition), is-a emas
  constructor(private channel: MessageChannel) {}

  setChannel(channel: MessageChannel): void {
    this.channel = channel; // runtime da almashtirish
  }

  notify(message: string): void {
    this.channel.send(message);
  }
}

const notifier = new Notifier(new EmailChannel());
notifier.notify("Salom"); // Email: Salom
notifier.setChannel(new SmsChannel());
notifier.notify("Salom"); // SMS: Salom
```

**Izoh:** Agar inheritance ishlatganimizda (`EmailNotifier extends Notifier` va h.k.), runtime da kanalni almashtira olmas edik va har bir kanal uchun alohida subclass kerak bo'lardi. Composition bilan `Notifier` istalgan `MessageChannel` ni qabul qiladi va uni xohlagan vaqtda almashtiradi — moslashuvchan, low coupling.

---

## 8. Polymorphism — figuralar maydoni

**Maqsad:** Yangi shakl qo'shilganda o'zgarmaydigan, umumiy yuzani hisoblovchi funksiya.

```ts
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
class Triangle implements Shape {
  constructor(private b: number, private h: number) {}
  area() {
    return (this.b * this.h) / 2;
  }
}

// Bu funksiya hech qachon o'zgarmaydi — yangi shakl qo'shilsa ham
function totalArea(shapes: Shape[]): number {
  return shapes.reduce((sum, s) => sum + s.area(), 0);
}

console.log(totalArea([new Circle(2), new Square(3), new Triangle(4, 5)]));
```

**Izoh:** `totalArea` har bir element tipini tekshirmaydi (`if shape instanceof ...` yo'q) — u faqat `Shape` interfeysiga ishonadi va polymorphism ga tayanadi. Yangi shakl (masalan, `Pentagon`) qo'shilsa, faqat yangi class yoziladi va `totalArea` o'zgarmaydi. Bu polymorphism va OCP ning birgalikdagi kuchini ko'rsatadi.

---

← [OOP bo'limiga qaytish](../../oop-patterns/README.md)
