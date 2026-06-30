# Event Loop — Masalalar Yechimlari

Bu fayl [Event Loop (Chuqur)](../../backend/03-event-loop.md) mavzusidagi "Masalalar" bo'limining yechimlarini o'z ichiga oladi. Avval o'zingiz yechib ko'ring, keyin solishtiring.

---

## 1-masala — Tartibni izohlang

```js
console.log('A');
setTimeout(() => console.log('B'), 0);
Promise.resolve().then(() => console.log('C'));
process.nextTick(() => console.log('D'));
console.log('E');
```

**Chiqish:**

```
A
E
D
C
B
```

**Izoh:**
- `A`, `E` — sinxron kod, darhol call stack'da bajariladi.
- Call stack bo'shadi -> microtask'lar: avval nextTick queue -> `D`.
- Keyin Promise queue -> `C`.
- Eng oxirida event loop timers fazasiga o'tadi -> `B` (macrotask).

---

## 2-masala — I/O callback ichida setTimeout vs setImmediate

```js
const fs = require('fs');
fs.readFile(__filename, () => {
  setTimeout(() => console.log('timeout'), 0);
  setImmediate(() => console.log('immediate'));
});
```

**Chiqish (har doim):**

```
immediate
timeout
```

**Nega:** `fs.readFile` callback **poll** fazasida bajariladi. Poll fazasidan keyin darhol **check** fazasi keladi. `setImmediate` aynan check fazasida ishlaydi, shuning uchun u darhol bajariladi. `setTimeout(0)` esa keyingi loop aylanishidagi **timers** fazasini kutishi kerak. Demak I/O kontekstida `setImmediate` **doim** oldin — bu deterministik.

---

## 3-masala — nextTick starvation va setImmediate yechimi

**Muammoli kod (timer hech qachon ishlamaydi):**

```js
function recurse() {
  process.nextTick(recurse);
}
recurse();
setTimeout(() => console.log('HECH QACHON'), 0);
// nextTick queue cheksiz to'lib turadi -> event loop timers fazasiga o'ta olmaydi
```

**Yechim (setImmediate bilan):**

```js
function recurse() {
  setImmediate(recurse);
}
recurse();
setTimeout(() => console.log('CHIQADI'), 100);
// setImmediate har loop iteratsiyasida bir marta ishlaydi -> timers fazasi nafas oladi -> timer ishlaydi
```

**Sabab:** `setImmediate` callback'lari check fazasida bo'ladi va event loop har iteratsiyada barcha fazalardan o'tadi, shu sabab timers fazasi ham o'z navbatini oladi. `nextTick` esa fazalar orasidan tashqarida, har doim oldin bo'shatiladi — rekursiya uni cheksiz qiladi.

---

## 4-masala — CPU-bound bloklash va yechimlar

**Bloklaydigan versiya:**

```js
const http = require('http');
http.createServer((req, res) => {
  let s = 0;
  for (let i = 0; i < 1e9; i++) s += i; // event loop BLOK
  res.end(String(s));
}).listen(3000);
// Bu sikl ishlayotganda boshqa so'rovlar javob olmaydi
```

**Yechim A — worker_threads:**

`heavy-worker.js`:
```js
const { parentPort, workerData } = require('worker_threads');
let s = 0;
for (let i = 0; i < workerData; i++) s += i;
parentPort.postMessage(s);
```

`server.js`:
```js
const http = require('http');
const { Worker } = require('worker_threads');

http.createServer((req, res) => {
  const worker = new Worker('./heavy-worker.js', { workerData: 1e9 });
  worker.on('message', (s) => res.end(String(s)));
  worker.on('error', (e) => { res.statusCode = 500; res.end(String(e)); });
}).listen(3000);
// Hisob alohida thread'da -> asosiy event loop bo'sh -> boshqa so'rovlar javob oladi
```

**Yechim B — chunking (setImmediate):**

```js
function sumChunked(n, cb) {
  let s = 0, i = 0;
  function step() {
    const end = Math.min(i + 1e6, n);
    for (; i < end; i++) s += i;
    if (i < n) setImmediate(step); // har bo'lak orasida event loop nafas oladi
    else cb(s);
  }
  step();
}
```

---

## 5-masala — Aralashma tartibini bashorat qilish

```js
setTimeout(() => console.log('T1'), 0);
setImmediate(() => console.log('I1'));
Promise.resolve().then(() => {
  console.log('P1');
  process.nextTick(() => console.log('N1'));
});
process.nextTick(() => console.log('N2'));
```

**Chiqish:**

```
N2
P1
N1
T1   (yoki I1 — main module'da T1/I1 tartibi noaniq bo'lishi mumkin)
I1
```

**Izoh:**
- Sinxron kod yo'q (faqat rejalashtirish).
- Microtask'lar: avval nextTick -> `N2`. Keyin Promise -> `P1` (uning ichida yangi `nextTick` qo'shiladi).
- Promise callback tugadi -> yana nextTick queue tekshiriladi -> `N1`.
- Endi event loop fazalarga o'tadi: timers (`T1`) va check (`I1`). Main module'dan chaqirilgani uchun `T1` va `I1` orasidagi tartib mashina yuklamasiga bog'liq, ko'pincha `T1` -> `I1` chiqadi.

**Muhim:** `N1` `T1`dan oldin chiqadi, chunki `N1` Promise callback tugagandan keyin darhol (microtask sifatida) bajariladi, timer'lar esa keyin.

