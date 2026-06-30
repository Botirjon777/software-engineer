# Node.js asoslari — Yechimlar

Bu fayl [`backend/02-nodejs-fundamentals.md`](../../backend/02-nodejs-fundamentals.md) dagi "Masalalar" bo'limining yechimlarini izohlari bilan beradi. Avval mustaqil urinib ko'ring, keyin solishtiring.

---

## 1. Tartibni bashorat qiling

**Natija:** `A → E → C → D → B`

```text
A   — sinxron, darhol
E   — sinxron, darhol
C   — process.nextTick (eng yuqori ustuvorlik, microtask'dan ham oldin)
D   — Promise microtask (nextTick'dan keyin)
B   — setTimeout (macrotask, timer fazasi, eng oxirida)
```

**Izoh:** Avval butun sinxron kod ishlaydi (`A`, `E`). So'ng joriy operatsiya tugagach: `process.nextTick` navbati (`C`) Promise microtask navbati (`D`) dan oldin bo'shatiladi. `setTimeout` (`B`) — macrotask, event loop'ning keyingi aylanishida, eng oxirida. Demak `nextTick > Promise > setTimeout`.

---

## 2. CommonJS modul

```js
// logger.js
function timestamp() {
  return new Date().toISOString();
}

function info(msg) {
  console.log(`[INFO]  ${timestamp()}  ${msg}`);
}

function error(msg) {
  console.error(`[ERROR] ${timestamp()}  ${msg}`);
}

module.exports = { info, error };
```

```js
// app.js
const logger = require('./logger');

logger.info('Server ishga tushdi');
logger.error('Bazaga ulanib bo\'lmadi');
// [INFO]  2026-06-30T...Z  Server ishga tushdi
// [ERROR] 2026-06-30T...Z  Bazaga ulanib bo'lmadi
```

**Izoh:** `module.exports`ga obyekt berdik, `require` shu obyektni qaytaradi. `info`/`error`ni alohida `exports.info = ...` bilan ham eksport qilsa bo'lardi.

---

## 3. require cache namoyishi

```js
// counter.js
let count = 0;
module.exports = {
  inc: () => ++count,
  get: () => count,
};
```

```js
// a.js
const c = require('./counter');
c.inc();
c.inc();
console.log('a.js:', c.get());   // 2
require('./b');                  // b.js'ni yuklaymiz

// b.js
const c = require('./counter');  // YANGI emas — cache'dan o'sha obyekt
console.log('b.js:', c.get());   // 2 (0 emas!)
```

**Nega 0 ga qaytmaydi:** `counter.js` *birinchi* `require`da bir marta bajariladi (`let count = 0` faqat shunda ishlaydi) va natija obyekti cache'lanadi. `b.js`dagi ikkinchi `require` modulni qayta ishga tushirmaydi — u cache'dan *aynan o'sha* obyektni oladi. `a.js` `count`ni 2 ga oshirgani uchun `b.js` ham 2 ni ko'radi. Bu — modullarning singleton xatti-harakati.

**Izoh:** Agar har faylga "yangi" hisoblagich kerak bo'lsa, obyekt o'rniga *factory funksiya* eksport qiling: `module.exports = () => { let count = 0; return {...}; }` va har joyda chaqiring.

---

## 4. Sync vs async

```js
const fs = require('fs');
const fsp = require('fs/promises');

// Xavfli (server kontekstida): event loop bloklanadi
function readSync() {
  const data = fs.readFileSync('big.txt', 'utf8');  // ⚠️ butun process kutadi
  return data;
}

// Xavfsiz: bloklamaydi
async function readAsync() {
  const data = await fsp.readFile('big.txt', 'utf8');
  return data;
}
```

```js
// Namoyish — sync handler boshqa so'rovlarni qotirib qo'yadi
const http = require('http');
http.createServer((req, res) => {
  if (req.url === '/sync') {
    const data = fs.readFileSync('big.txt', 'utf8'); // ⚠️ shu vaqtda /fast ham kutadi
    res.end(data);
  } else if (req.url === '/fast') {
    res.end('tez javob');                            // sync o'qish davom etsa, bu ham bloklanadi
  }
}).listen(3000);
```

**Qaysi biri xavfli:** `readFileSync` server handler'ida — chunki u event loop'ni bloklaydi. Fayl o'qilguncha *boshqa hech bir so'rov* (hatto eng yengili ham) javob olmaydi. Bir vaqtda yuzlab client bo'lsa, hammasi navbatda qotadi.

