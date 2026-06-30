# Events va EventEmitter — Masalalar Yechimlari

Bu fayl [Events va EventEmitter](../../backend/04-events-eventemitter.md) mavzusidagi "Masalalar" bo'limining yechimlarini o'z ichiga oladi. Avval o'zingiz yechib ko'ring, keyin solishtiring.

---

## 1-masala — Timer class

```js
const EventEmitter = require('events');

class Timer extends EventEmitter {
  constructor() {
    super();
    this.count = 0;
    this.interval = null;
  }

  start() {
    this.interval = setInterval(() => {
      this.count++;
      this.emit('tick', this.count);
    }, 1000);
  }

  stop() {
    clearInterval(this.interval);
    this.interval = null;
    this.emit('stopped', this.count);
  }
}

const t = new Timer();
t.on('tick', (n) => console.log('tick:', n));
t.on('stopped', (total) => console.log('To\'xtadi, jami:', total));

t.start();
setTimeout(() => t.stop(), 3500); // tick: 1, 2, 3 -> To'xtadi, jami: 3
```

---

## 2-masala — once vs on

```js
const e = new (require('events'))();

// once — faqat birinchi connect
e.once('connect', (id) => console.log('once connect:', id));
e.emit('connect', 1); // once connect: 1
e.emit('connect', 2); // hech narsa

// on — har connect
e.on('connect2', (id) => console.log('on connect:', id));
e.emit('connect2', 1); // on connect: 1
e.emit('connect2', 2); // on connect: 2
```

**Farq:** `once` listener birinchi `emit`dan keyin avtomatik o'chadi; `on` har `emit`da ishlayveradi.

---

## 3-masala — Anonymous vs named listener'ni off qilish

```js
const e = new (require('events'))();

// XATO: anonymous arrow — o'chirib bo'lmaydi
e.on('x', () => console.log('anonymous'));
e.off('x', () => console.log('anonymous')); // BOSHQA referens -> o'chmaydi
e.emit('x'); // 'anonymous' baribir chiqadi

// TO'G'RI: named function — referens saqlanadi
function handler() { console.log('named'); }
e.on('y', handler);
e.off('y', handler); // o'chadi
e.emit('y'); // hech narsa
```

**Sabab:** `off` listener'ni referens (xotira manzili) bo'yicha taqqoslaydi. Ikkita bir xil ko'rinishdagi arrow funksiya — ikki xil referens, shuning uchun o'chmaydi.

---

## 4-masala — 'error' event va crash

```js
const e = new (require('events'))();

// Listener YO'Q -> crash
// e.emit('error', new Error('crash')); // Uncaught Error: crash -> dastur o'ladi

// Listener BOR -> xavfsiz
e.on('error', (err) => console.error('Ushlandi:', err.message));
e.emit('error', new Error('endi xavfsiz')); // Ushlandi: endi xavfsiz
console.log('Dastur davom etmoqda'); // bu qator ishlaydi
```

**Izoh:** `'error'` event'iga listener bo'lmasa, Node throw qiladi va process crash bo'ladi. Listener qo'shilsa, xato boshqariladi va dastur yashaydi.

---

## 5-masala — MaxListenersExceededWarning va yechimlar

```js
const e = new (require('events'))();

// Ogohlantirish chiqaradi (>10)
for (let i = 0; i < 15; i++) e.on('data', () => {});
// MaxListenersExceededWarning: 15 data listeners added...
```

**Yechim (a) — setMaxListeners:**

```js
const e1 = new (require('events'))();
e1.setMaxListeners(20);
for (let i = 0; i < 15; i++) e1.on('data', () => {}); // ogohlantirish yo'q
```

**Yechim (b) — listener'larni to'g'ri o'chirish (afzalroq):**

```js
const e2 = new (require('events'))();
function makeHandler(i) { return () => {}; }
const handlers = [];
for (let i = 0; i < 15; i++) {
  const h = makeHandler(i);
  handlers.push(h);
  e2.on('data', h);
}
// ish tugagach tozalash:
handlers.forEach((h) => e2.off('data', h));
```

**Eslatma:** (b) — to'g'riroq yechim, chunki (a) ko'pincha leak'ni shunchaki yashiradi. Avval **nega** shuncha listener kerakligini aniqlash lozim.

---

## 6-masala — emit sinxronligi va async qilish

**Sinxron (default):**

```js
const e = new (require('events'))();
e.on('test', () => console.log('2: listener'));

console.log('1: oldin');
e.emit('test');
console.log('3: keyin');
// 1: oldin
// 2: listener
// 3: keyin
```

**Listener async qilinganda:**

```js
const e2 = new (require('events'))();
e2.on('test', () => setImmediate(() => console.log('2: async listener')));

console.log('1: oldin');
e2.emit('test');
console.log('3: keyin');
// 1: oldin
// 3: keyin
// 2: async listener   <- endi keyin chiqadi
```