---

## 6-masala — UV_THREADPOOL_SIZE ta'siri

```js
process.env.UV_THREADPOOL_SIZE = 1; // dasturni ishga tushirishdan oldin
const crypto = require('crypto');
const start = Date.now();
for (let i = 0; i < 4; i++) {
  crypto.pbkdf2('p', 's', 100000, 64, 'sha512', () => {
    console.log(`#${i}: ${Date.now() - start}ms`);
  });
}
```

**Kutilgan natija (pool=1):** 4 ta operatsiya **ketma-ket** ishlaydi (chunki faqat 1 thread). Natijalar taxminan: ~T, ~2T, ~3T, ~4T (bir-biriga teng oraliqlarda ortib boradi).

**Pool=4 (default) bilan:** 4 tasi **parallel** ishlaydi, hammasi taxminan bir vaqtda (~T) tugaydi.

**Xulosa:** `crypto.pbkdf2`, `fs`, `dns.lookup` thread pool'ni ishlatadi. Pool o'lchami concurrency'ni cheklaydi: agar parallel og'ir operatsiyalar ko'p bo'lsa, `UV_THREADPOOL_SIZE`ni oshirish (lekin CPU yadrolaridan oshmaslik mantiqan) yordam beradi.

---

## 7-masala — await microtask sifatida ishlaydi

```js
console.log('1: start');

async function f() {
  console.log('2: f boshlandi');
  await Promise.resolve();
  console.log('4: await dan keyin'); // bu MICROTASK sifatida ishlaydi
}

f();
console.log('3: f dan keyin sinxron kod');

// Chiqish:
// 1: start
// 2: f boshlandi
// 3: f dan keyin sinxron kod
// 4: await dan keyin
```

**Izoh:** `f()` chaqirilganda `await`gacha bo'lgan qism sinxron ishlaydi (`2`). `await` funksiyani pauza qiladi, qolgan qism (`4`) Promise microtask'ga aylanadi. Shuning uchun `3` (sinxron) avval, `4` esa call stack bo'shagandan keyin microtask sifatida chiqadi.

---

## 8-masala — Event loop lag o'lchash

```js
function measureLag(intervalMs = 100) {
  let last = Date.now();
  setInterval(() => {
    const now = Date.now();
    const lag = now - last - intervalMs; // kutilgandan ortiqcha kechikish
    last = now;
    console.log(`lag: ${lag}ms`);
  }, intervalMs);
}

measureLag();

// Sun'iy bloklash — lag o'sadi
setInterval(() => {
  const end = Date.now() + 200;
  while (Date.now() < end) {} // 200ms BLOK
}, 500);
```

**Kutilgan natija:** Bloklash bo'lmaganda `lag` ~0ga yaqin. Sun'iy 200ms bloklash paydo bo'lganda, navbatdagi `measureLag` callback kechikadi va `lag` ~100-200ms ga sakraydi. Bu asosiy thread band ekanligini ko'rsatadi.

---

## 9-masala — dns.lookup thread pool'ni to'ldiradi

```js
process.env.UV_THREADPOOL_SIZE = 4;
const dns = require('dns');
const fs = require('fs');

const start = Date.now();

// 4 ta sekin dns.lookup pool'ni egallaydi
for (let i = 0; i < 4; i++) {
  dns.lookup(`example${i}.com`, () => {});
}

// fs operatsiyasi endi pool bo'shashini kutishi kerak
fs.readFile(__filename, () => {
  console.log(`fs tugadi: ${Date.now() - start}ms`);
});
```

**Kutilgan natija:** `dns.lookup` (getaddrinfo) thread pool'ni ishlatadi. Agar 4 ta sekin lookup pool'ni to'ldirsa, `fs.readFile` worker bo'shaguncha kutadi, shuning uchun uning tugashi sezilarli kechikadi.

**Yechim:** `dns.resolve` (haqiqiy async tarmoq) ishlatish pool'ni egallamaydi, yoki `UV_THREADPOOL_SIZE`ni oshirish.

---

## 10-masala — HTTP serverda og'ir so'rov muammosi

```js
http.createServer((req, res) => {
  let s = 0;
  for (let i = 0; i < 5e9; i++) s += i;
  res.end(String(s));
}).listen(3000);
```

**Muammo:** `for` sikli — sinxron CPU-bound ish. U bajarilayotganda asosiy thread (va u bilan butun event loop) bloklanadi. Shu vaqt davomida poll fazasi yangi ulanishlarni qabul qila olmaydi, boshqa so'rovlarning callback'lari ishlamaydi. Demak bitta so'rov serverni hamma uchun "muzlatib" qo'yadi.

**Yechim 1 — worker_threads:** Hisobni alohida thread'ga ko'chirish; asosiy event loop bo'sh qoladi va boshqa so'rovlarga xizmat qiladi.

**Yechim 2 — chunking (setImmediate):** Sikni bo'laklarga bo'lib, har bo'lak orasida `setImmediate` orqali event loop'ga nafas berish. Shunda boshqa so'rovlar oraliqda qayta ishlanadi.

**Qo'shimcha yechimlar:** `cluster` (yukni jarayonlar bo'yicha taqsimlash), `child_process` (alohida jarayon), yoki og'ir ishni tashqi queue + worker servisga uzatish.

---

← [Backend bo'limiga qaytish](../../backend/README.md)
