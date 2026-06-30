# Node.js asoslari

Node.js aslida nima — V8, libuv va C++ binding'lardan tashkil topgan runtime; uning single-threaded modeli va u haqdagi keng tarqalgan tushunmovchilik; module system'lar (CommonJS vs ESM); `process`, `Buffer`, `fs`, `path` kabi asosiy qismlar; CPU-bound ish uchun `worker_threads`/`cluster`/`child_process`; va npm ekotizimi. Bu — har bir Node.js intervyusi siz bilishingizni kutadigan poydevor.

## Mundarija

- [Node.js nima (V8 + libuv + bindings)](#nodejs-nima-v8--libuv--bindings)
- [Nega serverda JavaScript](#nega-serverda-javascript)
- [Single-threaded model va tushunmovchilik](#single-threaded-model-va-tushunmovchilik)
- [Module system: CommonJS vs ESM](#module-system-commonjs-vs-esm)
- [Module resolution va require cache](#module-resolution-va-require-cache)
- [global object](#global-object)
- [process: argv, env, exit, nextTick](#process-argv-env-exit-nexttick)
- [Buffer](#buffer)
- [fs: sync vs async](#fs-sync-vs-async)
- [path moduli](#path-moduli)
- [worker_threads, cluster, child_process](#worker_threads-cluster-child_process)
- [npm, package.json, semver, lockfile](#npm-packagejson-semver-lockfile)
- [Error-first callback va util.promisify](#error-first-callback-va-utilpromisify)
- [Intervyu savollari (Q&A)](#intervyu-savollari-qa)
- [Masalalar](#masalalar)

---

## Node.js nima (V8 + libuv + bindings)

**💡 Tushuncha:** **Node.js** — bu JavaScript'ni server muhitida ishlatuvchi **runtime**. U til emas (til — JavaScript), balki JavaScript'ga "qo'l-oyoq" beruvchi muhit. Uchta asosiy qismdan iborat:

- **V8** — Google'ning JavaScript engine'i (Chrome'dagi bilan bir xil). JS kodini mashina kodiga kompilyatsiya qilib bajaradi.
- **libuv** — C tilidagi kutubxona. **Event loop**ni va asinxron I/O (fayl, tarmoq) bilan ishlashni, hamda **thread pool**ni ta'minlaydi. Node.js'ning "non-blocking" sehri shu yerda.
- **C++ binding'lar** — V8 (JS dunyosi) bilan libuv va OS imkoniyatlari (JS'da bo'lmagan) orasidagi ko'prik. `fs`, `net`, `crypto` kabi modullar ostida shular turadi.

```text
  Sizning JS kodingiz
        │
        ▼
  ┌──────────────┐     ┌──────────────┐
  │     V8       │ ←→  │  C++ bindings │
  │ (JS bajaradi)│     └──────┬───────┘
  └──────────────┘            │
                       ┌──────▼───────┐
                       │    libuv     │  (event loop + thread pool + OS I/O)
                       └──────────────┘
```

**⚠️ Ehtiyot bo'l:** "Node.js — bu til" deyish — xato. Node.js runtime, JavaScript esa til. Shuningdek V8 — faqat JS'ni bajaradi; fayl o'qish kabi ishlar V8'da emas, libuv/OS'da bo'ladi.

---

## Nega serverda JavaScript

**💡 Tushuncha:** Tarixan JavaScript faqat brauzerda yashardi. Node.js (2009) uni serverga olib chiqdi. Sabablari:

- **Bitta til** — frontend va backend bir tilda (`full-stack JS`), jamoa bilim almashishi oson.
- **Non-blocking I/O** — Node.js ko'p bir vaqtli ulanishni (concurrent connections) kam resurs bilan, ya'ni I/O-og'ir vazifalarni (API, chat, real-time) samarali eplaydi.
- **npm** — dunyodagi eng katta paket ekotizimi.
- **JSON** — JS'ning tabiiy formati, web API'lar uchun ideal.

**⚠️ Ehtiyot bo'l:** Node.js I/O-bound ishlar uchun a'lo, lekin **CPU-bound** (og'ir hisob-kitob, video kodlash) ishlar uchun yomon tanlov — ular bitta thread'ni bloklaydi. Bunda `worker_threads` yoki boshqa til kerak bo'ladi.

---

## Single-threaded model va tushunmovchilik

**💡 Tushuncha:** Node.js sizning JavaScript kodingizni **bitta thread**da (event loop thread) bajaradi. Ya'ni ikki JS funksiyasi bir vaqtning o'zida *parallel* ishlamaydi. Lekin bu Node.js "sekin" yoki "bir vaqtda bitta ish qiladi" degani EMAS.

Sir shundaki: I/O operatsiyalari (fayl, tarmoq, DB) JS thread'ini *bloklamaydi*. Node.js ularni OS/libuv'ga topshiradi va boshqa ishni davom ettiradi; tayyor bo'lganda callback chaqiriladi.

```js
console.log('1');
setTimeout(() => console.log('3'), 0);   // keyinroq, navbatga tushadi
console.log('2');
// Natija: 1, 2, 3 — setTimeout JS'ni bloklamaydi
```

**Tushunmovchilik:** "Single-threaded = bir vaqtda bir ish". Aniqrog'i: *JavaScript kodi* bitta thread'da, ammo I/O **parallel** (libuv thread pool va OS orqali) bajariladi. Demak Node.js minglab ulanishni bir vaqtda eplay oladi.

```js
// Bu — bloklaydi! (CPU-bound, butun event loop muzlaydi)
function fib(n) { return n < 2 ? n : fib(n - 1) + fib(n - 2); }
fib(45);   // ⚠️ shu hisob tugamaguncha boshqa hech qaysi so'rov javob olmaydi
```

**⚠️ Ehtiyot bo'l:** Og'ir sinxron hisob (katta loop, sync kriptografiya, `JSON.parse` ulkan fayl) event loop'ni bloklaydi va *barcha* client'larni kutdiradi. Bunday ishlarni `worker_threads`'ga bering.

---

## Module system: CommonJS vs ESM

**💡 Tushuncha:** Node.js ikki module tizimini qo'llaydi:

**CommonJS (CJS)** — Node.js'ning klassik tizimi. `require()` bilan import, `module.exports` bilan eksport. Sinxron yuklanadi.

```js
// math.js
function add(a, b) { return a + b; }
module.exports = { add };
// yoki: exports.add = add;

// app.js
const { add } = require('./math');
console.log(add(2, 3));   // 5
```

**ES Modules (ESM)** — JavaScript'ning standart, zamonaviy tizimi. `import`/`export`. Statik tahlil qilinadi (tree-shaking imkoni).

```js
// math.mjs
export function add(a, b) { return a + b; }
export default function multiply(a, b) { return a * b; }

// app.mjs
import multiply, { add } from './math.mjs';
```

| | CommonJS | ESM |
|---|---|---|
| Import | `require()` | `import` |
| Export | `module.exports` | `export` |
| Yuklash | sinxron | asinxron |
| `__dirname`/`__filename` | bor | yo'q (`import.meta.url`) |
| Top-level `await` | yo'q | bor |
| Faylga ishora | default `.js` | `.mjs` yoki `"type":"module"` |

**ESM'ni yoqish:** `.mjs` kengaytmasi yoki `package.json`'da `"type": "module"`.

**⚠️ Ehtiyot bo'l:** ESM va CJS'ni aralashtirib ishlatish nozik. ESM'dan CJS'ni `import` qilsa bo'ladi, lekin CJS'dan ESM'ni `require` qila olmaysiz (ESM async). `module.exports` (CJS) va `export default` (ESM) — bir narsa emas. `require is not defined in ES module scope` — `"type":"module"` qo'ygach `require` ishlatib chiqadigan tipik xato.

---

## Module resolution va require cache

**💡 Tushuncha:** `require('x')` chaqirilganda Node.js `x`ni qanday topadi (resolution):

1. **Core module** bo'lsa (`fs`, `path`, `http`) — to'g'ridan-to'g'ri ichkaridan oladi.
2. **Yo'l** (`./`, `../`, `/`) bo'lsa — shu yo'ldan fayl izlaydi: avval aniq nom, keyin `.js`, `.json`, `.node` kengaytmalari, keyin papka bo'lsa `index.js` yoki `package.json`'dagi `main`.
3. Aks holda — **`node_modules`** papkasini joriy papkadan boshlab yuqoriga (parent) qarab izlaydi, ildizgacha.

**require cache:** Modul *birinchi* `require`da yuklanadi va bajariladi; natija **cache**lanadi. Keyingi `require`lar yangidan bajarmaydi — cache'dan o'sha *bir xil* obyektni qaytaradi (singleton xatti-harakat).

```js
// counter.js
let count = 0;
module.exports = { inc: () => ++count, get: () => count };

// a.js
const c = require('./counter');
c.inc();
console.log(c.get());   // 1

// b.js — a.js'dan keyin require qilingan bo'lsa, BIR XIL obyekt
const c2 = require('./counter');
console.log(c2.get());  // 1 (yangidan 0 emas! cache'dan o'sha obyekt)
```

**⚠️ Ehtiyot bo'l:** Modul faqat bir marta ishga tushadi. Agar modul ichida side-effect (masalan DB ulanish) bo'lsa, u necha marta `require` qilinsa ham bir marta sodir bo'ladi. Bu odatda foydali (singleton), lekin "har safar yangi nusxa" kutsangiz — xato manbai. Cache'ni `delete require.cache[require.resolve('./x')]` bilan tozalash mumkin (kamdan-kam, ehtiyotkorlik bilan).

---

## global object

**💡 Tushuncha:** Brauzerda global obyekt `window`, Node.js'da esa `global` (zamonaviy standart: `globalThis`). Node.js'da tez-tez ishlatiladigan "global"lar aslida har modulga beriladi:

```js
console.log(__dirname);    // joriy fayl papkasi (CJS)
console.log(__filename);   // joriy faylning to'liq yo'li (CJS)
console.log(process.pid);  // process ID
setTimeout(fn, 1000);      // timer'lar global
setInterval(fn, 1000);
Buffer.from('salom');      // Buffer global
```

**⚠️ Ehtiyot bo'l:** `__dirname` va `__filename` — bular *modul-darajadagi* o'zgaruvchilar, haqiqiy global emas, va ESM'da mavjud emas (`import.meta.url` ishlatiladi). Brauzer odatlari bilan `var x = 1` yozsangiz, u global emas, *modul* doirasida qoladi — Node.js'da har fayl o'z scope'iga ega.

---

## process: argv, env, exit, nextTick

**💡 Tushuncha:** `process` — global obyekt; u ishlayotgan Node.js process'i haqida ma'lumot beradi va uni boshqaradi.

```js
// argv — komanda qatori argumentlari
// $ node app.js --port 8080
console.log(process.argv);
// ['/usr/bin/node', '/path/app.js', '--port', '8080']
// [0] = node yo'li, [1] = skript yo'li, [2..] = argumentlar

// env — environment variable'lar
console.log(process.env.NODE_ENV);   // 'production'
const port = process.env.PORT || 3000;

// exit — process'ni to'xtatish (0 = muvaffaqiyat, 0 emas = xato)
process.exit(1);

// nextTick — joriy operatsiyadan keyin, event loop davom etishidan OLDIN
process.nextTick(() => console.log('nextTick'));
Promise.resolve().then(() => console.log('promise'));
console.log('sync');
// Natija: sync → nextTick → promise
```

**Foydali event'lar:**

```js
process.on('exit', (code) => console.log('chiqyapti, kod:', code));
process.on('uncaughtException', (err) => { /* oxirgi himoya */ });
process.on('SIGTERM', () => { /* graceful shutdown */ });
```

**⚠️ Ehtiyot bo'l:** `process.nextTick` callback'lari Promise microtask'laridan ham *oldin* ishlaydi. Uni rekursiv chaqirsangiz (`nextTick` ichida yana `nextTick`), event loop'ni "och qoldirib" (I/O'ga navbat bermay) bloklab qo'yishingiz mumkin. `process.exit()` ham xavfli — kutilayotgan I/O (log yozish, fayl) tugamasdan jarayonni darrov o'ldiradi.

---

## Buffer

**💡 Tushuncha:** **Buffer** — Node.js'da xom (raw) binar ma'lumotni saqlovchi qism. JavaScript string'lari matn (Unicode) uchun; lekin fayl, tarmoq, kriptografiya — bayt (byte) bilan ishlaydi. Buffer — V8 heap'idan tashqaridagi belgilangan o'lchamli xotira bo'lagi.

```js
const buf = Buffer.from('salom', 'utf8');
console.log(buf);              // <Buffer 73 61 6c 6f 6d>
console.log(buf.length);       // 5 (bayt soni)
console.log(buf.toString('utf8'));   // 'salom'
console.log(buf.toString('hex'));    // '73616c6f6d'
console.log(buf.toString('base64')); // 'c2Fsb20='

const empty = Buffer.alloc(4);       // 4 bayt, nol bilan to'ldirilgan
```

**⚠️ Ehtiyot bo'l:** `Buffer.from(string)` da string `.length` bayt sonига teng emas — UTF-8'da bir belgi bir necha bayt bo'lishi mumkin (masalan, "ё" yoki emoji). `Buffer.allocUnsafe(n)` tezroq, lekin xotirani tozalamaydi — eski ma'lumot "qalashib" qolishi mumkin, shuning uchun ishonchli holatda `Buffer.alloc` ishlating.

---

## fs: sync vs async

**💡 Tushuncha:** `fs` (file system) moduli faylga ishlash uchun uchta uslub beradi:

```js
const fs = require('fs');
const fsp = require('fs/promises');

// 1. Async callback (error-first) — bloklamaydi
fs.readFile('data.txt', 'utf8', (err, data) => {
  if (err) return console.error(err);
  console.log(data);
});

// 2. Sync — BLOKLAYDI (butun event loop kutadi)
const data = fs.readFileSync('data.txt', 'utf8');
console.log(data);

// 3. Promise-based (zamonaviy, async/await bilan)
async function read() {
  const data = await fsp.readFile('data.txt', 'utf8');
  console.log(data);
}
```

**⚠️ Ehtiyot bo'l:** `readFileSync` server *so'rovini ishlovchi* kodda ishlatilsa — falokat: u event loop'ni bloklaydi va shu vaqtda BARCHA client'lar kutadi. Sync versiyalar faqat dastur *ishga tushishida* (config o'qish) yoki bir martalik skriptlarда maqbul. Server logikasida har doim async/promise versiyani ishlating.

---

## path moduli

**💡 Tushuncha:** `path` — fayl yo'llari bilan xavfsiz ishlash uchun. Windows (`\`) va Unix (`/`) ajratuvchilari farqini o'zi hal qiladi — yo'llarni qo'lда string sifatida birlashtirmang.

```js
const path = require('path');

path.join('users', 'ali', 'file.txt');   // 'users/ali/file.txt' (OS'ga mos)
path.resolve('src', 'app.js');           // absolyut yo'l: '/.../src/app.js'
path.basename('/a/b/file.txt');          // 'file.txt'
path.dirname('/a/b/file.txt');           // '/a/b'
path.extname('file.txt');                // '.txt'
path.join(__dirname, 'config', 'db.json'); // ishonchli absolyut qurish
```

**⚠️ Ehtiyot bo'l:** Yo'llarni `'a/' + 'b'` kabi qo'lда yopishtirmang — ortiqcha yoki yetishmayotgan `/`, OS farqi va xavfsizlik (path traversal `../../`) muammolari chiqadi. `path.join`/`path.resolve` ishlating. Foydalanuvchidan kelgan yo'lni hech qachon to'g'ridan-to'g'ri ishonmang.

---

## worker_threads, cluster, child_process

**💡 Tushuncha:** Node.js bitta thread'da JS bajaradi, lekin ko'p yadroli (multi-core) protsessordan foydalanish va CPU-bound ishlarni bloklamasdan bajarish uchun uchta vosita bor:

- **`worker_threads`** — *bir process ichida* qo'shimcha thread'lar. CPU-og'ir ishlar (hisob, parsing) uchun ideal; thread'lar xotirani (`SharedArrayBuffer`) bo'lishishi mumkin.
- **`cluster`** — bitta server kodining *bir nechta process* (har yadroga bittadan) nusxasini ishga tushiradi va portni baham ko'radi. Web serverni multi-core'da scale qilish uchun.
- **`child_process`** — *boshqa dastur* yoki alohida skriptni ishga tushiradi (`spawn`, `exec`, `fork`). Masalan, `ffmpeg` chaqirish.

```js
// worker_threads — CPU-og'ir ishni event loop'dan tashqariga chiqarish
const { Worker } = require('worker_threads');
const worker = new Worker('./heavy-task.js');
worker.on('message', (result) => console.log('Natija:', result));
worker.postMessage({ n: 45 });
```

```js
// cluster — har CPU yadrosida bitta server nusxasi
const cluster = require('cluster');
const os = require('os');
if (cluster.isPrimary) {
  os.cpus().forEach(() => cluster.fork());   // worker'lar
} else {
  require('./server');   // har worker o'z serverini ishlatadi
}
```

| Vosita | Nima yaratadi | Qachon |
|---|---|---|
| `worker_threads` | thread (bir process) | CPU-bound hisob |
| `cluster` | process (server nusxalari) | Web serverni scale qilish |
| `child_process` | alohida process/dastur | tashqi buyruq/skript |

**⚠️ Ehtiyot bo'l:** `worker_threads` xotira bo'lishadi, `cluster`/`child_process` esa alohida xotirali process'lar — ular orasida ma'lumot xabar (message) orqali, *nusxalanib* uzatiladi (katta data uchun qimmat). Cluster I/O-bound serverni tezlashtiradi, lekin bitta og'ir CPU so'rovni tezlashtirmaydi (uni `worker_threads` hal qiladi).

---

## npm, package.json, semver, lockfile

**💡 Tushuncha:** **npm** — Node.js paket menejeri. **`package.json`** — loyiha "pasporti": nomi, versiyasi, skriptlari va bog'liqliklari (dependencies).

```json
{
  "name": "my-api",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js",
    "test": "jest"
  },
  "dependencies": {
    "express": "^4.18.2"
  },
  "devDependencies": {
    "jest": "^29.0.0",
    "nodemon": "^3.0.0"
  }
}
```

**dependencies vs devDependencies:**
- **`dependencies`** — ilova *ishlashi* uchun kerak (express, pg). Prod'da o'rnatiladi.
- **`devDependencies`** — faqat *ishlab chiqish/test* uchun (jest, nodemon, eslint). Prod'da o'rnatilmaydi (`npm install --production`).

**Semver (`MAJOR.MINOR.PATCH`):** masalan `4.18.2`.
- **MAJOR** (4) — buzuvchi o'zgarish (breaking).
- **MINOR** (18) — orqaga mos yangi funksiya.
- **PATCH** (2) — bug tuzatish.
- `^4.18.2` — `4.x.x` (MAJOR qotgan): `4.99.0` bo'ladi, `5.0.0` yo'q.
- `~4.18.2` — `4.18.x` (faqat PATCH).
- `4.18.2` — aniq versiya.

**Lockfile (`package-lock.json`):** har bir paketning (va ularning bog'liqliklarining) *aniq* versiyasini va manzilini qotirib yozadi. Shu fayl tufayli har bir mashinada *aynan bir xil* `node_modules` o'rnatiladi — "menda ishlardi" muammosini bartaraf etadi. Lockfile **Git'ga commit qilinadi**.

```bash
npm install          # package.json'ga ko'ra o'rnatadi, lockfile'ni yangilaydi
npm ci               # lockfile'ga ko'ra ANIQ o'rnatadi (CI/prod uchun, tez va aniq)
npm install express  # yangi dependency qo'shadi
npm install -D jest  # devDependency qo'shadi
```

**node_modules resolution:** `require('express')` qilsangiz, Node.js joriy papkadagi `node_modules`'dan boshlab, topilmasa parent papkalarga ko'tarilib izlaydi.

**⚠️ Ehtiyot bo'l:** `node_modules`'ni Git'ga *qo'ymang* (`.gitignore`da), lekin `package-lock.json`'ni *albatta* commit qiling. CI'da `npm install` emas, `npm ci` ishlating — u lockfile'ni qat'iy hurmat qiladi va aniq, takrorlanadigan build beradi. `^` (caret) tufayli `npm install` ba'zan kutilmagan yangi minor versiyani tortib kelishi mumkin — lockfile shuni nazoratда ushlaydi.

---

## Error-first callback va util.promisify

**💡 Tushuncha:** Promise'lardan oldin Node.js'ning asinxron uslubi **error-first callback** edi (Node.js konvensiyasi): callback'ning *birinchi* argumenti — xato (yo'q bo'lsa `null`), ikkinchisi — natija.

```js
fs.readFile('data.txt', 'utf8', (err, data) => {
  if (err) {                 // ❗ doim avval xatoni tekshir
    console.error('Xato:', err);
    return;                  // return unutilmasin!
  }
  console.log(data);         // natija
});
```

**`util.promisify`** — error-first callback uslubidagi funksiyani Promise qaytaradigan funksiyaga aylantiradi (async/await bilan ishlatish uchun):

```js
const util = require('util');
const fs = require('fs');

const readFileAsync = util.promisify(fs.readFile);

async function main() {
  try {
    const data = await readFileAsync('data.txt', 'utf8');
    console.log(data);
  } catch (err) {
    console.error('Xato:', err);   // rad etilgan promise — catch'ga tushadi
  }
}
```

**⚠️ Ehtiyot bo'l:** Error-first callback'da `if (err) return` dagi `return`ni unutsangiz, kod xatodan keyin ham davom etib, `data` `undefined` bo'lganda ishlab, ikkinchi xatoga olib keladi. Ko'p paketlar allaqachon promise versiyasini beradi (`fs/promises`) — qo'lда `promisify` qilishdan oldin shuni tekshiring.

---

## Intervyu savollari (Q&A)

### ❓ Node.js nima? Til emasligini tushuntiring.

**✅ Javob:** Node.js — JavaScript'ni serverda ishlatuvchi runtime. U til emas (til — JS). Uch qismdan iborat: V8 (JS engine), libuv (event loop + async I/O + thread pool), va C++ binding'lar (V8 bilan OS imkoniyatlari orasidagi ko'prik).

### ❓ libuv nima vazifa bajaradi?

**✅ Javob:** libuv — C kutubxonasi; u event loop'ni, asinxron I/O'ni (fayl, tarmoq) va thread pool'ni ta'minlaydi. Node.js'ning non-blocking xatti-harakati shu yerda: I/O OS/libuv'ga topshiriladi, JS thread bo'shab boshqa ishni davom ettiradi.

### ❓ "Single-threaded" deganda nima nazarda tutiladi? Eng katta tushunmovchilik nima?

**✅ Javob:** *JavaScript kodingiz* bitta thread'da (event loop) bajariladi. Tushunmovchilik — "demak bir vaqtda bitta ish". Aslida I/O parallel (libuv thread pool va OS orqali) bajariladi, shuning uchun Node.js minglab ulanishni bir vaqtda eplaydi. Faqat *CPU-bound* JS bloklaydi.

### ❓ CommonJS va ESM farqlari?

**✅ Javob:** CommonJS — `require`/`module.exports`, sinxron, `__dirname` bor. ESM — `import`/`export`, asinxron, statik tahlil (tree-shaking), top-level `await` bor, `__dirname` yo'q (`import.meta.url`). ESM `.mjs` yoki `"type":"module"` bilan yoqiladi. CJS'dan ESM'ni `require` qilib bo'lmaydi.

### ❓ require cache nima va nima uchun muhim?

**✅ Javob:** Modul birinchi `require`da bajariladi va natija cache'lanadi; keyingi `require`lar *o'sha bir xil* obyektni qaytaradi (singleton). Shuning uchun modul ichidagi side-effect (DB ulanish) faqat bir marta sodir bo'ladi. "Har safar yangi nusxa" kutsangiz, bu xato manbai.

### ❓ module resolution qanday ishlaydi?

**✅ Javob:** Core modul bo'lsa ichkaridan; yo'l (`./`,`/`) bo'lsa shu yo'ldan (kengaytma, `index.js`, `main`); aks holda `node_modules`'dan joriy papkadan ildizgacha yuqoriga izlaydi.

### ❓ process.nextTick va Promise.then tartibi qanaqa?

**✅ Javob:** `process.nextTick` callback'lari Promise microtask'laridan ham oldin ishlaydi. Tartib: sinxron kod → `nextTick` navbati → Promise microtask'lari → keyin event loop fazalari. `nextTick`ni rekursiv ishlatish event loop'ni och qoldiradi.

### ❓ Buffer nima uchun kerak?

**✅ Javob:** JS string'lari matn (Unicode) uchun; fayl, tarmoq, kripto esa xom baytlar bilan ishlaydi. Buffer — V8 heap'idan tashqaridagi belgilangan o'lchamli binar xotira. `Buffer.from`, `toString('hex'/'base64')` bilan kodlash o'zgartiriladi.

### ❓ readFile va readFileSync orasidagi tanlov qachon?

**✅ Javob:** Server so'rovini ishlovchi kodda har doim async/promise (`readFile`/`fs.promises`) — sync versiya event loop'ni bloklab, barcha client'larni kutdiradi. Sync faqat dastur ishga tushishida (config) yoki bir martalik skriptlarda maqbul.

### ❓ worker_threads, cluster va child_process qachon ishlatiladi?

**✅ Javob:** `worker_threads` — CPU-bound hisobni event loop'dan chiqarish (bir process, thread'lar). `cluster` — web serverni multi-core'da scale qilish (server nusxalari, portni baham). `child_process` — tashqi dastur/skript ishga tushirish (`spawn`/`exec`/`fork`).

### ❓ dependencies va devDependencies farqi?

**✅ Javob:** `dependencies` — ilova ishlashi uchun kerak (express, pg), prod'da o'rnatiladi. `devDependencies` — faqat ishlab chiqish/test uchun (jest, eslint, nodemon), `--production`da o'rnatilmaydi.

### ❓ Semverда `^4.18.2` va `~4.18.2` farqi?

**✅ Javob:** `^4.18.2` MAJOR'ni qotiradi → `4.x.x` (`4.99.0` ha, `5.0.0` yo'q). `~4.18.2` faqat PATCH'ga ruxsat → `4.18.x`. Aniq `4.18.2` — faqat o'sha versiya.

### ❓ package-lock.json nima uchun kerak va Git'ga qo'yiladimi?

**✅ Javob:** Lockfile har paketning aniq versiyasi va manzilini qotiradi — har mashinada bir xil `node_modules`. Ha, u Git'ga commit qilinadi. CI/prod'da `npm ci` lockfile'ga qat'iy amal qiladi (`npm install` emas).

### ❓ Error-first callback konvensiyasi nima?

**✅ Javob:** Callback'ning birinchi argumenti xato (yo'q bo'lsa `null`), ikkinchisi natija. `if (err) return ...` bilan avval xatoni tekshirish shart. `return` unutilsa, kod xatodan keyin davom etib ikkinchi xatoga olib keladi.

### ❓ util.promisify nima qiladi?

**✅ Javob:** Error-first callback uslubidagi funksiyani Promise qaytaradigan funksiyaga aylantiradi, async/await bilan ishlatish uchun. Lekin ko'p modullar allaqachon promise versiyasini beradi (`fs/promises`) — avval shuni tekshiring.

### ❓ Node.js qanaqa ish turlari uchun yomon tanlov?

**✅ Javob:** CPU-bound (og'ir hisob, video kodlash) — ular bitta JS thread'ni bloklaydi va barcha so'rovlarni kutdiradi. Node.js I/O-bound (API, real-time, chat) uchun a'lo. CPU ish kerak bo'lsa — `worker_threads` yoki boshqa til.

---

## Masalalar

> Yechimlar: [solutions/backend/02-nodejs-fundamentals.md](../solutions/backend/02-nodejs-fundamentals.md)

1. **Tartibni bashorat qiling.** Quyidagi kod nima tartibda chop etadi? Sababini `nextTick`, microtask va sinxron kod orqali tushuntiring:
   ```js
   console.log('A');
   setTimeout(() => console.log('B'), 0);
   process.nextTick(() => console.log('C'));
   Promise.resolve().then(() => console.log('D'));
   console.log('E');
   ```

2. **CommonJS modul.** `logger.js` moduli yarating: u `info(msg)` va `error(msg)` funksiyalarini eksport qilsin (vaqt bilan birga chop etsin). Uni boshqa fayldan `require` qilib ishlating.

3. **require cache namoyishi.** Ichida `let count = 0` va `inc()`/`get()` bo'lgan modul yozing. Uni ikki xil fayldan `require` qiling va `inc` chaqirib, ikkala faylda `get()` bir xil qiymat berishini tushuntiring. Nega 0 ga qaytmaydi?

4. **Sync vs async.** Bir xil katta faylni `fs.readFileSync` va `fs/promises` bilan o'qiydigan ikki versiya yozing. Server kontekstida qaysi biri xavfli va nima uchun? Misol bilan ko'rsating.

5. **CPU-bound bloklash.** `fib(42)` ni HTTP handler ichida hisoblaydigan server yozing, so'ng uni `worker_threads`'ga ko'chiring. Ikki holatда bir vaqtda kelgan ikkinchi so'rov javobiga nima bo'ladi?

6. **Buffer kodlash.** `'Salom, dunyo!'` matnini Buffer'ga aylantiring va uni `hex` hamda `base64` ko'rinishida chop eting. Keyin `base64` natijani qayta matnga qaytaring. UTF-8 belgi soni va bayt soni nega farq qilishi mumkinligini tushuntiring.

7. **promisify.** Error-first callback uslubidagi o'zingizning `readConfig(path, cb)` funksiyangizni yozing (1 soniya kechikib ma'lumot qaytaradigan, ba'zan xato beradigan). Uni `util.promisify` bilan o'rab, `async/await` orqali `try/catch` bilan ishlating.

8. **package.json tahlili.** Berilgan `package.json`da `"express": "^4.18.2"`, `"jest"` esa `devDependencies`da. (a) `^` qaysi versiyalarga ruxsat beradi? (b) Prod build'da jest o'rnatiladimi? (c) `npm install` va `npm ci` farqini va lockfile rolini tushuntiring.

9. **cluster bilan scale.** 4 yadroli serverда `cluster` orqali 4 ta worker ishga tushiradigan kod yozing. Bu nimani tezlashtiradi va nimani tezlashtirmaydi? `worker_threads`'dan farqi nima?

10. **Module aralashmasi.** `"type": "module"` qo'yilgan loyihada `const x = require('./a')` yozsangiz qanday xato chiqadi va nima uchun? Uni qanday to'g'rilaysiz (ikki yo'l bilan)?

← [Backend bo'limiga qaytish](./README.md)
