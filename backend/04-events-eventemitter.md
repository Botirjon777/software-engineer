# Events va EventEmitter

Node.js asosan **event-driven** (hodisalarga asoslangan) arxitekturaga quriladi. HTTP server, stream'lar, socket'lar, fayl o'qish — bularning deyarli barchasi `EventEmitter`'ga tayanadi. Bu hujjat event-driven arxitektura, `EventEmitter` class metodlari (`on`, `emit`, `once`, `off`), custom emitter yaratish, `error` event'ining maxsus roli, memory leak va `maxListeners`, sync vs async emit, observer pattern, `EventTarget` bilan farqi va oddiy pub/sub qurishni qamrab oladi.

## Mundarija

- [Event-driven arxitektura](#event-driven-arxitektura)
- [EventEmitter asoslari: on, emit, once, off](#eventemitter-asoslari-on-emit-once-off)
- [Custom EventEmitter yaratish](#custom-eventemitter-yaratish)
- [Maxsus 'error' event](#maxsus-error-event)
- [Memory leak va maxListeners](#memory-leak-va-maxlisteners)
- [Sync vs async emit](#sync-vs-async-emit)
- [Observer pattern](#observer-pattern)
- [EventTarget vs EventEmitter](#eventtarget-vs-eventemitter)
- [Oddiy pub/sub qurish](#oddiy-pubsub-qurish)
- [Interview Q&A](#interview-qa)
- [Masalalar](#masalalar)

---

## Event-driven arxitektura

**💡 Tushuncha:** Event-driven arxitekturada kod "biror narsa yuz berganda javob beradi" tamoyili bo'yicha ishlaydi. Bir tomon **hodisa chiqaradi** (emit), boshqa tomonlar esa **hodisani tinglaydi** (listen) va reaksiya bildiradi. Bu **loose coupling** (zaif bog'lanish)ni ta'minlaydi — emitter listener'lar haqida hech narsa bilmaydi.

Node'da bu pattern markaziy: HTTP server `'request'` event chiqaradi, stream `'data'`/`'end'`/`'error'` chiqaradi, process `'exit'`/`'uncaughtException'` chiqaradi.

```js
const server = require('http').createServer();
server.on('request', (req, res) => res.end('salom'));
server.on('error', (err) => console.error(err));
server.listen(3000);
// server qachon so'rov kelishini bilmaydi — kelganda 'request' chiqaradi
```

**Afzalliklari:** modulli, kengaytiriladigan (yangi listener qo'shish oson), komponentlar bir-biriga bog'liq emas.

---

## EventEmitter asoslari: on, emit, once, off

**💡 Tushuncha:** `EventEmitter` — `events` modulidagi class. U "event nomi -> listener'lar ro'yxati" xaritasini saqlaydi va `emit` chaqirilganda barcha mos listener'larni chaqiradi.

```js
const EventEmitter = require('events');
const emitter = new EventEmitter();

// on — listener qo'shish (har emit'da ishlaydi)
emitter.on('salom', (ism) => {
  console.log(`Salom, ${ism}!`);
});

// emit — event chiqarish + argumentlar uzatish
emitter.emit('salom', 'Ali');  // Salom, Ali!
emitter.emit('salom', 'Vali'); // Salom, Vali!
```

Asosiy metodlar:

- **`on(event, listener)`** (yoki `addListener`) — listener qo'shadi, har `emit`da ishlaydi.
- **`once(event, listener)`** — listener faqat **bir marta** ishlaydi, keyin avtomatik o'chadi.
- **`emit(event, ...args)`** — event chiqaradi, argumentlarni listener'larga uzatadi. Listener bo'lsa `true`, bo'lmasa `false` qaytaradi.
- **`off(event, listener)`** (yoki `removeListener`) — aniq listener'ni o'chiradi.
- **`removeAllListeners([event])`** — barcha (yoki bitta event'ning) listener'larini o'chiradi.
- **`listenerCount(event)`** — listener'lar sonini qaytaradi.
- **`prependListener(event, listener)`** — listener'ni ro'yxat **boshiga** qo'shadi.

```js
function handler(data) { console.log('once:', data); }

emitter.once('start', handler);
emitter.emit('start', 1); // once: 1
emitter.emit('start', 2); // hech narsa — listener o'chgan

// off bilan o'chirish (named function kerak!)
emitter.on('tick', handler);
emitter.off('tick', handler);
```

**⚠️ Ehtiyot bo'l:** `off`/`removeListener` faqat **aynan o'sha funksiya referensi** bilan ishlaydi. Anonymous (nomsiz) arrow funksiyani o'chira olmaysiz, chunki uning referensi saqlanmaydi:

```js
emitter.on('x', () => console.log('a')); // bu listener'ni keyin o'chirib bo'lmaydi!
```

---

## Custom EventEmitter yaratish

**💡 Tushuncha:** O'z class'ingizni `EventEmitter`'dan `extend` qilib, unga event chiqarish qobiliyatini berasiz. Bu Node'da juda keng tarqalgan pattern.

```js
const EventEmitter = require('events');

class Order extends EventEmitter {
  constructor(id) {
    super(); // EventEmitter konstruktorini chaqirish SHART
    this.id = id;
    this.status = 'new';
  }

  pay() {
    this.status = 'paid';
    this.emit('paid', { id: this.id, at: Date.now() });
  }

  ship() {
    this.status = 'shipped';
    this.emit('shipped', this.id);
  }
}

const order = new Order(42);
order.on('paid', (info) => console.log('To\'lov qabul qilindi:', info));
order.on('shipped', (id) => console.log(`Buyurtma ${id} jo'natildi`));

order.pay();  // To'lov qabul qilindi: { id: 42, at: ... }
order.ship(); // Buyurtma 42 jo'natildi
```

**⚠️ Ehtiyot bo'l:** `super()`ni chaqirishni unutsangiz, `this.emit` ishlamaydi va xato beradi. Bu custom emitter'lardagi eng keng tarqalgan xatolardan biri.

---

## Maxsus 'error' event

**💡 Tushuncha:** `'error'` — `EventEmitter`'da maxsus ahamiyatga ega event. Agar `'error'` event chiqsa va unga **birorta ham listener bo'lmasa**, Node `Error`ni **throw qiladi** va dastur **crash** bo'ladi (uncaught exception).

```js
const emitter = require('events').EventEmitter;
const e = new emitter();

e.emit('error', new Error('Nimadir buzildi'));
// Listener yo'q -> dastur CRASH bo'ladi:
// Uncaught Error: Nimadir buzildi
```

To'g'ri yondashuv — har doim `'error'` listener qo'shish:

```js
e.on('error', (err) => {
  console.error('Xato ushlandi:', err.message);
  // crash o'rniga log/cleanup
});
e.emit('error', new Error('Endi xavfsiz')); // dastur yashaydi
```

**✅ Javob (nega shunday?):** Bu Node'ning ataylab qilingan dizayni. Async xatolarni "jim" o'tkazib yuborish xavfli, shuning uchun listener bo'lmasa Node sizni majburan crash qiladi — bu yashirin buglardan himoyalaydi.

**⚠️ Ehtiyot bo'l:** Stream'lar bilan ishlaganda (`fs.createReadStream`, socket'lar) `'error'` listener qo'shishni **hech qachon unutmang** — aks holda kichik xato (masalan, fayl topilmadi) butun serverni yiqitadi.

---

## Memory leak va maxListeners

**💡 Tushuncha:** Har bir `on` chaqiruvi listener qo'shadi. Agar bir event'ga listener qo'shaverib o'chirmasangiz (masalan, har so'rovda yangi listener), ular yig'ilib **memory leak**ka aylanadi. Node bundan ogohlantirish uchun **default 10 ta** listener chegarasiga ega.

```js
const e = new (require('events'))();
for (let i = 0; i < 11; i++) {
  e.on('data', () => {});
}
// Ogohlantirish:
// MaxListenersExceededWarning: Possible EventEmitter memory leak detected.
// 11 data listeners added. Use emitter.setMaxListeners() to increase limit
```

**Chegara nimani anglatadi:** Bu **xato emas, faqat ogohlantirish** — leak bo'lishi mumkinligini bildiradi. Ko'pincha bu haqiqatan ham bug (listener'larni o'chirmaslik) belgisi.

Boshqarish usullari:

```js
e.setMaxListeners(20);          // bitta emitter uchun chegarani oshirish
e.setMaxListeners(0);           // 0 -> cheksiz (ehtiyotkorlik bilan!)
require('events').defaultMaxListeners = 15; // global default
```

**To'g'ri yechim — listener'larni tozalash:**

```js
function onData(d) { /* ... */ }
e.on('data', onData);
// ish tugagach:
e.off('data', onData); // yoki once() ishlatish
```

**⚠️ Ehtiyot bo'l:** `setMaxListeners`ni shunchaki ogohlantirish chiqdi deb oshirib qo'yish — odatda noto'g'ri yechim. Avval **nega** shuncha listener yig'ilayotganini tekshiring; ko'pincha bu o'chirilmagan listener (leak) belgisidir.

---

## Sync vs async emit

**💡 Tushuncha:** `emit` **sinxron** ishlaydi. `emitter.emit('x')` chaqirilganda barcha listener'lar **darhol, navbat bilan, joriy call stack'da** chaqiriladi — keyingi qatorga o'tishdan oldin.

```js
const e = new (require('events'))();
e.on('test', () => console.log('2: listener'));

console.log('1: emit dan oldin');
e.emit('test');
console.log('3: emit dan keyin');

// Chiqish (sinxron tartib):
// 1: emit dan oldin
// 2: listener
// 3: emit dan keyin
```

Agar listener uzoq sinxron ish qilsa, u emit qilgan kodni bloklaydi. Async xatti-harakat kerak bo'lsa, listener ichida `setImmediate`/`process.nextTick`/Promise ishlatiladi:

```js
e.on('test', () => {
  setImmediate(() => console.log('keyin (async) ishlaydi'));
});
```

**⚠️ Ehtiyot bo'l:** `emit` sinxron bo'lgani uchun listener'da tashlangan **sinxron xato** to'g'ri emit qilgan joyga "qaytib" boradi (try/catch bilan ushlash mumkin). Lekin listener async (Promise) bo'lsa, undagi xatoni emit qilgan joydagi try/catch **ushlay olmaydi** — bu klassik tuzoq:

```js
try {
  e.emit('test'); // listener async bo'lsa, undagi reject bu yerda ushlanmaydi
} catch (err) {
  // faqat SINXRON listener xatosini ushlaydi
}
```

---

## Observer pattern

**💡 Tushuncha:** `EventEmitter` — aslida **Observer pattern**ning Node'dagi amaliy ko'rinishi. Observer pattern'da **Subject** (kuzatiluvchi obyekt) o'z holati o'zgarganda barcha **Observer**larga (kuzatuvchilarga) xabar beradi. `emit` = "xabar berish", `on` = "kuzatuvchi sifatida ro'yxatdan o'tish".

```js
// Observer pattern: subject holati o'zgarsa, observer'lar xabardor bo'ladi
class Temperature extends require('events') {
  set(value) {
    this.value = value;
    this.emit('change', value); // barcha observer'larga xabar
  }
}

const sensor = new Temperature();
// Observer 1: ekranga chiqarish
sensor.on('change', (v) => console.log(`Ekran: ${v}°C`));
// Observer 2: ogohlantirish
sensor.on('change', (v) => { if (v > 30) console.log('Issiq!'); });

sensor.set(35); // Ekran: 35°C  +  Issiq!
```

**Afzalligi:** Subject (Temperature) observer'lar haqida hech narsa bilmaydi — yangi observer qo'shish uchun Subject kodini o'zgartirish shart emas (Open/Closed printsipi).

---

## EventTarget vs EventEmitter

**💡 Tushuncha:** `EventTarget` — brauzer (Web) standartidagi event interfeysi (`addEventListener`/`dispatchEvent`), Node'ga ham qo'shilgan. `EventEmitter` — Node'ning o'z, an'anaviy API'si. Ikkalasi o'xshash, lekin muhim farqlar bor.

| Xususiyat | `EventEmitter` (Node) | `EventTarget` (Web standart) |
|-----------|----------------------|------------------------------|
| Listener qo'shish | `on(name, fn)` | `addListener`... yo'q -> `addEventListener(name, fn)` |
| Event chiqarish | `emit(name, ...args)` | `dispatchEvent(new Event(name))` |
| Argumentlar | bir nechta argument uzatish mumkin | bitta `Event` obyekti |
| `once` | `once()` metod | `{ once: true }` option |
| `'error'` maxsus | ha (listener yo'q -> crash) | yo'q |
| `maxListeners` | bor | yo'q |
| Platforma | faqat Node | brauzer + Node |

```js
// EventTarget (Web-uyg'un)
const target = new EventTarget();
target.addEventListener('ping', (event) => {
  console.log('ping:', event.detail);
});
target.dispatchEvent(new CustomEvent('ping', { detail: 42 }));
```

**✅ Javob (qaysi birini tanlash?):** Node-only kod uchun `EventEmitter` qulayroq (ko'p argument, `'error'` semantikasi, ekosistema). Brauzer bilan kod ulashish yoki Web standartlariga moslik kerak bo'lsa `EventTarget` afzal.

---

## Oddiy pub/sub qurish

**💡 Tushuncha:** **Pub/Sub** (Publish/Subscribe) — event-driven pattern: publisher'lar **mavzu (topic)** bo'yicha xabar e'lon qiladi, subscriber'lar mavzuga obuna bo'ladi. Publisher va subscriber bir-birini bilmaydi — ular faqat mavzu (kanal) orqali bog'lanadi.

`EventEmitter` ustiga oddiy pub/sub:

```js
class PubSub {
  constructor() {
    this.subscribers = new Map(); // topic -> Set(callbacks)
  }

  subscribe(topic, callback) {
    if (!this.subscribers.has(topic)) {
      this.subscribers.set(topic, new Set());
    }
    this.subscribers.get(topic).add(callback);
    // unsubscribe funksiyasini qaytarish (qulay pattern)
    return () => this.subscribers.get(topic).delete(callback);
  }

  publish(topic, data) {
    const subs = this.subscribers.get(topic);
    if (!subs) return;
    for (const cb of subs) {
      cb(data);
    }
  }
}

const bus = new PubSub();

const unsub = bus.subscribe('user.created', (u) => console.log('Email yuborildi:', u.email));
bus.subscribe('user.created', (u) => console.log('Log:', u.id));

bus.publish('user.created', { id: 1, email: 'a@b.com' });
// Email yuborildi: a@b.com
// Log: 1

unsub(); // obunani bekor qilish
bus.publish('user.created', { id: 2 }); // faqat Log: 2
```

**⚠️ Ehtiyot bo'l:** Bu **in-memory, bir-jarayonli** pub/sub. Mikroservislar yoki ko'p instansiya orasida xabar almashish kerak bo'lsa, Redis Pub/Sub, RabbitMQ, Kafka kabi tashqi broker'lar kerak — bu lokal pattern butun tarmoq bo'ylab ishlamaydi.

---

## Interview Q&A

### ❓ EventEmitter nima va u qanday ishlaydi?

**✅ Javob:** `EventEmitter` — `events` modulidagi class bo'lib, "event nomi -> listener'lar ro'yxati" xaritasini saqlaydi. `on` bilan listener qo'shiladi, `emit` bilan event chiqarilganda barcha mos listener'lar **sinxron, ketma-ket** chaqiriladi. Bu Node'ning event-driven arxitekturasining asosi.

### ❓ `on` va `once` farqi nima?

**✅ Javob:** `on` listener'ni doimiy qo'shadi — har `emit`da ishlaydi. `once` listener'ni faqat bir marta ishlatadi, birinchi `emit`dan keyin avtomatik o'chiradi. `once` bir martalik hodisalar (masalan, ulanish ochilishi, dastlabki yuklash) uchun ideal.

### ❓ Listener'ni qanday o'chirasiz va qanday tuzoq bor?

**✅ Javob:** `off(event, listener)` (yoki `removeListener`) bilan. Tuzoq: faqat **aynan o'sha funksiya referensi** ishlaydi. Anonymous arrow funksiyani (`on('x', () => ...)`) o'chirib bo'lmaydi, chunki referensi saqlanmaydi. Shuning uchun o'chirish kerak bo'lgan listener'larni named function sifatida saqlash lozim.

### ❓ `'error'` event nima uchun maxsus?

**✅ Javob:** Agar `'error'` event chiqsa-yu, unga listener bo'lmasa, Node `Error`ni throw qiladi va dastur crash bo'ladi. Bu ataylab qilingan: async xatolarni jim o'tkazib yuborish xavfli. Shuning uchun har bir emitter/stream uchun `'error'` listener qo'shish majburiy amaliyot.

### ❓ EventEmitter memory leak qanday yuzaga keladi?

**✅ Javob:** Listener qo'shilib, lekin o'chirilmaganda (masalan, har so'rovda yangi listener qo'shilsa). Ular yig'ilib xotirani egallaydi. Node 10 tadan ortiq listener'da `MaxListenersExceededWarning` chiqaradi. To'g'ri yechim — `off`/`once` bilan tozalash; `setMaxListeners`ni oshirish ko'pincha leak'ni yashirish, hal qilish emas.

### ❓ `emit` sinxronmi yoki asinxron?

**✅ Javob:** **Sinxron.** `emit` chaqirilganda barcha listener'lar darhol, joriy call stack'da, ketma-ket chaqiriladi va keyingina `emit`dan keyingi qator ishlaydi. Async kerak bo'lsa, listener ichida `setImmediate`/`process.nextTick`/Promise ishlatiladi.

### ❓ Listener'da async xato (Promise reject) qanday boshqariladi?

**✅ Javob:** `emit` sinxron bo'lgani uchun, emit qilgan joydagi `try/catch` faqat sinxron listener xatolarini ushlaydi. Async listener'da `await`/Promise reject bo'lsa, u emit qilgan joyga qaytmaydi — `unhandledRejection`ga aylanadi. Yechim: listener ichida xatoni o'zi ushlash yoki emitter'da `'error'` orqali uzatish.

### ❓ Observer pattern va EventEmitter qanday bog'liq?

**✅ Javob:** `EventEmitter` — Observer pattern'ning Node implementatsiyasi. Subject (emitter) holati o'zgarganda `emit` orqali barcha observer'larga (listener'larga) xabar beradi. Bu loose coupling beradi: subject observer'lar haqida bilmaydi.

### ❓ EventEmitter va EventTarget farqi nima?

**✅ Javob:** `EventEmitter` — Node'ning an'anaviy API'si (`on`/`emit`, ko'p argument, `'error'` maxsus semantikasi, `maxListeners`). `EventTarget` — Web standarti (`addEventListener`/`dispatchEvent`, bitta `Event` obyekti), brauzer va Node'da ishlaydi. Node-only kodda EventEmitter qulay; brauzer bilan moslik kerak bo'lsa EventTarget.

### ❓ Pub/Sub va EventEmitter farqi bormi?

**✅ Javob:** Konseptual jihatdan pub/sub — publisher va subscriber'lar **mavzu (topic)** orqali bog'lanadigan pattern. `EventEmitter` lokal, bir-jarayonli pub/sub sifatida ishlatilishi mumkin. Lekin tarmoq bo'ylab (mikroservislar) pub/sub uchun Redis/RabbitMQ/Kafka kabi tashqi broker kerak — EventEmitter faqat bitta process ichida.

### ❓ `prependListener` qachon kerak?

**✅ Javob:** U listener'ni ro'yxat **boshiga** qo'shadi, shuning uchun u boshqa (oldin qo'shilgan) listener'lardan oldin ishlaydi. Bu listener'lar bajarilish tartibi muhim bo'lganda (masalan, biror narsani boshqalardan oldin loglash yoki tekshirish) ishlatiladi.

### ❓ `emit` qaytaradigan qiymat nima?

**✅ Javob:** `emit` event uchun kamida bitta listener bo'lsa `true`, hech qanday listener bo'lmasa `false` qaytaradi. Bu event "eshitildimi yoki yo'qmi" tekshirish uchun foydali bo'lishi mumkin.

### ❓ Custom EventEmitter yaratishda eng keng tarqalgan xato qaysi?

**✅ Javob:** Konstruktor'da `super()`ni chaqirishni unutish. `class X extends EventEmitter` qilganda `super()` chaqirilmasa, ichki listener xaritasi initsializatsiya bo'lmaydi va `this.emit`/`this.on` ishlamaydi yoki xato beradi.

### ❓ `removeAllListeners` qachon xavfli?

**✅ Javob:** Argumentsiz `removeAllListeners()` **barcha** event'larning barcha listener'larini o'chiradi — shu jumladan kutubxonalar yoki ichki kod qo'shgan listener'larni ham. Bu kutilmagan xatti-harakatlarga olib kelishi mumkin. Yaxshisi aniq event nomini berish: `removeAllListeners('data')`.

---

## Masalalar

> Yechimlar: [solutions/backend/04-events-eventemitter.md](../solutions/backend/04-events-eventemitter.md)

1. `EventEmitter`'dan `extend` qiluvchi `Timer` class yarating. U `start()` chaqirilganda har soniyada `'tick'` event chiqarsin (raqam bilan) va `stop()` chaqirilganda `'stopped'` event chiqarib to'xtasin.

2. `once` yordamida faqat birinchi `'connect'` event'iga javob beradigan kichik dastur yozing. So'ng `once`ni `on` bilan almashtirib, farqni ko'rsating.

3. Anonymous arrow funksiya bilan qo'shilgan listener'ni `off` orqali o'chirib bo'lmasligini ko'rsating, keyin uni named function bilan to'g'rilang.

4. `'error'` event'iga listener bo'lmaganda dastur crash bo'lishini ko'rsating, so'ng listener qo'shib, dastur "yashashini" ta'minlang.

5. 15 ta listener qo'shib `MaxListenersExceededWarning`ni keltirib chiqaring. So'ng buni ikki xil usulda hal qiling: (a) `setMaxListeners`, (b) listener'larni to'g'ri o'chirish.

6. `emit`ning sinxronligini isbotlovchi `console.log` tartibi misolini yozing. Keyin listener'ni `setImmediate` bilan async qiling va tartib o'zgarishini ko'rsating.

7. Observer pattern asosida `Stock` (aksiya) class yarating: narx o'zgarganda barcha kuzatuvchilarga xabar bersin. Kamida 2 ta turli observer (loglash, ogohlantirish) qo'shing.

8. `subscribe`/`publish`/`unsubscribe` metodlariga ega to'liq `PubSub` class yozing. `subscribe` unsubscribe funksiyasini qaytarsin. Bir nechta topic'ni qo'llab-quvvatlasin.

9. `EventEmitter` va `EventTarget` yordamida bir xil "ping/pong" mantig'ini ikki xil API bilan yozing va kod farqlarini izohlang.

10. Async listener'da tashlangan Promise reject xatosi emit qilgan joydagi `try/catch` bilan ushlanmasligini ko'rsating, so'ng xatoni to'g'ri boshqarishning kamida bitta usulini qo'llang.

---

← [Backend bo'limiga qaytish](./README.md)