**Izoh:** Sync versiya faqat dastur *ishga tushishida* (config faylini bir marta o'qish) yoki bir martalik CLI skriptlarda maqbul — u yerda parallel so'rovlar yo'q.

---

## 5. CPU-bound bloklash

```js
// fib.js — og'ir hisob
function fib(n) { return n < 2 ? n : fib(n - 1) + fib(n - 2); }
module.exports = { fib };
```

**Versiya A — bloklaydi:**

```js
const http = require('http');
const { fib } = require('./fib');

http.createServer((req, res) => {
  const result = fib(42);           // ⚠️ event loop muzlaydi
  res.end(String(result));
}).listen(3000);
// Ikkinchi so'rov birinchi fib(42) tugamaguncha KUTADI.
```

**Versiya B — worker_threads:**

```js
// worker.js
const { parentPort } = require('worker_threads');
function fib(n) { return n < 2 ? n : fib(n - 1) + fib(n - 2); }
parentPort.on('message', (n) => parentPort.postMessage(fib(n)));
```

```js
// server.js
const http = require('http');
const { Worker } = require('worker_threads');

http.createServer((req, res) => {
  const worker = new Worker('./worker.js');
  worker.postMessage(42);
  worker.on('message', (result) => {
    res.end(String(result));
    worker.terminate();
  });
}).listen(3000);
// Hisob alohida thread'da — event loop bo'sh, boshqa so'rovlar darrov javob oladi.
```

**Ikkinchi so'rovga nima bo'ladi:**
- **A (bloklaydi):** Ikkinchi so'rov navbatда turadi — birinchi `fib(42)` tugamaguncha javob olmaydi. Hatto `/health` kabi yengil endpoint ham kutadi.
- **B (worker):** Hisob asosiy thread'dan tashqarida bo'lgani uchun event loop bo'sh qoladi; ikkinchi so'rov darhol qabul qilinadi va o'z worker'ida hisoblanadi.

**Izoh:** Prod'da har so'rovga yangi worker yaratish qimmat — odatda worker pool (oldindan yaratilgan worker'lar to'plami) ishlatiladi (masalan `piscina` kutubxonasi).

---

## 6. Buffer kodlash

```js
const text = 'Salom, dunyo!';
const buf = Buffer.from(text, 'utf8');

console.log('hex:   ', buf.toString('hex'));
// hex:    53616c6f6d2c2064756e796f21
console.log('base64:', buf.toString('base64'));
// base64: U2Fsb20sIGR1bnlvIQ==

// base64'dan qaytarish
const back = Buffer.from('U2Fsb20sIGR1bnlvIQ==', 'base64').toString('utf8');
console.log('qaytdi:', back);    // Salom, dunyo!

console.log('belgilar:', text.length);   // 13 (string uzunligi)
console.log('baytlar: ', buf.length);    // 13 (bu yerda teng, chunki ASCII)
```

**Belgi vs bayt soni nega farq qiladi:** UTF-8'da ASCII belgilar (lotin harflari, raqamlar) 1 bayt, lekin boshqa belgilar ko'proq bayt egallaydi:

```js
const t = 'Ёжик 🦔';
console.log(t.length);                    // belgilar (kod birliklari)
console.log(Buffer.from(t).length);       // ancha ko'proq baytlar
// Kirill harflari 2 bayt, emoji 4 bayt — string.length bayt soniga teng emas.
```

**Izoh:** Shuning uchun `Content-Length` (bayt) ni hisoblashda `string.length` emas, `Buffer.byteLength(str)` ishlatiladi.

---

## 7. promisify

```js
const util = require('util');

// Error-first callback uslubidagi funksiya
function readConfig(path, cb) {
  setTimeout(() => {
    if (path === 'missing.json') {
      return cb(new Error('Fayl topilmadi: ' + path));   // xato — birinchi argument
    }
    cb(null, { db: 'postgres', port: 5432 });             // null = xato yo'q, keyin natija
  }, 1000);
}

// Promise versiyaga aylantirish
const readConfigAsync = util.promisify(readConfig);

async function main() {
  try {
    const config = await readConfigAsync('config.json');
    console.log('Config:', config);          // { db: 'postgres', port: 5432 }

    await readConfigAsync('missing.json');   // bu xato beradi
  } catch (err) {
    console.error('Xato:', err.message);     // Xato: Fayl topilmadi: missing.json
  }
}
main();
```

**Izoh:** `util.promisify` ishlashi uchun funksiya Node konvensiyasiga *qat'iy* amal qilishi shart: oxirgi argument callback, callback'ning birinchi argumenti xato. `cb(null, data)` — muvaffaqiyat (promise resolve), `cb(err)` — xato (promise reject, `catch`ga tushadi).

---

## 8. package.json tahlili

**(a) `^4.18.2` qaysi versiyalarga ruxsat beradi:** `>=4.18.2 <5.0.0` — ya'ni MAJOR (4) qotiriladi, MINOR va PATCH yangilanishi mumkin. `4.18.5`, `4.19.0` ha; `5.0.0` yo'q.

**(b) Prod build'da jest:** Yo'q. `jest` `devDependencies`da, `npm install --production` (yoki `NODE_ENV=production`) faqat `dependencies`ni o'rnatadi. Test vositalari prod image'ni shishirmaydi.

**(c) `npm install` vs `npm ci` va lockfile:**
- `npm install` — `package.json`ga qaraydi, kerak bo'lsa lockfile'ni *yangilaydi* (`^` tufayli yangi minor tortishi mumkin). Lokal ishlab chiqish uchun.
- `npm ci` — *faqat* `package-lock.json`ga qaraydi, aniq versiyalarni o'rnatadi, `node_modules`ni tozalab qayta quradi. Lockfile bilan `package.json` mos kelmasa — xato beradi. CI/prod uchun: tez, aniq, takrorlanadigan.
- **Lockfile roli** — har bir paketning (va tranzitiv bog'liqliklarning) aniq versiyasi va integriteti. U tufayli har bir mashinada aynan bir xil `node_modules` o'rnatiladi.

**Izoh:** Qoida — lockfile'ni doim commit qilish, CI'da `npm ci` ishlatish.

---

## 9. cluster bilan scale

```js
const cluster = require('cluster');
const os = require('os');
const http = require('http');

if (cluster.isPrimary) {
  const cpus = os.cpus().length;            // masalan 4
  console.log(`Primary ${process.pid} — ${cpus} worker yaratiladi`);
  for (let i = 0; i < cpus; i++) cluster.fork();

  cluster.on('exit', (worker) => {
    console.log(`Worker ${worker.process.pid} o'ldi, qayta yaratamiz`);
    cluster.fork();                         // crash bo'lsa qayta tiklash
  });
} else {
  http.createServer((req, res) => {
    res.end(`Salom worker ${process.pid}'dan`);
  }).listen(3000);                          // portni barcha worker baham ko'radi
}
```

**Nimani tezlashtiradi:** I/O-bound web serverni multi-core'da — endi 4 ta process portni baham ko'rib, kelgan ulanishlarni o'zaro taqsimlaydi. Umumiy o'tkazuvchanlik (throughput) oshadi, 1 ta yadro o'rniga 4 tasi ishlaydi.

**Nimani tezlashtirmaydi:** Bitta og'ir CPU-bound *so'rovni* — u baribir bitta worker'ning bitta thread'ida hisoblanadi va o'sha worker'ni bloklaydi. Cluster faqat so'rovlarni process'lar orasida taqsimlaydi, bitta so'rovni parallellashtirmaydi.

**`worker_threads`'dan farqi:** `cluster` — alohida xotirali *process*lar (server nusxalari), web serverni scale qilish uchun; aloqa IPC orqali. `worker_threads` — bir process ichida *thread*lar (xotira bo'lishadi), bitta og'ir CPU vazifani event loop'dan chiqarish uchun.

**Izoh:** Amaliyotda ko'pincha cluster o'rniga PM2 yoki Kubernetes (bir nechta pod) ishlatiladi — bir xil g'oya, lekin tashqi orchestrator boshqaradi.

---

## 10. Module aralashmasi

**Xato:** `"type": "module"` qo'yilgan loyihada `const x = require('./a')` yozsangiz:

```text
ReferenceError: require is not defined in ES module scope, you can use import instead
```

**Nega:** `"type": "module"` butun loyihadagi `.js` fayllarni ESM deb belgilaydi. ESM scope'ida `require`, `module.exports`, `__dirname`, `__filename` mavjud emas — bular CommonJS'ga xos. ESM faqat `import`/`export` ishlatadi.

**To'g'rilash — ikki yo'l:**

```js
// 1-yo'l: ESM sintaksisiga o'tish (loyiha ESM bo'lib qolsin)
import x from './a.js';     // ⚠️ ESM'da kengaytma (.js) majburiy
```

```js
// 2-yo'l: shu faylni CommonJS qilish — uni .cjs deb nomlash
// fayl: app.cjs
const x = require('./a.cjs');   // .cjs fayl har doim CommonJS, "type" ta'sir qilmaydi
```

**Izoh:** ESM'da `__dirname` kerak bo'lsa: `import { fileURLToPath } from 'url'; const __dirname = path.dirname(fileURLToPath(import.meta.url));`. ESM import'larida nisbiy yo'lга fayl kengaytmasini (`.js`) yozish majburiy — bu CJS'dан eng ko'p sezilarli farqlardan biri.

---

← [Backend bo'limiga qaytish](../../backend/README.md)