**Izoh:** `emit` o'zi har doim sinxron listener'ni chaqiradi, lekin listener ichidagi `setImmediate` ishni event loop'ning check fazasiga kechiktiradi, shuning uchun `3` `2`dan oldin chiqadi.

---

## 7-masala — Observer pattern: Stock

```js
const EventEmitter = require('events');

class Stock extends EventEmitter {
  constructor(symbol, price) {
    super();
    this.symbol = symbol;
    this.price = price;
  }
  setPrice(newPrice) {
    const old = this.price;
    this.price = newPrice;
    this.emit('price', { symbol: this.symbol, old, current: newPrice });
  }
}

const aapl = new Stock('AAPL', 100);

// Observer 1: loglash
aapl.on('price', (p) => console.log(`${p.symbol}: ${p.old} -> ${p.current}`));

// Observer 2: ogohlantirish
aapl.on('price', (p) => {
  const change = ((p.current - p.old) / p.old) * 100;
  if (Math.abs(change) > 5) console.log(`Ogohlantirish: ${change.toFixed(1)}% o'zgarish!`);
});

aapl.setPrice(110);
// AAPL: 100 -> 110
// Ogohlantirish: 10.0% o'zgarish!
```

---

## 8-masala — To'liq PubSub class

```js
class PubSub {
  constructor() {
    this.topics = new Map(); // topic -> Set(callback)
  }

  subscribe(topic, callback) {
    if (!this.topics.has(topic)) this.topics.set(topic, new Set());
    this.topics.get(topic).add(callback);
    return () => {
      const set = this.topics.get(topic);
      if (set) set.delete(callback);
    };
  }

  publish(topic, data) {
    const set = this.topics.get(topic);
    if (!set) return false;
    for (const cb of set) cb(data);
    return true;
  }

  unsubscribeAll(topic) {
    this.topics.delete(topic);
  }
}

const bus = new PubSub();
const unsub = bus.subscribe('news', (d) => console.log('Obunachi A:', d));
bus.subscribe('news', (d) => console.log('Obunachi B:', d));
bus.subscribe('sport', (d) => console.log('Sport:', d));

bus.publish('news', 'Yangilik!');  // A va B
bus.publish('sport', 'Gol!');      // Sport
unsub();
bus.publish('news', 'Yana yangilik'); // faqat B
```

---

## 9-masala — EventEmitter vs EventTarget ping/pong

**EventEmitter versiyasi:**

```js
const EventEmitter = require('events');
const e = new EventEmitter();
e.on('ping', (n) => {
  console.log('ping olindi:', n);
  e.emit('pong', n + 1);
});
e.on('pong', (n) => console.log('pong olindi:', n));
e.emit('ping', 1); // ping olindi: 1 -> pong olindi: 2
```

**EventTarget versiyasi:**

```js
const target = new EventTarget();
target.addEventListener('ping', (event) => {
  console.log('ping olindi:', event.detail);
  target.dispatchEvent(new CustomEvent('pong', { detail: event.detail + 1 }));
});
target.addEventListener('pong', (event) => console.log('pong olindi:', event.detail));
target.dispatchEvent(new CustomEvent('ping', { detail: 1 }));
```

**Farqlar:**
- `on` vs `addEventListener`.
- `emit(name, arg)` (to'g'ridan-to'g'ri argument) vs `dispatchEvent(new CustomEvent(name, { detail }))` (data `event.detail` ichida).
- EventEmitter bir nechta argument uzata oladi; EventTarget faqat bitta Event obyekti.
- EventTarget — Web standarti, brauzerda ham ishlaydi.

---

## 10-masala — Async listener xatosini boshqarish

**Muammo (ushlanmaydi):**

```js
const e = new (require('events'))();
e.on('task', async () => {
  throw new Error('async xato');
});

try {
  e.emit('task'); // emit sinxron tugaydi, lekin async xato KEYIN reject bo'ladi
} catch (err) {
  console.log('Bu ISHLAMAYDI'); // bu yerda ushlanmaydi
}
// -> unhandledRejection
```

**Yechim — listener ichida xatoni o'zi ushlash:**

```js
const e2 = new (require('events'))();
e2.on('task', async () => {
  try {
    throw new Error('async xato');
  } catch (err) {
    console.error('Listener ichida ushlandi:', err.message);
    e2.emit('error', err); // yoki emitter orqali uzatish (error listener bilan)
  }
});
e2.on('error', (err) => console.error('error event:', err.message));
e2.emit('task'); // Listener ichida ushlandi: async xato + error event: async xato
```

**Sabab:** `emit` sinxron — u listener'ni boshlaydi va darhol qaytadi. Async listener'ning Promise'i keyin reject bo'ladi, ammo `emit` allaqachon qaytib bo'lgan, shu sabab tashqi `try/catch` uni ko'rmaydi. Xatoni listener'ning o'zi `try/catch` bilan boshqarishi yoki emitter'ning `'error'` mexanizmiga uzatishi kerak.

---

← [Backend bo'limiga qaytish](../../backend/README.md)
