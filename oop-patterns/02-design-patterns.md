# Design Patterns (GoF)

Design Pattern — bu dasturlashda tez-tez uchraydigan muammolarga yillar davomida sinab ko'rilgan, qayta ishlatiladigan yechim shabloni. Ushbu hujjat "Gang of Four" (GoF) ning klassik 23 patternidan eng muhimlarini TypeScript misollari, real use-case lar va "qachon ishlatmaslik" tavsiyalari bilan tushuntiradi.

Pattern lar — bu tayyor kod emas, balki **g'oya** va **muloqot tili**. "Bu yerda Observer ishlatamiz" deyish — bir jumlada butun arxitektura qarorini yetkazadi. Intervyularda eng ko'p so'raladiganlar: **Singleton, Factory, Observer, Strategy, Decorator**.

## Mundarija

- [Design Pattern nima va nega](#design-pattern-nima-va-nega)
- [3 kategoriya](#3-kategoriya)
- [Creational: Singleton](#singleton)
- [Creational: Factory Method](#factory-method)
- [Creational: Abstract Factory](#abstract-factory)
- [Creational: Builder](#builder)
- [Creational: Prototype](#prototype)
- [Structural: Adapter](#adapter)
- [Structural: Decorator](#decorator)
- [Structural: Facade](#facade)
- [Structural: Proxy](#proxy)
- [Structural: Composite](#composite)
- [Behavioral: Observer](#observer)
- [Behavioral: Strategy](#strategy)
- [Behavioral: Command](#command)
- [Behavioral: Iterator](#iterator)
- [Behavioral: State](#state)
- [Behavioral: Template Method](#template-method)
- [Anti-pattern eslatma](#anti-pattern-eslatma)
- [Savol-Javoblar](#savol-javoblar)
- [Masalalar](#masalalar)

---

## Design Pattern nima va nega

**💡 Tushuncha:** Design pattern — umumiy dizayn muammolariga sinangan, takrorlanuvchi yechim. U "qaytadan g'ildirak ixtiro qilish" o'rniga, jamoaviy tajribadan foydalanish imkonini beradi.

Nega kerak:
- **Sinab ko'rilgan yechim:** millionlab dasturchilar tomonidan tekshirilgan.
- **Umumiy lug'at:** "Factory ishlat" — bir og'iz so'z bilan tushunarli.
- **Maintainability:** standart strukturalar yangi a'zolarga tanish.

**⚠️ Ehtiyot bo'l:** Pattern — bu maqsad emas, vosita. Muammo bo'lmasa pattern qo'shish "over-engineering". Avval muammoni aniqlang, keyin pattern tanlang — teskari emas.

---

## 3 kategoriya

GoF pattern larini uch guruhga ajratadi:

1. **Creational (yaratuvchi):** object yaratish jarayonini boshqaradi. *Singleton, Factory Method, Abstract Factory, Builder, Prototype.*
2. **Structural (strukturaviy):** object va class larni kattaroq strukturalarga birlashtiradi. *Adapter, Decorator, Facade, Proxy, Composite.*
3. **Behavioral (xulq-atvor):** object lar orasidagi muloqot va mas'uliyat taqsimotini boshqaradi. *Observer, Strategy, Command, Iterator, State, Template Method.*

---

## Singleton

**Muammo:** Butun dastur davomida bir class ning faqat bitta nusxasi (instance) bo'lishi va unga global kirish kerak (masalan, config, logger, DB connection pool).

**Implementatsiya:**

```ts
class Database {
  private static instance: Database;
  private constructor() {} // tashqaridan new bilan yaratib bo'lmaydi

  static getInstance(): Database {
    if (!Database.instance) {
      Database.instance = new Database();
    }
    return Database.instance;
  }

  query(sql: string): void {
    console.log(`Bajarilmoqda: ${sql}`);
  }
}

const db1 = Database.getInstance();
const db2 = Database.getInstance();
console.log(db1 === db2); // true — bir xil instance
```

**Real use-case:** Logger, konfiguratsiya menejeri, connection pool, cache.

**Tuzoqlari (qachon ishlatmaslik):**
- **Global state:** Singleton — yashirin global o'zgaruvchi. Test qilishni qiyinlashtiradi (mock qilish qiyin).
- **Tight coupling:** kodning ko'p qismi bevosita `getInstance()` ga bog'lanib qoladi (DIP buziladi).
- **Ko'pincha anti-pattern** deb hisoblanadi. Zamonaviy yondashuv: Singleton o'rniga **Dependency Injection container** ga bir martalik (singleton-scoped) registratsiya qilish.

**⚠️ Ehtiyot bo'l:** Singleton ni cheklab ishlating. NestJS/Angular kabi framework larda DI container allaqachon singleton scope beradi — o'z qo'lingiz bilan Singleton yozish kamdan-kam kerak.

---

## Factory Method

**Muammo:** Object yaratish logikasini izolyatsiya qilish kerak — qaysi konkret class yaratilishini client bilmasligi kerak.

**Implementatsiya:**

```ts
interface Transport {
  deliver(): string;
}
class Truck implements Transport {
  deliver() {
    return "Quruqlikda yetkazildi";
  }
}
class Ship implements Transport {
  deliver() {
    return "Dengizda yetkazildi";
  }
}

abstract class Logistics {
  abstract createTransport(): Transport; // factory method
  planDelivery(): string {
    const transport = this.createTransport();
    return transport.deliver();
  }
}
class RoadLogistics extends Logistics {
  createTransport() {
    return new Truck();
  }
}
class SeaLogistics extends Logistics {
  createTransport() {
    return new Ship();
  }
}
```

**Real use-case:** Turli xil document parser lar, ulanish turlari (DB driver), UI element yaratish.

**Qachon ishlatmaslik:** Agar faqat bitta turdagi object bo'lsa va kelajakda kengaytirish kutilmasa — oddiy `new` yetarli. Factory qo'shish keraksiz murakkablik.

---

## Abstract Factory

**Muammo:** O'zaro bog'liq object lar **oilasini** (family) yaratish kerak, ularning konkret class larini bilmasdan. Masalan, UI uchun "Windows tugmasi + Windows oynasi" yoki "Mac tugmasi + Mac oynasi".

```ts
interface Button {
  render(): string;
}
interface Checkbox {
  render(): string;
}

interface GUIFactory {
  createButton(): Button;
  createCheckbox(): Checkbox;
}

class WindowsFactory implements GUIFactory {
  createButton(): Button {
    return { render: () => "Windows tugma" };
  }
  createCheckbox(): Checkbox {
    return { render: () => "Windows checkbox" };
  }
}
class MacFactory implements GUIFactory {
  createButton(): Button {
    return { render: () => "Mac tugma" };
  }
  createCheckbox(): Checkbox {
    return { render: () => "Mac checkbox" };
  }
}
```

**Real use-case:** Cross-platform UI kutubxonalar, turli ma'lumotlar bazasi providerlari uchun bog'liq object lar to'plami.

**Farqi:** Factory Method — bitta object yaratadi. Abstract Factory — o'zaro mos object lar oilasini yaratadi.

---

## Builder

**Muammo:** Murakkab object ni bosqichma-bosqich qurish kerak, ko'p ixtiyoriy parametrlar bilan (constructor "telescoping" muammosini hal qiladi).

```ts
class Burger {
  size = "";
  cheese = false;
  bacon = false;
}

class BurgerBuilder {
  private burger = new Burger();
  setSize(size: string): this {
    this.burger.size = size;
    return this; // fluent interface (chaining)
  }
  addCheese(): this {
    this.burger.cheese = true;
    return this;
  }
  addBacon(): this {
    this.burger.bacon = true;
    return this;
  }
  build(): Burger {
    return this.burger;
  }
}

const burger = new BurgerBuilder()
  .setSize("Katta")
  .addCheese()
  .addBacon()
  .build();
```

**Real use-case:** Query builder lar (SQL/ORM), HTTP request builder, murakkab konfiguratsiya object lari.

**Qachon ishlatmaslik:** Object oddiy va 2-3 parametrli bo'lsa — Builder ortiqcha. Oddiy constructor yoki object literal yetarli.

---

## Prototype

**Muammo:** Mavjud object dan nusxa (clone) olish kerak, uni noldan yaratish qimmat yoki murakkab bo'lganda.

```ts
interface Cloneable<T> {
  clone(): T;
}
class Document implements Cloneable<Document> {
  constructor(public title: string, public tags: string[]) {}
  clone(): Document {
    // deep copy — tags massivini ham nusxalash
    return new Document(this.title, [...this.tags]);
  }
}

const original = new Document("Shartnoma", ["muhim"]);
const copy = original.clone();
copy.tags.push("nusxa"); // original ga ta'sir qilmaydi
```

**Real use-case:** Murakkab konfiguratsiya object larini nusxalash, game obyektlarini ko'paytirish, default template lar.

**⚠️ Ehtiyot bo'l:** Shallow copy vs deep copy. Object ichidagi massiv/nested object lar reference orqali ulashilib qolmasligi uchun deep copy kerak.

---

## Adapter

**Muammo:** Bir-biriga mos kelmaydigan ikkita interfeysni birlashtirish kerak. Eski/uchinchi tomon kodini o'zgartirmasdan ishlatish.

```ts
// Bizning tizim shu interfeysni kutadi
interface JsonLogger {
  log(data: object): void;
}

// Uchinchi tomon kutubxonasi — boshqa interfeys
class XmlLibrary {
  writeXml(xml: string): void {
    console.log(xml);
  }
}

// Adapter — XmlLibrary ni JsonLogger ga moslaydi
class XmlLoggerAdapter implements JsonLogger {
  constructor(private xmlLib: XmlLibrary) {}
  log(data: object): void {
    const xml = `<log>${JSON.stringify(data)}</log>`;
    this.xmlLib.writeXml(xml);
  }
}
```

**Real use-case:** Legacy tizim integratsiyasi, uchinchi tomon API/SDK larini umumiy interfeysga keltirish.

---

## Decorator

**Muammo:** Object ga yangi xatti-harakatni **dinamik** ravishda, subclass yaratmasdan qo'shish kerak. Inheritance "kombinatsiya portlashi" muammosini hal qiladi.

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

// Asosiy decorator
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

// Dinamik ravishda qatlamlash
let coffee: Coffee = new SimpleCoffee();
coffee = new MilkDecorator(coffee);
coffee = new SugarDecorator(coffee);
console.log(coffee.description(), coffee.cost()); // "Qahva + sut + shakar" 14
```

**Real use-case:** HTTP middleware (request/response qatlamlari), stream o'rash (compression, encryption), UI komponent bezatish, TypeScript/Angular decorator lar.

**Qachon ishlatmaslik:** Kombinatsiyalar oz va sobit bo'lsa — oddiy subclass yoki konfiguratsiya yetarli.

---

## Facade

**Muammo:** Murakkab subsystem (ko'p class) ustiga oddiy, yagona interfeys berish. Client murakkablikni bilmasligi kerak.

```ts
class CPU {
  freeze() {}
  jump(pos: number) {}
  execute() {}
}
class Memory {
  load(pos: number, data: string) {}
}
class HardDrive {
  read(): string {
    return "boot data";
  }
}

// Facade — uch subsystemni bitta oddiy metodga yashiradi
class ComputerFacade {
  private cpu = new CPU();
  private memory = new Memory();
  private hd = new HardDrive();

  start(): void {
    this.cpu.freeze();
    this.memory.load(0, this.hd.read());
    this.cpu.jump(0);
    this.cpu.execute();
  }
}

new ComputerFacade().start(); // client faqat shuni biladi
```

**Real use-case:** SDK/kutubxona uchun "soddalashtirilgan API", murakkab biznes-jarayonni bitta service metodiga o'rash.

---

## Proxy

**Muammo:** Real object ga kirishni nazorat qilish kerak — lazy loading, kesh, kirish huquqi (access control) yoki logging uchun.

```ts
interface Image {
  display(): void;
}
class RealImage implements Image {
  constructor(private file: string) {
    console.log(`${file} diskdan yuklanmoqda...`); // qimmat operatsiya
  }
  display() {
    console.log(`${this.file} ko'rsatilmoqda`);
  }
}

// Proxy — real object ni faqat kerak bo'lganda yaratadi (lazy)
class ImageProxy implements Image {
  private real: RealImage | null = null;
  constructor(private file: string) {}
  display() {
    if (!this.real) this.real = new RealImage(this.file); // lazy init
    this.real.display();
  }
}

const img = new ImageProxy("rasm.png"); // hali yuklanmadi
img.display(); // endi yuklanadi va ko'rsatiladi
```

**Real use-case:** Lazy loading, caching proxy, access control proxy, virtual proxy, JavaScript ning `Proxy` object i.

---

## Composite

**Muammo:** Object larni daraxtsimon ierarxiyada tashkil qilish va yagona (leaf) hamda guruh (composite) object lar bilan bir xil ishlash kerak.

```ts
interface FileSystemNode {
  size(): number;
}
class FileLeaf implements FileSystemNode {
  constructor(private bytes: number) {}
  size() {
    return this.bytes;
  }
}
class Folder implements FileSystemNode {
  private children: FileSystemNode[] = [];
  add(node: FileSystemNode): void {
    this.children.push(node);
  }
  size(): number {
    return this.children.reduce((sum, c) => sum + c.size(), 0); // rekursiv
  }
}

const root = new Folder();
root.add(new FileLeaf(100));
const sub = new Folder();
sub.add(new FileLeaf(50));
root.add(sub);
console.log(root.size()); // 150
```

**Real use-case:** Fayl tizimlari, DOM/UI komponent daraxti, menyu strukturalari, tashkiliy ierarxiyalar.

---

## Observer

**Muammo:** Bir object (subject) holati o'zgarganda, unga bog'liq ko'p object lar (observers) avtomatik xabardor bo'lishi kerak — ularni qattiq bog'lamasdan.

**Bu — eng muhim behavioral pattern.** U **pub/sub (publish-subscribe)** g'oyasiga asoslanadi: nashr qiluvchi (publisher) eventni e'lon qiladi, obunachilar (subscribers) reaksiya bildiradi. Event-driven arxitektura, RxJS, DOM event listener lar — barchasi shu g'oyaga asoslangan.

```ts
interface Observer {
  update(data: string): void;
}

class Subject {
  private observers: Observer[] = [];
  subscribe(o: Observer): void {
    this.observers.push(o);
  }
  unsubscribe(o: Observer): void {
    this.observers = this.observers.filter((x) => x !== o);
  }
  notify(data: string): void {
    this.observers.forEach((o) => o.update(data)); // hammaga xabar
  }
}

class EmailSubscriber implements Observer {
  update(data: string): void {
    console.log(`Email: ${data}`);
  }
}

const news = new Subject();
const sub = new EmailSubscriber();
news.subscribe(sub);
news.notify("Yangi maqola chiqdi!"); // "Email: Yangi maqola chiqdi!"
```

**Real use-case:** Event systems, UI data binding, RxJS Observable lar, DOM addEventListener, state management (Redux/store), WebSocket notifications.

**Pub/Sub bilan bog'liqlik:** Klassik Observer da subject va observer bevosita tanish. Pub/Sub da ular orasida **message broker/event bus** turadi — bu coupling ni yanada pasaytiradi.

**⚠️ Ehtiyot bo'l:** Observer larni `unsubscribe` qilishni unutmang — aks holda memory leak yuzaga keladi (observer lar yig'ilib qoladi).

---

## Strategy

**Muammo:** Bir oilaga oid algoritmlarni ajratish va ularni runtime da almashtirish kerak. `if/else` blokini polymorphism bilan almashtirish.

```ts
interface SortStrategy {
  sort(data: number[]): number[];
}
class QuickSort implements SortStrategy {
  sort(data: number[]) {
    return [...data].sort((a, b) => a - b); // soddalashtirilgan
  }
}
class BubbleSort implements SortStrategy {
  sort(data: number[]) {
    return [...data].sort((a, b) => a - b);
  }
}

class Sorter {
  constructor(private strategy: SortStrategy) {}
  setStrategy(s: SortStrategy): void {
    this.strategy = s; // runtime da almashtirish
  }
  run(data: number[]): number[] {
    return this.strategy.sort(data);
  }
}

const sorter = new Sorter(new QuickSort());
sorter.run([3, 1, 2]);
sorter.setStrategy(new BubbleSort()); // strategiyani almashtirdik
```

**Real use-case:** To'lov usullari, saralash/siqish algoritmlari, validatsiya qoidalari, narx hisoblash (chegirma strategiyalari).

**Qachon ishlatmaslik:** Faqat bir-ikki variant bo'lsa va ular o'zgarmasa — oddiy `if` yetarli. Strategy faqat algoritmlar oilasi va ularning almashinuvi bo'lsa ma'noli.

---

## Command

**Muammo:** So'rovni (request) alohida object sifatida o'rash kerak — buni navbatga qo'yish, log qilish yoki bekor qilish (undo) uchun.

```ts
interface Command {
  execute(): void;
  undo(): void;
}
class Light {
  on() {
    console.log("Chiroq yoqildi");
  }
  off() {
    console.log("Chiroq o'chirildi");
  }
}
class LightOnCommand implements Command {
  constructor(private light: Light) {}
  execute() {
    this.light.on();
  }
  undo() {
    this.light.off();
  }
}

class RemoteControl {
  private history: Command[] = [];
  submit(cmd: Command): void {
    cmd.execute();
    this.history.push(cmd);
  }
  undoLast(): void {
    this.history.pop()?.undo();
  }
}
```

**Real use-case:** Undo/redo (editor lar), task queue/job scheduler, transaction lar, menu/button action lar.

---

## Iterator

**Muammo:** Kollektsiya elementlarini, uning ichki strukturasini ochmasdan, ketma-ket aylanib chiqish.

```ts
class NumberCollection {
  private items: number[] = [];
  add(n: number): void {
    this.items.push(n);
  }
  // JS ning standart iterator protokoli
  [Symbol.iterator](): Iterator<number> {
    let index = 0;
    const items = this.items;
    return {
      next(): IteratorResult<number> {
        return index < items.length
          ? { value: items[index++], done: false }
          : { value: undefined, done: true };
      },
    };
  }
}

const col = new NumberCollection();
col.add(1);
col.add(2);
for (const n of col) console.log(n); // 1, 2 — for...of ishlaydi
```

**Real use-case:** Custom kollektsiyalar, daraxt/graf traversal, pagination, lazy sequence lar (generator lar).

---

## State

**Muammo:** Object o'z ichki holatiga qarab xatti-harakatini o'zgartirishi kerak — katta `if/switch` bloklarisiz. "Object xuddi class ini o'zgartirgandek ko'rinadi".

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
  next() {}
  status() {
    return "Yetkazildi";
  }
}

class Order {
  private state: OrderState = new PendingState();
  setState(s: OrderState): void {
    this.state = s;
  }
  next(): void {
    this.state.next(this);
  }
  status(): string {
    return this.state.status();
  }
}
```

**Real use-case:** Order/document workflow, UI komponent holatlari, game character holatlari, TCP connection holatlari.

**Farqi (Strategy bilan):** Strategy — client algoritmni tanlaydi. State — object o'zi holatini avtomatik almashtiradi.

---

## Template Method

**Muammo:** Algoritmning umumiy skeletini parent class da belgilab, ayrim qadamlarni subclass larga qoldirish.

```ts
abstract class DataProcessor {
  // template method — skelet sobit
  process(): void {
    this.read();
    this.transform();
    this.save();
  }
  protected abstract read(): void;
  protected abstract transform(): void;
  protected save(): void {
    console.log("Standart saqlash");
  }
}

class CsvProcessor extends DataProcessor {
  protected read() {
    console.log("CSV o'qildi");
  }
  protected transform() {
    console.log("CSV o'zgartirildi");
  }
}
```

**Real use-case:** Framework hook lar (lifecycle metodlari), ETL pipeline, test setup/teardown, build jarayonlari.

**Farqi (Strategy bilan):** Template Method — inheritance orqali (compile-time). Strategy — composition orqali (runtime, moslashuvchanroq).

---

## Anti-pattern eslatma

**⚠️ Ehtiyot bo'l:** Pattern lar noto'g'ri ishlatilsa zararli bo'ladi:

- **God Object:** hamma narsani qiluvchi ulkan class (SRP buzilishi).
- **Singleton abuse:** hamma joyda Singleton — yashirin global state, test qiyin.
- **Over-engineering:** muammo yo'q joyga pattern qo'shish — keraksiz murakkablik.
- **Premature pattern:** kelajakda kerak bo'lar deb pattern qo'shish (YAGNI buzilishi).

Oltin qoida: **Pattern muammoni ergashishi kerak, muammo pattern ni emas.** Avval oddiy yechim yozing; murakkablik real bo'lganda pattern ga refaktoring qiling.

---

## Savol-Javoblar

### ❓ Design pattern nima va nega kerak?

**✅ Javob:** Bu tez-tez uchraydigan dizayn muammolariga sinangan, qayta ishlatiladigan yechim shabloni. U vaqtni tejaydi (g'ildirakni qayta ixtiro qilmaslik), umumiy lug'at beradi (jamoaviy muloqot) va maintainable, tanish strukturalar yaratadi.

### ❓ GoF pattern larining 3 kategoriyasi qaysilar?

**✅ Javob:** **Creational** (object yaratish — Singleton, Factory, Builder), **Structural** (object larni birlashtirish — Adapter, Decorator, Facade), **Behavioral** (object lar muloqoti — Observer, Strategy, Command).

### ❓ Singleton nima va nega ko'pincha anti-pattern deyiladi?

**✅ Javob:** Singleton — class ning faqat bitta instance ini ta'minlovchi pattern. Anti-pattern deyilishining sababi: u yashirin global state yaratadi, tight coupling keltiradi (DIP buziladi) va test qilishni qiyinlashtiradi (mock qilish qiyin). Zamonaviy alternativa — DI container da singleton scope.

### ❓ Factory Method va Abstract Factory farqi?

**✅ Javob:** Factory Method — bitta object yaratish uchun, subclass qaysi konkret tipni yaratishini hal qiladi. Abstract Factory — o'zaro bog'liq object lar **oilasini** yaratadi (masalan, butun UI to'plami). Abstract Factory ko'pincha ichida bir nechta Factory Method dan foydalanadi.

### ❓ Builder qanday muammoni hal qiladi?

**✅ Javob:** Murakkab object ni bosqichma-bosqich qurish va "telescoping constructor" muammosini (juda ko'p parametrli constructor) hal qiladi. Fluent interface (method chaining) orqali o'qiladigan kod beradi. Query builder lar bunga klassik misol.

### ❓ Decorator va Inheritance orasidagi farq?

**✅ Javob:** Inheritance — compile-time da, sobit ierarxiya. Decorator — runtime da, dinamik. Decorator "kombinatsiya portlashi" muammosini hal qiladi: N ta xususiyatning har bir kombinatsiyasi uchun alohida subclass yozish o'rniga, decorator larni runtime da qatlamlash mumkin (masalan, Coffee + Milk + Sugar).

### ❓ Adapter qachon kerak?

**✅ Javob:** Ikkita mos kelmaydigan interfeysni birlashtirish kerak bo'lganda — ayniqsa eski (legacy) yoki uchinchi tomon kodini o'zgartira olmaganda. Adapter "tarjimon" vazifasini bajaradi: bizning kutilgan interfeysni tashqi kutubxonaning interfeysiga moslaydi.

### ❓ Facade va Adapter farqi nima?

**✅ Javob:** Facade — murakkab subsystem ustiga **soddalashtirilgan** interfeys (maqsad: oddiylashtirish). Adapter — mavjud interfeysni **boshqa kutilgan** interfeysga moslash (maqsad: moslik). Facade yangi soddalashtirilgan API yaratadi, Adapter mavjud API ni o'zgartiradi.

### ❓ Proxy ning asosiy turlari qaysilar?

**✅ Javob:** **Virtual proxy** (lazy loading — qimmat object ni kechiktirib yaratish), **Protection proxy** (access control), **Caching proxy** (natijalarni keshlash), **Remote proxy** (uzoq object bilan ishlash). Barchasi real object ga kirishni nazorat qiladi.

### ❓ Observer pattern qanday ishlaydi va u qayerda qo'llaniladi?

**✅ Javob:** Subject o'z holatini o'zgartirganda, ro'yxatga olingan barcha observer larni `notify()` orqali xabardor qiladi. Bu pub/sub g'oyasi. Qo'llaniladi: DOM event lar, RxJS, state management (Redux), WebSocket, UI data binding. Asosiy ehtiyot: `unsubscribe` qilmaslik memory leak ga olib keladi.

### ❓ Observer va Pub/Sub orasidagi farq?

**✅ Javob:** Klassik Observer da subject observer larni bevosita biladi (to'g'ridan-to'g'ri reference). Pub/Sub da ular orasida **event bus/message broker** turadi — publisher va subscriber bir-birini bilmaydi. Pub/Sub coupling ni yanada pasaytiradi va masshtablanadi.

### ❓ Strategy va State pattern lari nimasi bilan farqlanadi?

**✅ Javob:** Strukturasi o'xshash (ikkalasi ham object ga delegatsiya qiladi), lekin maqsadi farqli. Strategy — **client** algoritmni tanlaydi va u odatda o'zgarmaydi. State — object **o'zi** holatini avtomatik o'zgartiradi va holatlar bir-biriga o'tadi. Strategy "qanday qilish", State "qaysi holatda" haqida.

### ❓ Strategy va Template Method farqi?

**✅ Javob:** Template Method — **inheritance** orqali (parent skeletni belgilaydi, subclass qadamlarni implement qiladi, compile-time). Strategy — **composition** orqali (algoritm alohida object, runtime da almashtirish mumkin). "Favor composition" qoidasiga ko'ra Strategy moslashuvchanroq.

### ❓ Command pattern ning asosiy foydasi nima?

**✅ Javob:** So'rovni object sifatida o'rab, uni navbatga qo'yish, log qilish va eng muhimi — **undo/redo** qilish imkonini beradi. Editor lar, transaction tizimlari va task queue lar bunga asoslanadi. Har bir command `execute()` va `undo()` metodlariga ega.

### ❓ Pattern larni haddan oshirib ishlatishning xavfi nima?

**✅ Javob:** Over-engineering — muammo bo'lmagan joyga pattern qo'shish keraksiz murakkablik, ko'proq kod va qiyinroq o'qish keltiradi (YAGNI buzilishi). Pattern muammoni ergashishi kerak. Avval oddiy yechim, keyin real ehtiyoj paydo bo'lganda refaktoring — bu to'g'ri yo'l.

---

## Masalalar

> Yechimlar: [02-design-patterns yechimlari](../solutions/oop-patterns/02-design-patterns.md)

1. **Singleton Logger:** Yagona instance li `Logger` Singleton yozing (`log`, `getInstance`). So'ng uni nega DI bilan almashtirish yaxshiroq ekanini bir-ikki qator izoh bilan ko'rsating.

2. **Factory — shakl yaratuvchi:** `ShapeFactory` yozing. U `"circle" | "square"` qabul qilib, mos `Shape` object qaytarsin. Yangi shakl qo'shish oson bo'lsin.

3. **Builder — HTTP request:** Fluent `RequestBuilder` yozing (`setUrl`, `setMethod`, `addHeader`, `setBody`, `build`). Method chaining ishlasin.

4. **Decorator — narx hisoblash:** `Coffee` interfeysi va `MilkDecorator`, `SugarDecorator`, `WhipDecorator` yozing. Ularni runtime da qatlamlab, jami narx va tavsifni chiqaring.

5. **Observer — yangiliklar:** `NewsAgency` (subject) va `Subscriber` (observer) yozing. Bir nechta obunachi qo'shing, `unsubscribe` ni ham qo'llab-quvvatlang va memory leak dan saqlaning.

6. **Strategy — to'lov:** `PaymentStrategy` (Card, PayPal, Crypto) yozing. `Checkout` class runtime da strategiyani almashtira olsin.

7. **Adapter — temperatura:** Faraqat Fahrenheit qaytaruvchi eski `WeatherApi` ni, Celsius kutadigan yangi tizimga moslovchi Adapter yozing.

8. **State — buyurtma:** `Order` ni `Pending → Shipped → Delivered` holatlari bilan State pattern orqali modellashtiring. Har bir `next()` keyingi holatga o'tsin.

9. **Command — undo:** `TextEditor` uchun `Command` (AddText) yozing. `execute` va `undo` ishlasin, history stack saqlansin.

10. **Composite — fayl tizimi:** `File` (leaf) va `Folder` (composite) yozing. `Folder.size()` rekursiv ravishda barcha bolalar hajmini qaytarsin.

---

← [OOP bo'limiga qaytish](./README.md)
