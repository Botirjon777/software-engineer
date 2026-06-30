# Design Patterns (GoF) — Masalalar Yechimlari

Bu fayl `02-design-patterns.md` dagi mashqlarning to'liq yechimlarini izohlari bilan o'z ichiga oladi. Har bir yechim ishlaydigan TypeScript implementatsiyasi va pattern ning mohiyatini tushuntiradi.

## Mundarija

- [1. Singleton Logger](#1-singleton-logger)
- [2. Factory — shakl yaratuvchi](#2-factory--shakl-yaratuvchi)
- [3. Builder — HTTP request](#3-builder--http-request)
- [4. Decorator — narx hisoblash](#4-decorator--narx-hisoblash)
- [5. Observer — yangiliklar](#5-observer--yangiliklar)
- [6. Strategy — to'lov](#6-strategy--tolov)
- [7. Adapter — temperatura](#7-adapter--temperatura)
- [8. State — buyurtma](#8-state--buyurtma)
- [9. Command — undo](#9-command--undo)
- [10. Composite — fayl tizimi](#10-composite--fayl-tizimi)

---

## 1. Singleton Logger

**Maqsad:** Yagona instance li Logger va nega DI afzal ekanini izohlash.

```ts
class Logger {
  private static instance: Logger;
  private logs: string[] = [];

  private constructor() {} // tashqaridan new yo'q

  static getInstance(): Logger {
    if (!Logger.instance) {
      Logger.instance = new Logger();
    }
    return Logger.instance;
  }

  log(message: string): void {
    this.logs.push(message);
    console.log(`[LOG] ${message}`);
  }

  getHistory(): string[] {
    return [...this.logs];
  }
}

const a = Logger.getInstance();
const b = Logger.getInstance();
a.log("Birinchi");
console.log(a === b); // true — bir xil instance
console.log(b.getHistory()); // ["Birinchi"]
```

**Nega DI afzal:** Singleton yashirin global state yaratadi — `Logger.getInstance()` ni chaqirgan har bir class unga qattiq bog'lanadi (DIP buziladi) va test paytida soxta (mock) logger berib bo'lmaydi. DI bilan logger ni `Logger` interfeysi sifatida constructor orqali bersak, test va almashtirish oson bo'ladi:

```ts
interface ILogger {
  log(m: string): void;
}
class Service {
  constructor(private logger: ILogger) {} // mock berish oson
}
```

---

## 2. Factory — shakl yaratuvchi

**Maqsad:** Tip nomidan mos `Shape` yaratuvchi factory; kengaytirish oson.

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

type ShapeType = "circle" | "square";

// Registry asosidagi factory — yangi shakl qo'shish oson (OCP)
class ShapeFactory {
  private registry: Record<ShapeType, (size: number) => Shape> = {
    circle: (size) => new Circle(size),
    square: (size) => new Square(size),
  };

  create(type: ShapeType, size: number): Shape {
    const creator = this.registry[type];
    if (!creator) throw new Error(`Noma'lum shakl: ${type}`);
    return creator(size);
  }
}

const factory = new ShapeFactory();
console.log(factory.create("circle", 5).area());
console.log(factory.create("square", 4).area());
```

**Izoh:** Client (`factory.create(...)`) konkret class larni (`Circle`, `Square`) bilmaydi — faqat tip nomini beradi. Registry yondashuvi yangi shakl qo'shishni soddalashtiradi: registry ga bitta qator qo'shiladi, `create` metodi o'zgarmaydi (OCP).

---

## 3. Builder — HTTP request

**Maqsad:** Fluent (method chaining) RequestBuilder.

```ts
interface HttpRequest {
  url: string;
  method: string;
  headers: Record<string, string>;
  body?: string;
}

class RequestBuilder {
  private request: HttpRequest = {
    url: "",
    method: "GET",
    headers: {},
  };

  setUrl(url: string): this {
    this.request.url = url;
    return this;
  }
  setMethod(method: string): this {
    this.request.method = method;
    return this;
  }
  addHeader(key: string, value: string): this {
    this.request.headers[key] = value;
    return this;
  }
  setBody(body: string): this {
    this.request.body = body;
    return this;
  }
  build(): HttpRequest {
    if (!this.request.url) throw new Error("URL majburiy");
    return this.request;
  }
}

const req = new RequestBuilder()
  .setUrl("https://api.example.com/users")
  .setMethod("POST")
  .addHeader("Content-Type", "application/json")
  .setBody(JSON.stringify({ name: "Ali" }))
  .build();

console.log(req);
```

**Izoh:** Har bir setter `this` qaytaradi — bu fluent interface (method chaining) ni ta'minlaydi. Builder "telescoping constructor" muammosini hal qiladi: ko'p ixtiyoriy parametrli object ni o'qiladigan tarzda, bosqichma-bosqich quradi. `build()` da validatsiya qilinadi.

---

## 4. Decorator — narx hisoblash

**Maqsad:** Runtime da qatlamlanadigan qahva decorator lari.

```ts
interface Coffee {
  cost(): number;
  description(): string;
}

class SimpleCoffee implements Coffee {
  cost() {
    return 10;
  }
  description() {
    return "Qahva";
  }
}

abstract class CoffeeDecorator implements Coffee {
  constructor(protected coffee: Coffee) {}
  abstract cost(): number;
  abstract description(): string;
}

class MilkDecorator extends CoffeeDecorator {
  cost() {
    return this.coffee.cost() + 3;
  }
  description() {
    return this.coffee.description() + " + sut";
  }
}
class SugarDecorator extends CoffeeDecorator {
  cost() {
    return this.coffee.cost() + 1;
  }
  description() {
    return this.coffee.description() + " + shakar";
  }
}
class WhipDecorator extends CoffeeDecorator {
  cost() {
    return this.coffee.cost() + 5;
  }
  description() {
    return this.coffee.description() + " + qaymoq";
  }
}

let coffee: Coffee = new SimpleCoffee();
coffee = new MilkDecorator(coffee);
coffee = new SugarDecorator(coffee);
coffee = new WhipDecorator(coffee);

console.log(coffee.description()); // Qahva + sut + shakar + qaymoq
console.log(coffee.cost()); // 19
```

**Izoh:** Har bir decorator `Coffee` ni o'rab (wrap) oladi va o'z hissasini qo'shadi. Qatlamlash runtime da, istalgan tartibda va kombinatsiyada bo'ladi. Inheritance bilan har bir kombinatsiya (Qahva+Sut+Shakar, Qahva+Sut+Qaymoq, ...) uchun alohida class kerak bo'lardi — decorator buni "kombinatsiya portlashi" muammosini hal qilib bartaraf etadi.

---

## 5. Observer — yangiliklar

**Maqsad:** Subject/Observer, unsubscribe va memory leak dan himoya.

```ts
interface Observer {
  update(news: string): void;
}

class NewsAgency {
  private observers = new Set<Observer>(); // Set — dublikat va o'chirish qulay

  subscribe(o: Observer): void {
    this.observers.add(o);
  }
  unsubscribe(o: Observer): void {
    this.observers.delete(o); // memory leak oldini olish
  }
  publish(news: string): void {
    this.observers.forEach((o) => o.update(news));
  }
}

class EmailSubscriber implements Observer {
  constructor(private name: string) {}
  update(news: string): void {
    console.log(`${this.name} email oldi: ${news}`);
  }
}

const agency = new NewsAgency();
const ali = new EmailSubscriber("Ali");
const vali = new EmailSubscriber("Vali");

agency.subscribe(ali);
agency.subscribe(vali);
agency.publish("Birinchi xabar"); // Ali va Vali oladi

agency.unsubscribe(ali);
agency.publish("Ikkinchi xabar"); // faqat Vali oladi
```

**Izoh:** `NewsAgency` (subject) holati o'zgarganda barcha obunachilarga `update()` orqali xabar beradi — bu Observer/pub-sub. `Set` ishlatish dublikatlardan va `unsubscribe` da qulaylikdan foydalanadi. **Memory leak xavfi:** agar obunachi `unsubscribe` qilinmasa, subject uni reference qilib ushlab turadi va garbage collector uni tozalay olmaydi. Shuning uchun obunani bekor qilish muhim.

---

## 6. Strategy — to'lov

**Maqsad:** Runtime da almashtiriladigan to'lov strategiyalari.

```ts
interface PaymentStrategy {
  pay(amount: number): string;
}

class CardStrategy implements PaymentStrategy {
  pay(amount: number) {
    return `Karta: ${amount} so'm`;
  }
}
class PayPalStrategy implements PaymentStrategy {
  pay(amount: number) {
    return `PayPal: ${amount} so'm`;
  }
}
class CryptoStrategy implements PaymentStrategy {
  pay(amount: number) {
    return `Crypto: ${amount} so'm`;
  }
}

class Checkout {
  constructor(private strategy: PaymentStrategy) {}

  setStrategy(strategy: PaymentStrategy): void {
    this.strategy = strategy; // runtime da almashtirish
  }

  pay(amount: number): void {
    console.log(this.strategy.pay(amount));
  }
}

const checkout = new Checkout(new CardStrategy());
checkout.pay(50000); // Karta: 50000 so'm
checkout.setStrategy(new CryptoStrategy());
checkout.pay(50000); // Crypto: 50000 so'm
```

**Izoh:** Har bir to'lov usuli alohida strategiya class i. `Checkout` `if/else` bilan emas, balki abstraksiya (`PaymentStrategy`) orqali ishlaydi va strategiyani runtime da almashtira oladi. Yangi usul qo'shilsa — faqat yangi class, `Checkout` o'zgarmaydi (OCP). Bu Strategy va OCP ning bog'liqligini ko'rsatadi.

---

## 7. Adapter — temperatura

**Maqsad:** Fahrenheit beruvchi eski API ni Celsius kutadigan tizimga moslash.

```ts
// Eski/uchinchi tomon API — faqat Fahrenheit beradi
class LegacyWeatherApi {
  getTempFahrenheit(): number {
    return 98.6;
  }
}

// Bizning yangi tizim shu interfeysni kutadi
interface CelsiusWeather {
  getTempCelsius(): number;
}

// Adapter — Fahrenheit ni Celsius ga "tarjima" qiladi
class WeatherAdapter implements CelsiusWeather {
  constructor(private legacy: LegacyWeatherApi) {}

  getTempCelsius(): number {
    const f = this.legacy.getTempFahrenheit();
    return Math.round(((f - 32) * 5) / 9 * 10) / 10; // F -> C
  }
}

const adapter = new WeatherAdapter(new LegacyWeatherApi());
console.log(adapter.getTempCelsius()); // 37
```

**Izoh:** `LegacyWeatherApi` ni o'zgartira olmaymiz (uchinchi tomon kodi). Adapter uning `getTempFahrenheit()` chiqishini bizning tizim kutadigan `getTempCelsius()` ga moslaydi. Bu Adapter ning mohiyati: ikkita mos kelmaydigan interfeys orasida "tarjimon" bo'lish.

---

## 8. State — buyurtma

**Maqsad:** Order workflow ni State pattern bilan modellashtirish.

```ts
interface OrderState {
  next(order: Order): void;
  status(): string;
}

class PendingState implements OrderState {
  next(order: Order) {
    order.setState(new ShippedState());
  }
  status() {
    return "Kutilmoqda";
  }
}
class ShippedState implements OrderState {
  next(order: Order) {
    order.setState(new DeliveredState());
  }
  status() {
    return "Yuborildi";
  }
}
class DeliveredState implements OrderState {
  next() {
    console.log("Buyurtma allaqachon yetkazilgan");
  }
  status() {
    return "Yetkazildi";
  }
}

class Order {
  private state: OrderState = new PendingState();
  setState(state: OrderState): void {
    this.state = state;
  }
  next(): void {
    this.state.next(this);
  }
  status(): string {
    return this.state.status();
  }
}

const order = new Order();
console.log(order.status()); // Kutilmoqda
order.next();
console.log(order.status()); // Yuborildi
order.next();
console.log(order.status()); // Yetkazildi
```

**Izoh:** Har bir holat alohida class va keyingi holatga o'tishni o'zi biladi. `Order` da `if (status === ...)` bloklari yo'q — bu State pattern ning afzalligi: yangi holat qo'shish yoki o'tishlarni o'zgartirish lokal qoladi va katta `switch` larsiz amalga oshadi.

---

## 9. Command — undo

**Maqsad:** execute/undo li matn editori.

```ts
interface Command {
  execute(): void;
  undo(): void;
}

class TextEditor {
  content = "";
}

class AddTextCommand implements Command {
  constructor(private editor: TextEditor, private text: string) {}
  execute(): void {
    this.editor.content += this.text;
  }
  undo(): void {
    this.editor.content = this.editor.content.slice(0, -this.text.length);
  }
}

class CommandManager {
  private history: Command[] = [];
  run(cmd: Command): void {
    cmd.execute();
    this.history.push(cmd);
  }
  undo(): void {
    this.history.pop()?.undo();
  }
}

const editor = new TextEditor();
const manager = new CommandManager();

manager.run(new AddTextCommand(editor, "Salom "));
manager.run(new AddTextCommand(editor, "Dunyo"));
console.log(editor.content); // "Salom Dunyo"
manager.undo();
console.log(editor.content); // "Salom "
```

**Izoh:** Har bir amal (`AddTextCommand`) `execute()` va `undo()` ga ega object. `CommandManager` bajarilgan command larni history stack da saqlaydi va `undo()` da oxirgisini orqaga qaytaradi. Command pattern ayni shu undo/redo, queue va logging stsenariylari uchun ideal.

---

## 10. Composite — fayl tizimi

**Maqsad:** Leaf (File) va Composite (Folder) ni bir xil interfeys bilan ishlatish.

```ts
interface FileSystemNode {
  name: string;
  size(): number;
}

class FileLeaf implements FileSystemNode {
  constructor(public name: string, private bytes: number) {}
  size(): number {
    return this.bytes;
  }
}

class Folder implements FileSystemNode {
  private children: FileSystemNode[] = [];
  constructor(public name: string) {}

  add(node: FileSystemNode): void {
    this.children.push(node);
  }

  size(): number {
    // rekursiv — har bir bola File yoki Folder bo'lishi mumkin
    return this.children.reduce((sum, child) => sum + child.size(), 0);
  }
}

const root = new Folder("root");
root.add(new FileLeaf("a.txt", 100));

const sub = new Folder("docs");
sub.add(new FileLeaf("b.txt", 50));
sub.add(new FileLeaf("c.txt", 25));
root.add(sub);

console.log(root.size()); // 175 (100 + 50 + 25)
```

**Izoh:** `FileLeaf` (yagona element) va `Folder` (guruh) bir xil `FileSystemNode` interfeysini implement qiladi. Shuning uchun `size()` ni chaqirganda File yoki Folder ekanini farqlash shart emas — `Folder.size()` rekursiv ravishda barcha bolalarni yig'adi. Bu Composite ning kuchi: daraxtsimon strukturani yagona interfeys orqali bir xil ishlatish.

---

← [OOP bo'limiga qaytish](../../oop-patterns/README.md)
