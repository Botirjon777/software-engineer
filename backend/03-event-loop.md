# Event Loop (Chuqur)

Event Loop — Node.js ning yuragi. U bitta thread ustida ishlay turib, minglab parallel ulanishni boshqarishga imkon beradi. Bu hujjat Node nega non-blocking ekanligidan boshlab, call stack, libuv, thread pool, event loop fazalari tartibi, microtask navbatlari, `setTimeout` vs `setImmediate`, `process.nextTick` tuzoqlari va eng qiyin tartib misollarigacha — hammasini qadama-qadam ochib beradi. Bu mavzu backend intervyularning eng sevimli (va eng ko'p adashtiruvchi) qismi.

## Mundarija

- [Nega Node non-blocking](#nega-node-non-blocking)
- [Call Stack — bitta thread qanday ishlaydi](#call-stack--bitta-thread-qanday-ishlaydi)
- [libuv va Thread Pool](#libuv-va-thread-pool)
- [Event Loop fazalari (tartib bilan)](#event-loop-fazalari-tartib-bilan)
- [Microtask navbatlari: nextTick vs Promise](#microtask-navbatlari-nexttick-vs-promise)
- [setTimeout vs setImmediate](#settimeout-vs-setimmediate)
- [process.nextTick tuzoqlari va starvation](#processnexttick-tuzoqlari-va-starvation)
- [Async I/O ichki ishlashi](#async-io-ichki-ishlashi)
- [Event Loop'ni bloklash (CPU-bound) va yechimlar](#event-loopni-bloklash-cpu-bound-va-yechimlar)
- [Qiyin tartib misoli (qadama-qadam)](#qiyin-tartib-misoli-qadama-qadam)
- [Interview Q&A](#interview-qa)
- [Masalalar](#masalalar)

---

## Nega Node non-blocking

**💡 Tushuncha:** Node.js bitta asosiy (main) thread'da JavaScript bajaradi, lekin uni "sekin" deb o'ylash xato. Sirning kaliti shundaki, Node sekin operatsiyalarni (disk o'qish, tarmoq so'rovi, DB) **bloklamasdan** OS yoki thread pool'ga "topshiradi", o'zi esa boshqa ish bilan davom etadi. Operatsiya tugagach, callback navbatga qo'yiladi.

Taqqoslash uchun ikki model:

- **Blocking (sinxron) model** — har bir so'rov tugaguncha thread kutadi. 1000 ta parallel ulanish uchun 1000 ta thread kerak (har biri ~1MB stack, context-switch xarajati katta).
- **Non-blocking (event-driven) model** — bitta thread I/O ni "boshlab qo'yadi" va kutmaydi. OS tayyor bo'lganda xabar beradi. Bitta thread minglab ulanishni boshqaradi.

```js
const fs = require('fs');

// Blocking — bu qator tugamaguncha keyingisi ishlamaydi
const data = fs.readFileSync('big.txt');
console.log('1) fayl oqildi');
console.log('2) keyingi qator');

// Non-blocking — readFile darhol qaytadi, callback keyin chaqiriladi
fs.readFile('big.txt', (err, data) => {
  console.log('3) fayl oqildi (callback)');
});
console.log('4) bu DARHOL chiqadi, faylni kutmaydi');
// Chiqish: 1, 2, 4, 3
```

**⚠️ Ehtiyot bo'l:** "Non-blocking" degani "parallel" degani emas. JavaScript kodingiz **hali ham bitta thread'da** ketma-ket ishlaydi. Faqat I/O kutish (waiting) bloklamaydi. CPU-bound og'ir hisob-kitob baribir asosiy thread'ni bloklaydi (pastda batafsil).

---

## Call Stack — bitta thread qanday ishlaydi

**💡 Tushuncha:** **Call stack** — funksiya chaqiruvlari joylashadigan LIFO (last-in, first-out) struktura. V8 har bir funksiya chaqiruvini stack'ga `push` qiladi, funksiya `return` qilganda `pop` qiladi. Event loop **faqat call stack bo'sh bo'lgandagina** keyingi callback'ni stack'ga qo'ya oladi.

```js
function uch() { console.log('uch'); }
function ikki() { uch(); }
function bir() { ikki(); }
bir();
// Stack o'sishi: bir -> ikki -> uch -> (uch pop) -> (ikki pop) -> (bir pop)
```

Muhim qoida: **call stack to'liq bo'shamaguncha hech qanday timer, I/O callback yoki microtask ishga tushmaydi.** Shuning uchun `while(true){}` butun event loop'ni muzlatib qo'yadi — stack hech qachon bo'shamaydi.

```js
setTimeout(() => console.log('timer'), 0);
while (true) {} // Bu hech qachon tugamaydi -> 'timer' HECH QACHON chiqmaydi
```

---

## libuv va Thread Pool

**💡 Tushuncha:** **libuv** — C tilida yozilgan kutubxona bo'lib, Node'ga event loop, OS thread pool va cross-platform async I/O (Linux'da `epoll`, macOS'da `kqueue`, Windows'da `IOCP`) beradi. Aynan libuv Node'ni non-blocking qiladi.

Ikki xil "async" mexanizmni ajratish juda muhim:

1. **Tarmoq I/O (network)** — bu **thread pool ishlatmaydi**. OS ning o'zi async qobiliyatidan (`epoll`/`kqueue`/`IOCP`) foydalanadi. Socket'lar event-driven tarzda kuzatiladi.
2. **Fayl tizimi (fs), DNS (`dns.lookup`), ba'zi crypto operatsiyalari** — bularda OS yetarli async API bermaydi, shu sabab libuv ularni **thread pool**'da bajaradi.

**Thread pool** — default holda **4 ta thread** (`UV_THREADPOOL_SIZE` orqali 1..1024 oralig'ida o'zgartirish mumkin).

```js
process.env.UV_THREADPOOL_SIZE = 8; // dasturni ishga tushirishdan OLDIN o'rnatilishi shart
```

**⚠️ Ehtiyot bo'l:** Agar bir vaqtda 4 tadan ko'p `fs` yoki `crypto.pbkdf2` chaqirsangiz, 5-chisi navbat kutadi — chunki pool to'lib qoladi. Bu ko'pincha "nega kechikish bor?" degan sirli muammoning sababi bo'ladi.

```js
const crypto = require('crypto');
const start = Date.now();
function hash() {
  crypto.pbkdf2('parol', 'tuz', 100000, 64, 'sha512', () => {
    console.log(Date.now() - start);
  });
}
// 4 ta default pool -> birinchi 4 tasi parallel, 5-chisi keyin
for (let i = 0; i < 5; i++) hash();
// Birinchi 4 ta natija deyarli bir xil vaqtda, 5-chisi ~2x kechikadi
```

---

## Event Loop fazalari (tartib bilan)

**💡 Tushuncha:** Event loop bir necha **faza (phase)**dan iborat, har biri o'z callback navbatiga ega. Har bir "tick"da loop fazalarni **qat'iy tartibda** aylanib chiqadi. Har bir fazada uning navbatidagi callback'lar bajariladi (limitgacha), so'ng keyingi fazaga o'tadi.

Fazalar tartibi:

```
   ┌───────────────────────────┐
┌─>│           timers          │  setTimeout, setInterval callbacklari
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │     pending callbacks     │  ba'zi tizim operatsiyalari (masalan TCP error)
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │       idle, prepare       │  faqat ichki (internal) ishlatish uchun
│  └─────────────┬─────────────┘      ┌───────────────┐
│  ┌─────────────┴─────────────┐      │   incoming:   │
│  │           poll            │<─────┤  connections, │  I/O callbacklarini kutadi/bajaradi
│  └─────────────┬─────────────┘      │   data, etc.  │
│  ┌─────────────┴─────────────┐      └───────────────┘
│  │           check           │  setImmediate callbacklari
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
└──┤      close callbacks      │  socket.on('close'), va h.k.
   └───────────────────────────┘
```

Fazalarning vazifasi:

- **timers** — vaqti yetgan `setTimeout` / `setInterval` callback'lari bajariladi.
- **pending callbacks** — oldingi loop iteratsiyasidan kechiktirilgan ba'zi tizim callback'lari (masalan, ba'zi TCP xatolari).
- **idle, prepare** — faqat libuv ichki ehtiyojlari uchun; siz bularni bevosita ishlatmaysiz.
- **poll** — eng muhim faza. Bu yerda yangi I/O eventlari olinadi va ularning callback'lari bajariladi. Agar navbat bo'sh bo'lsa va `setImmediate` kutmasa, loop shu yerda **bloklanib turishi** (I/O kutib) mumkin.
- **check** — `setImmediate` callback'lari aynan shu fazada ishlaydi.
- **close callbacks** — `socket.on('close', ...)` kabi yopilish callback'lari.

**⚠️ Ehtiyot bo'l:** **Microtask'lar (`process.nextTick` va Promise) FAZA EMAS.** Ular **har bir faza orasida** (aniqrog'i, har bir callback bajarilgandan keyin call stack bo'shaganda) bajariladi. Buni keyingi bo'limda batafsil ko'ramiz — bu eng ko'p chalkashtiradigan joy.

---

## Microtask navbatlari: nextTick vs Promise

**💡 Tushuncha:** Node'da **ikki xil microtask navbati** bor va ular fazalardan ajralib turadi:

1. **`process.nextTick` queue** — eng yuqori ustuvorlik.
2. **Promise microtask queue** (`Promise.then`, `catch`, `finally`, `await`dan keyingi qism, `queueMicrotask`).

**Ustuvorlik tartibi** (har bir operatsiyadan keyin call stack bo'shaganda):

```
1. process.nextTick queue  (TO'LIQ bo'shatiladi)
2. Promise microtask queue (TO'LIQ bo'shatiladi)
3. keyin event loop keyingi fazaga / callback'ga o'tadi
```

Ya'ni: har bir macrotask (timer callback, I/O callback, `setImmediate` va h.k.) bajarilgach, **avval butun `nextTick` navbati**, **keyin butun Promise navbati** bo'shatiladi, **shundan keyingina** keyingi macrotask boshlanadi.

```js
console.log('start');

setTimeout(() => console.log('timeout'), 0);

Promise.resolve().then(() => console.log('promise'));

process.nextTick(() => console.log('nextTick'));

console.log('end');

// Chiqish:
// start
// end
// nextTick   <- microtask, Promise'dan oldin
// promise    <- microtask, timeout'dan oldin
// timeout    <- macrotask, eng oxirida
```

Tushuntirish:
1. `start` va `end` sinxron — darhol chiqadi (call stack).
2. Call stack bo'shadi -> `nextTick` queue bo'shatiladi -> `nextTick`.
3. So'ng Promise queue -> `promise`.
4. Endi event loop timers fazasiga o'tadi -> `timeout`.

**⚠️ Ehtiyot bo'l:** Agar `nextTick` callback ichida yana `nextTick` qo'shsangiz, u **shu microtask sikli ichida** bajariladi (event loop keyingi fazaga o'tmaydi). Bu starvation'ga olib keladi (pastda).

---

## setTimeout vs setImmediate

**💡 Tushuncha:** `setTimeout(fn, 0)` — **timers** fazasida ishlaydi. `setImmediate(fn)` — **check** fazasida ishlaydi. Tartib **kontekstga bog'liq**.

**Holat 1 — main module'dan (I/O sikli tashqarisida) chaqirilganda — tartib NOANIQ:**

```js
setTimeout(() => console.log('timeout'), 0);
setImmediate(() => console.log('immediate'));
// Tartib har xil bo'lishi mumkin: ba'zan timeout oldin, ba'zan immediate.
```

Sababi: `setTimeout(fn, 0)` aslida minimal ~1ms timer'ga aylanadi. Agar event loop boshlangunga qadar 1ms o'tib bo'lsa, timer tayyor bo'ladi va timers fazasi avval ishlaydi. Agar o'tmagan bo'lsa, check fazasi (immediate) avval keladi. Bu mashina yuklamasiga bog'liq — shuning uchun **deterministik emas**.

**Holat 2 — I/O callback ICHIDAN chaqirilganda — tartib ANIQ:**

```js
const fs = require('fs');
fs.readFile(__filename, () => {
  setTimeout(() => console.log('timeout'), 0);
  setImmediate(() => console.log('immediate'));
});
// HAR DOIM:
// immediate
// timeout
```

Sababi: `fs.readFile` callback'i **poll** fazasida bajariladi. Poll fazasidan keyin darhol **check** fazasi keladi -> shu sabab `setImmediate` (check) avval ishlaydi. `setTimeout` esa keyingi loop aylanishidagi timers fazasini kutadi.

**✅ Javob (qoidaviy):** I/O callback ichida `setImmediate` **doim** `setTimeout(0)`dan oldin ishlaydi. Bu yagona deterministik holat.

---

## process.nextTick tuzoqlari va starvation

**💡 Tushuncha:** `process.nextTick` callback'ni "joriy operatsiyadan keyin, lekin event loop keyingi fazaga o'tishidan oldin" bajaradi. U **har qanday Promise microtask'dan ham oldin** ishlaydi. Bu kuchli, lekin xavfli.

**Starvation muammosi** — agar `nextTick` o'zini rekursiv chaqirsa, event loop **hech qachon** keyingi fazaga o'ta olmaydi:

```js
let count = 0;
function recurse() {
  count++;
  process.nextTick(recurse); // nextTick queue hech qachon bo'shamaydi
}
recurse();

setTimeout(() => console.log('Bu HECH QACHON chiqmaydi'), 0);
// Timer ishga tushmaydi, chunki nextTick queue cheksiz to'lib turadi -> I/O starvation
```

**⚠️ Ehtiyot bo'l:** Bir xil rekursiyani `setImmediate` bilan qilsangiz, starvation **bo'lmaydi** — chunki `setImmediate` har bir loop iteratsiyasida bir marta ishlaydi va event loop boshqa fazalarga (timers, poll) "nafas olish" imkonini beradi:

```js
function recurse() {
  setImmediate(recurse); // bu xavfsiz — boshqa fazalar bloklanmaydi
}
recurse();
setTimeout(() => console.log('Bu CHIQADI'), 100); // ishlaydi
```

**Qachon `nextTick` ishlatish kerak:**
- Konstruktor `emit` qilishdan oldin foydalanuvchiga listener qo'shish imkonini berish uchun (async-ni "kafolatlash").
- Xatoni joriy operatsiya tugagandan keyin tashlash uchun.

Aks holda, ko'p hollarda `queueMicrotask` yoki `setImmediate` xavfsizroq tanlovdir.

---

## Async I/O ichki ishlashi

**💡 Tushuncha:** `fs.readFile('a.txt', cb)` chaqirilganda nima sodir bo'ladi — qadamlar:

1. JS funksiya C++ binding'ga o'tadi.
2. libuv operatsiyani **thread pool**'dagi bo'sh thread'ga topshiradi (fs uchun).
3. JS asosiy thread'i **darhol** qaytadi — kod davom etadi (non-blocking).
4. Thread pool'dagi worker thread diskdan o'qiydi (bu bloklash worker'da bo'ladi, main'da emas).
5. O'qish tugagach, libuv natijani **poll** fazasiga "tayyor" deb belgilaydi.
6. Event loop poll fazasiga yetganda, sizning callback'ingiz call stack'ga qo'yiladi va bajariladi.

Tarmoq I/O uchun (masalan, HTTP server) thread pool ishlatilmaydi — OS ning `epoll`/`kqueue`/`IOCP` mexanizmi socket'larni kuzatadi va poll fazasi tayyor socket'larni qayta ishlaydi.

```js
const http = require('http');
http.createServer((req, res) => {
  // Bu callback poll fazasida ishlaydi; thread pool emas, OS event notification
  res.end('salom');
}).listen(3000);
```

**⚠️ Ehtiyot bo'l:** `dns.lookup()` thread pool'ni ishlatadi (getaddrinfo blocking), lekin `dns.resolve()` haqiqiy async tarmoq so'rovi — pool'ni egallamaydi. Ko'p DNS lookup pool'ni to'ldirib, `fs` operatsiyalarini sekinlashtirib qo'yishi mumkin.

---

## Event Loop'ni bloklash (CPU-bound) va yechimlar

**💡 Tushuncha:** Event loop'ni bloklash — bu **asosiy thread'da uzoq sinxron CPU ishi** bajarish. Bu vaqtda hech qanday callback, timer, I/O ishlamaydi — server "muzlaydi" va boshqa so'rovlarga javob bermaydi.

```js
// YOMON: katta sinxron hisob butun serverni bloklaydi
app.get('/hash', (req, res) => {
  let result = 0;
  for (let i = 0; i < 1e10; i++) result += i; // ~bir necha soniya BLOK
  res.json({ result });
});
// Bu so'rov davomida serveringiz BOSHQA HECH KIMGA javob bermaydi
```

Bloklashni aniqlash belgilari: ortib borayotgan latency, "event loop lag" metrikasi, health-check'lar timeout bo'lishi.

**Yechimlar:**

1. **`worker_threads`** — CPU-bound ishni alohida thread'ga ko'chirish. Asosiy event loop bo'sh qoladi.

```js
const { Worker } = require('worker_threads');
function heavyTask(data) {
  return new Promise((resolve, reject) => {
    const worker = new Worker('./heavy-worker.js', { workerData: data });
    worker.on('message', resolve);
    worker.on('error', reject);
  });
}
```

2. **Ishni bo'laklarga bo'lish (chunking)** — katta sikni `setImmediate` orqali qismlarga ajratish, har bo'lak orasida event loop'ga nafas berish.

```js
function processBigArray(arr, i = 0) {
  const end = Math.min(i + 1000, arr.length);
  for (; i < end; i++) { /* ish */ }
  if (i < arr.length) setImmediate(() => processBigArray(arr, i));
}
```

3. **`child_process`** — alohida Node jarayoniga ko'chirish (to'liq izolyatsiya kerak bo'lganda).
4. **`cluster`** — bir nechta jarayon ishga tushirib, yukni CPU yadrolari bo'yicha taqsimlash.
5. **Native/optimallashtirilgan kutubxonalar** yoki tashqi servis (masalan, og'ir hisobni queue + worker servisga uzatish).

**⚠️ Ehtiyot bo'l:** `JSON.parse`/`JSON.stringify` katta obyektlarda, `bcrypt`ning sinxron varianti, og'ir regex (ReDoS), katta massiv `sort` — bularning hammasi yashirin bloklash manbalari.

---

## Qiyin tartib misoli (qadama-qadam)

Bu intervyularning klassik "tuzoq" savoli. Aralashmani diqqat bilan tahlil qilamiz:

```js
console.log('1: start');

setTimeout(() => {
  console.log('2: setTimeout');
  Promise.resolve().then(() => console.log('3: promise ichida timeout'));
  process.nextTick(() => console.log('4: nextTick ichida timeout'));
}, 0);

setImmediate(() => {
  console.log('5: setImmediate');
});

Promise.resolve().then(() => console.log('6: promise'));

process.nextTick(() => console.log('7: nextTick'));

console.log('8: end');
```

**Qadama-qadam tahlil:**

**1-bosqich — sinxron kod (call stack):**
- `1: start` chiqadi.
- `setTimeout` ro'yxatga olinadi (timers fazasiga).
- `setImmediate` ro'yxatga olinadi (check fazasiga).
- `Promise.then` Promise microtask queue'ga tushadi.
- `process.nextTick` nextTick queue'ga tushadi.
- `8: end` chiqadi.

Hozircha chiqdi: `1: start`, `8: end`.

**2-bosqich — call stack bo'shadi, microtask'lar (event loop fazaga o'tishdan OLDIN):**
- Avval **nextTick queue**: `7: nextTick`.
- So'ng **Promise queue**: `6: promise`.

Chiqdi: `7: nextTick`, `6: promise`.

**3-bosqich — event loop fazalarni boshlaydi. timers fazasi:**
- `setTimeout` callback ishlaydi -> `2: setTimeout`.
- Uning ichida `Promise.then` -> Promise queue'ga, `nextTick` -> nextTick queue'ga qo'shiladi.
- Callback tugadi, call stack bo'shadi -> microtask'lar bajariladi:
  - nextTick avval: `4: nextTick ichida timeout`.
  - keyin Promise: `3: promise ichida timeout`.

Chiqdi: `2: setTimeout`, `4: nextTick ichida timeout`, `3: promise ichida timeout`.

**4-bosqich — check fazasi:**
- `setImmediate` callback -> `5: setImmediate`.

**Yakuniy to'liq chiqish:**

```
1: start
8: end
7: nextTick
6: promise
2: setTimeout
4: nextTick ichida timeout
3: promise ichida timeout
5: setImmediate
```

**✅ Javob (umumiy formula):**
1. Barcha **sinxron** kod ishlaydi.
2. Har bir macrotask oxirida: **nextTick queue (to'liq)** -> **Promise queue (to'liq)**.
3. Macrotask'lar faza tartibida: **timers** -> **poll/pending** -> **check (setImmediate)** -> ...

**⚠️ Ehtiyot bo'l:** Yuqoridagi misolda `setTimeout` va `setImmediate` main module'dan chaqirilgan, shuning uchun ularning bir-biriga nisbatan aniq tartibi (timers vs check) mashina yuklamasiga bog'liq edge-case'ga ega bo'lishi mumkin. Lekin bu yerda `setTimeout` callback ichida yana microtask'lar borligi tahlilning asosiy nuqtasi. Agar `setTimeout` I/O callback ichida bo'lganda edi, `setImmediate` aniq oldin chiqardi.

---

## Interview Q&A

### ❓ Event loop nima va u nima uchun kerak?

**✅ Javob:** Event loop — libuv ta'minlaydigan mexanizm bo'lib, u callback'larni navbatlardan olib, call stack bo'sh bo'lganda bajaradi. U Node'ga bitta thread ustida ko'p sonli async operatsiyalarni (I/O, timer'lar) boshqarishga imkon beradi: og'ir kutishlar OS yoki thread pool'ga topshiriladi, asosiy thread esa bo'sh qoladi.

### ❓ Node "single-threaded" deyiladi, lekin bu to'g'rimi?

**✅ Javob:** Yarim to'g'ri. **JavaScript bajarilishi** bitta thread'da bo'ladi (V8 main thread). Lekin libuv ostida **thread pool** (default 4 thread) `fs`, `crypto`, `dns.lookup` kabi operatsiyalar uchun ishlaydi, OS esa tarmoq I/O ni o'z thread'larida boshqaradi. `worker_threads` bilan haqiqiy parallel JS ham mumkin. Demak: "JS kodi single-threaded, lekin Node butunlay single-threaded emas."

### ❓ Event loop fazalarini tartib bilan ayting.

**✅ Javob:** timers -> pending callbacks -> idle/prepare (internal) -> poll -> check -> close callbacks. Har bir faza o'z navbatiga ega; loop ularni shu tartibda aylanib chiqadi. Microtask'lar (nextTick, Promise) faza emas — ular har bir callback orasida bajariladi.

### ❓ `process.nextTick` va `Promise.then` orasidagi farq va ustuvorlik?

**✅ Javob:** Ikkalasi ham microtask, lekin `nextTick` queue **Promise queue'dan oldin** to'liq bo'shatiladi. Ya'ni har bir operatsiyadan keyin: avval barcha `nextTick`, keyin barcha Promise microtask, keyin event loop keyingi fazaga o'tadi.

### ❓ `setTimeout(fn, 0)` va `setImmediate(fn)` qaysi biri oldin ishlaydi?

**✅ Javob:** Main module'dan chaqirilsa — **noaniq** (timer'ning ~1ms resolution'i va mashina yuklamasiga bog'liq). I/O callback ichidan chaqirilsa — **doim `setImmediate` oldin**, chunki poll fazasidan keyin darhol check fazasi keladi.

### ❓ Microtask va macrotask farqi nima?

**✅ Javob:** Macrotask — timer callback, I/O callback, `setImmediate` kabi event loop fazalaridagi vazifalar. Microtask — `process.nextTick`, Promise reaksiyalari, `queueMicrotask`. Har bir macrotask'dan keyin **barcha** microtask'lar bo'shatiladi, shundan so'nggina keyingi macrotask boshlanadi.

### ❓ `await` event loop nuqtai nazaridan nima qiladi?

**✅ Javob:** `await` funksiyani "pauza" qiladi va qolgan qismini Promise microtask sifatida rejalashtiradi. `await expr`dan keyingi kod, agar `expr` allaqachon hal bo'lgan bo'lsa ham, **microtask** sifatida (sinxron emas) ishlaydi. Shuning uchun `await`dan keyingi qator joriy sinxron koddan keyin, lekin timer'lardan oldin ishlaydi.

### ❓ Thread pool nima uchun kerak va default o'lchami qancha?

**✅ Javob:** Ba'zi operatsiyalar (`fs`, `dns.lookup`, ba'zi `crypto`) OS tomonidan async API bermaydi, shuning uchun libuv ularni thread pool'da bloklamasdan bajaradi. Default **4 thread**; `UV_THREADPOOL_SIZE` (1..1024) bilan o'zgartiriladi.

### ❓ Tarmoq I/O thread pool ishlatadimi?

**✅ Javob:** Yo'q. TCP/HTTP kabi tarmoq I/O OS ning event notification mexanizmidan (`epoll`/`kqueue`/`IOCP`) foydalanadi — poll fazasi tayyor socket'larni qayta ishlaydi. Thread pool faqat fayl tizimi, DNS lookup va ayrim crypto uchun.

### ❓ Event loop'ni nima bloklaydi va qanday aniqlaysiz?

**✅ Javob:** Uzoq sinxron CPU ishi (katta sikl, og'ir `JSON.parse`, sinxron crypto, ReDoS regex). Belgilari: ortib borayotgan latency, health-check timeout. Aniqlash uchun "event loop lag" o'lchanadi (masalan, `perf_hooks` yoki kutubxona orqali). Yechim: `worker_threads`, chunking, `child_process`, `cluster`.

### ❓ `process.nextTick` starvation nima?

**✅ Javob:** Agar `nextTick` o'zini rekursiv qo'shsa, queue hech qachon bo'shamaydi va event loop keyingi fazaga (timers, poll) o'ta olmaydi — I/O va timer'lar "och qoladi" (starvation). `setImmediate` esa har iteratsiyada bir marta ishlagani uchun bu muammoni keltirmaydi.

### ❓ Nega `fs.readFileSync` serverni xavfli qiladi?

**✅ Javob:** U asosiy thread'ni o'qish tugagunicha bloklaydi. So'rovni boshqarayotgan vaqtda boshqa barcha ulanishlar "muzlaydi". Server kontekstida har doim async (`fs.readFile` yoki `fs.promises`) variant ishlatilishi kerak.

### ❓ `queueMicrotask` qachon ishlatiladi?

**✅ Javob:** `queueMicrotask` callback'ni Promise microtask queue'ga qo'yadi — `nextTick`dan keyin, lekin keyingi fazadan oldin. U "joriy ishdan keyin, lekin I/O dan oldin" mantiqiy ishlarni rejalashtirishning standart (Promise'ga asoslanmagan) usuli. `nextTick`dan ko'ra kamroq "agressiv" va starvation xavfi pastroq.

### ❓ `setInterval` va event loop bloklanishi qanday bog'liq?

**✅ Javob:** `setInterval` callback'ning **boshlanish vaqti** kafolatlanmaydi — agar event loop band bo'lsa (oldingi callback uzoq ishlasa), interval kechikadi yoki "yig'ilib" ketadi. Interval haqiqiy real-time emas; u shunchaki "tayyor bo'lgach, timers fazasida ishga tushadi".

### ❓ "Event loop lag" nimani anglatadi?

**✅ Javob:** Bu — timer rejalashtirilgan vaqt bilan haqiqiy bajarilgan vaqt orasidagi farq. Yuqori lag asosiy thread band (bloklangan) ekanligini ko'rsatadi. Monitoring uchun muhim metrik; o'sib borishi performance muammosi signalidir.

---

## Masalalar

> Yechimlar: [solutions/backend/03-event-loop.md](../solutions/backend/03-event-loop.md)

1. Quyidagi kodning chiqish tartibini yozing va har bir qatorni izohlang:
   ```js
   console.log('A');
   setTimeout(() => console.log('B'), 0);
   Promise.resolve().then(() => console.log('C'));
   process.nextTick(() => console.log('D'));
   console.log('E');
   ```

2. `fs.readFile` callback ICHIDA `setTimeout(0)` va `setImmediate` yozing. Qaysi biri oldin chiqadi va NEGA? Tushuntiring.

3. `process.nextTick`ni rekursiv chaqirib, `setTimeout`ning hech qachon ishlamasligini ko'rsatadigan kod yozing. So'ng uni `setImmediate` bilan almashtirib, muammoni hal qiling.

4. CPU-bound funksiya (masalan, 1..1e9 yig'indisi) butun event loop'ni bloklayotganini ko'rsating. So'ng uni `worker_threads` yoki chunking (`setImmediate`) orqali bloklamaydigan qiling.

5. Quyidagi aralashmaning to'liq chiqishini bashorat qiling:
   ```js
   setTimeout(() => console.log('T1'), 0);
   setImmediate(() => console.log('I1'));
   Promise.resolve().then(() => {
     console.log('P1');
     process.nextTick(() => console.log('N1'));
   });
   process.nextTick(() => console.log('N2'));
   ```

6. `UV_THREADPOOL_SIZE`ni 1 ga o'rnatib, 4 ta parallel `crypto.pbkdf2` chaqiring va ularning tugash vaqtlarini o'lchang. Pool o'lchamining ta'sirini tushuntiring.

7. `async`/`await` ishlatib, `await`dan keyingi kod sinxron emas, balki microtask sifatida ishlashini ko'rsatadigan kichik dastur yozing (`console.log` tartibida).

8. Event loop lag'ni o'lchaydigan oddiy funksiya yozing: har 100ms da timer qo'ying va haqiqiy o'tgan vaqt bilan kutilgan vaqt farqini chiqaring. Keyin sun'iy bloklash qo'shib, lag o'sishini ko'rsating.

9. `dns.lookup` ko'p marta chaqirilganda thread pool to'lib, `fs` operatsiyalari sekinlashishini ko'rsatadigan eksperiment dizayn qiling (kod + kutilgan natija).

10. Nima uchun quyidagi server kodida bitta og'ir so'rov boshqa barcha so'rovlarni bloklaydi? Muammoni tasvirlang va kamida 2 ta yechim taklif qiling:
    ```js
    http.createServer((req, res) => {
      let s = 0;
      for (let i = 0; i < 5e9; i++) s += i;
      res.end(String(s));
    }).listen(3000);
    ```

---

← [Backend bo'limiga qaytish](./README.md)
