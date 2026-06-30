# Async Patternlar

Node.js threadsiz konkurrentlikni qanday amalga oshiradi — callback'lardan promise va async/await'gacha, cancellation, timeout, retry va concurrency control. Bu mavzu mid darajadan boshlab so'raladi, lekin sequential vs parallel await, AbortController va backoff kabi nuqtalar senior suhbatda ham farqlovchi bo'ladi. Bu hujjat nazariya + amaliy patternlar + implementatsiya masalalarini qamrab oladi.

## Mundarija

- [Sync vs Async, Blocking vs Non-blocking](#sync-vs-async-blocking-vs-non-blocking)
- [Callback va error-first konvensiya](#callback-va-error-first-konvensiya)
- [Callback hell](#callback-hell)
- [Promises (state, chaining, error propagation)](#promises-state-chaining-error-propagation)
- [Promise kombinatorlari (all/allSettled/race/any)](#promise-kombinatorlari-allallsettledraceany)
- [async / await](#async--await)
- [Sequential vs Parallel await (perf xato)](#sequential-vs-parallel-await-perf-xato)
- [Top-level await](#top-level-await)
- [Callback'ni promisega aylantirish (promisify)](#callbackni-promisega-aylantirish-promisify)
- [Concurrency control (parallelizmni cheklash)](#concurrency-control-parallelizmni-cheklash)
- [AbortController va cancellation](#abortcontroller-va-cancellation)
- [Timeout](#timeout)
- [Retry va exponential backoff](#retry-va-exponential-backoff)
- [unhandledRejection](#unhandledrejection)
- [Intervyu Q&A](#intervyu-qa)
- [Masalalar](#masalalar)

---

## Sync vs Async, Blocking vs Non-blocking

**💡 Tushuncha:** Bu ikki o'q (axis) bir-biriga bog'liq, lekin bir xil emas:

- **Sync vs Async** — natija **qachon** qaytariladi: shu yerda darhol (sync) yoki keyinroq callback/promise/event orqali (async).
- **Blocking vs Non-blocking** — kutish paytida **thread bo'sh qoladimi**: bloklasa, thread boshqa ish qila olmaydi; non-blocking bo'lsa, qila oladi.

```js
const fs = require('node:fs');

// Sync + blocking: butun event loop muzlaydi
const buf = fs.readFileSync('big.log');     // hammasi to'xtaydi

// Async + non-blocking: loop boshqa so'rovlarga xizmat qiladi
fs.readFile('big.log', (err, buf) => { /* keyinroq */ });
```

Node'da bitta JS thread bor — uni **hech qachon bloklamaslik** kerak. Sync FS o'qish, og'ir CPU loop yoki 50 MB string'ni `JSON.parse` qilish **barcha** ulanishlarni to'xtatadi. Async I/O esa libuv thread pool (fayl, DNS) yoki kernel (tarmoq) ga yuklanadi va tugagach event sifatida qaytadi.

**⚠️ Ehtiyot bo'l:** Async **parallel degani emas**. Bitta `await` ham kooperativ — kod yo'l beradi, lekin baribir bir vaqtda faqat bitta JS callback ishlaydi. CPU ishini chinakam parallel qilish uchun `worker_threads` yoki alohida jarayon kerak.

---

## Callback va error-first konvensiya

**💡 Tushuncha:** Callback — async API tugaganda chaqiriladigan funksiya. Node'ning yadro konvensiyasi **error-first**: birinchi argument xato (yoki `null`), qolganlari natija.

```js
fs.readFile('config.json', 'utf8', (err, data) => {
  if (err) return handle(err);     // har doim avval err ni tekshir VA return qil
  const config = JSON.parse(data);
});
```

**⚠️ Ehtiyot bo'l:** Xatoni qayta ishlagandan keyin `return` qilishni unutsangiz, success yo'li `undefined` data bilan ishlaydi. Yana: async callback ichida sync `throw` qilish **caller'ga tarqalmaydi** — uni ushlovchi call stack yo'q, u `uncaughtException` ga aylanadi.

---

## Callback hell

**💡 Tushuncha:** Bir-biriga bog'liq async chaqiruvlarni ichma-ich joylashtirish chuqur tirnoqli "halokat piramidasi" (pyramid of doom) kod hosil qiladi — xato boshqaruvi takrorlanadi, oqimni kuzatish qiyinlashadi.

```js
getUser(id, (err, user) => {
  if (err) return done(err);
  getOrders(user.id, (err, orders) => {
    if (err) return done(err);
    getShipping(orders[0].id, (err, ship) => {
      if (err) return done(err);
      done(null, ship);
    });
  });
});
```

Promise'lar buni **zanjir**ga, async/await esa **chiziqli** kodga aylantiradi va xato boshqaruvini markazlashtiradi.

---

## Promises (state, chaining, error propagation)

**💡 Tushuncha:** `Promise` — kelajakdagi qiymatni ifodalovchi obyekt. U uch **holat**da bo'ladi: `pending`, `fulfilled` (qiymat bilan hal qilingan) yoki `rejected` (sabab bilan). Bir marta **settle** bo'lgach (fulfilled yoki rejected) — o'zgarmas; boshqa holatga o'tmaydi.

```ts
const p = new Promise<number>((resolve, reject) => {
  setTimeout(() => resolve(42), 100);   // bir marta, keyinroq settle bo'ladi
});
```

**Chaining va error propagation.** `.then` **yangi** promise qaytaradi, shuning uchun zanjirlar tuziladi. Rejection esa `.catch` (yoki `.then` ning ikkinchi argumenti) topilguncha keyingi `.then`'larni o'tkazib yuboradi.

```ts
fetchUser(id)
  .then(user => fetchOrders(user.id))   // promise qaytarish uni "yassilaydi"
  .then(orders => render(orders))
  .catch(err => log(err))               // YUQORIDAGI har qanday bosqich xatosini ushlaydi
  .finally(() => spinner.hide());        // ham success, ham failure'da ishlaydi
```

**Keng uchraydigan xatolar:**

```ts
// 1. return qilmaslik → keyingi .then kutmaydi, zanjir buziladi
.then(user => { fetchOrders(user.id); })   // XATO: undefined qaytaradi

// 2. chaining o'rniga ichma-ich promise (callback hell qaytadi)
.then(user => { fetchOrders(user.id).then(o => ...); })

// 3. catch'siz xatoni yutib yuborish → unhandled rejection

// 4. promise'ni yana new Promise ichiga o'rash (explicit construction antipattern)
function bad() {
  return new Promise((res, rej) => {
    fetch(url).then(res, rej);  // keraksiz o'ram, rejection'ni yo'qotish oson
  });
}
```

**⚠️ Ehtiyot bo'l:** Promise executor **sinxron** ishlaydi (qurilish paytida). `new Promise(() => { throw e })` promise'ni reject qiladi; lekin executor ichidagi `setTimeout` callback'da `throw` qilish — bu rejection emas, `uncaughtException` bo'ladi.

---

## Promise kombinatorlari (all/allSettled/race/any)

| Kombinator | Qachon resolve | Qachon reject | Natija |
|---|---|---|---|
| `Promise.all` | **hammasi** fulfill | **biror** reject (fail-fast) | qiymatlar massivi |
| `Promise.allSettled` | **hammasi** settle | hech qachon | `{status, value/reason}` massivi |
| `Promise.race` | **birinchi** settle | birinchi rejection bilan settle bo'lsa | birinchi settle qiymati/sababi |
| `Promise.any` | **birinchi** fulfill | **hammasi** reject (`AggregateError`) | birinchi fulfilled qiymat |

```ts
// Fan-out, fail-fast: bitta xato butun natijani bekor qiladi
const [a, b] = await Promise.all([fetchA(), fetchB()]);

// Qisman muvaffaqiyatga toqat: har bir natijani tekshiramiz
const results = await Promise.allSettled([fetchA(), fetchB()]);
const ok = results.filter(r => r.status === 'fulfilled').map(r => r.value);

// Birinchi javob bergani yutadi (eng tez mirror)
const fastest = await Promise.any([mirror1(), mirror2(), mirror3()]);
```

**⚠️ Ehtiyot bo'l:** `Promise.all` bitta reject bo'lganda boshqalarini **bekor qilmaydi** — ular ishlashda davom etadi, natijalari tashlanadi, rejection'lari esa **unhandled** bo'lib qolishi mumkin. Yon ta'sir muhim bo'lsa, har biriga `.catch` qo'shing yoki `allSettled` ishlating. Yana: barcha kombinatorlar promise'larni darhol boshlaydi; ishni qachon boshlashni boshqarmoqchi bo'lsangiz, **funksiya** uzating va kerakli vaqtda chaqiring.

---

## async / await

**💡 Tushuncha:** `async`/`await` — promise'lar ustidagi sintaktik shakar. `async` funksiya doim promise qaytaradi; `await` funksiyani awaited promise settle bo'lguncha to'xtatadi, keyin qiymati bilan davom etadi (yoki rejection'ni `throw` qiladi).

```ts
async function loadDashboard(id: string) {
  try {
    const user = await fetchUser(id);
    const orders = await fetchOrders(user.id);
    return { user, orders };
  } catch (err) {
    // YUQORIDAGI istalgan await rejection'ini ushlaydi — sync try/catch kabi
    logger.error({ err }, 'dashboard yuklash xatosi');
    throw err;   // qayta throw yoki fallback qaytarish
  }
}
```

**⚠️ Ehtiyot bo'l:** `await` faqat siz **aslida await qilgan** rejection'ni ushlaydi. `const p = mayReject();` deb keyin boshqa kod, so'ng keyinroq `await p` qilsangiz — yaratish va await orasidagi rejection unhandled bo'lib otilishi mumkin. Va `forEach(async ...)` **await qilmaydi** — loop barcha callback'larni otib darhol qaytadi; `for...of` + `await` yoki `Promise.all(map(...))` ishlating.

---

## Sequential vs Parallel await (perf xato)

**💡 Tushuncha:** Mustaqil operatsiyalarni ketma-ket await qilish ularni keraksiz **serializatsiya** qiladi. Bu real loyihalardagi eng keng tarqalgan performance xatolaridan biri.

```ts
// ❌ Ketma-ket: umumiy vaqt ≈ A + B + C
const a = await fetchA();
const b = await fetchB();
const c = await fetchC();

// ✅ Parallel: umumiy vaqt ≈ max(A, B, C)
const [a, b, c] = await Promise.all([fetchA(), fetchB(), fetchC()]);
```

Faqat haqiqiy **bog'liqlik** bo'lsa serializatsiya qiling (B uchun A natijasi kerak). Mustaqil bo'lsa — hammasini boshlang, keyin await qiling.

**⚠️ Ehtiyot bo'l:** Nozik variant loop ichida yashiringan:

```ts
// ❌ N marta serializatsiya qilingan round-trip
for (const id of ids) results.push(await fetchOne(id));

// ✅ konkurrent (lekin cheksiz fan-out'ga ehtiyot bo'l — concurrency control)
const results = await Promise.all(ids.map(fetchOne));
```

---

## Top-level await

**💡 Tushuncha:** ES modullarda (`.mjs` yoki `"type": "module"`) `await` ni modul eng yuqori darajasida ishlatish mumkin. Modul evaluatsiyasi u settle bo'lguncha to'xtaydi, va uni import qilgan modullar ham kutadi.

```ts
// db.ts (ES module)
export const db = await connect(process.env.DATABASE_URL);
```

**⚠️ Ehtiyot bo'l:** Top-level await sikllik bog'liqlikda deadlock yoki sekin startup keltirishi mumkin, va CommonJS'da mavjud emas. Uni bir martalik async initsializatsiya uchun (config, DB ulanish) ishlating, umumiy oqim boshqaruvi vositasi sifatida emas.

---

## Callback'ni promisega aylantirish (promisify)

**💡 Tushuncha:** Error-first callback API'sini promise qaytaradigan qilib o'rash. Node'da `util.promisify` bor; ko'p kutubxonalar promise variantini beradi (`fs/promises`).

```ts
import { promisify } from 'node:util';
import fs from 'node:fs';
const readFile = promisify(fs.readFile);
const data = await readFile('x.txt', 'utf8');
```

Qo'lda yozilgan (suhbatda so'raladigan implementatsiya):

```ts
function promisify<T>(
  fn: (...args: [...unknown[], (err: unknown, res: T) => void]) => void
) {
  return (...args: unknown[]) =>
    new Promise<T>((resolve, reject) => {
      fn(...args, (err: unknown, res: T) =>
        err ? reject(err) : resolve(res)
      );
    });
}
```

**⚠️ Ehtiyot bo'l:** `promisify` **oxirgi** argument error-first callback deb hisoblaydi. Ko'p success qiymati beradigan (`(err, a, b)`) yoki callback'i birinchi bo'lgan API'lar toza promisify bo'lmaydi — `util.promisify.custom` kerak bo'lishi mumkin.

---

## Concurrency control (parallelizmni cheklash)

**💡 Tushuncha:** 10 000 elementga `Promise.all(map(...))` qilish bir vaqtda 10 000 ulanish ochadi — socket, fayl deskriptor, DB pool tugaydi yoki rate limit'ga uriladi. **Parallelizmni** belgilangan `N` ga **cheklang**. `p-limit`/`p-map`'ning `concurrency` opsiyasi shuni qiladi.

```ts
// Concurrency limiter (suhbat implementatsiyasi)
function pLimit(concurrency: number) {
  let active = 0;
  const queue: (() => void)[] = [];
  const next = () => {
    active--;
    if (queue.length) queue.shift()!();
  };
  return <T>(fn: () => Promise<T>): Promise<T> =>
    new Promise<T>((resolve, reject) => {
      const run = () => {
        active++;
        fn().then(resolve, reject).finally(next);
      };
      if (active < concurrency) run();
      else queue.push(run);
    });
}

// Ishlatish: istalgan paytda ko'pi bilan 5 ta jarayon
const limit = pLimit(5);
const results = await Promise.all(ids.map(id => limit(() => fetchOne(id))));
```

**Batching** — soddaroq qarindoshi: ro'yxatni belgilangan o'lchamdagi bo'laklarga bo'lib, har bo'lakni keyingisidan oldin await qilish. Osonroq, lekin sekinroq, chunki bitta sekin element butun bo'lakni to'xtatadi.

```ts
async function inBatches<T, R>(items: T[], size: number, fn: (t: T) => Promise<R>) {
  const out: R[] = [];
  for (let i = 0; i < items.length; i += size) {
    const chunk = items.slice(i, i + size);
    out.push(...(await Promise.all(chunk.map(fn))));
  }
  return out;
}
```

**⚠️ Ehtiyot bo'l:** Concurrency limit **quyi oqim** (downstream) limitlarini hisobga olishi kerak — DB pool o'lchami, tashqi API rate limit, xotira (har in-flight task bufer ushlaydi). 20 lik pool'ni bosib ketadigan 1000 lik limit shunchaki navbatni siljitadi, muammoni hal qilmaydi.

---

## AbortController va cancellation

**💡 Tushuncha:** Promise'larning o'zi **bekor qilinmaydi** — boshlangach settle'gacha ishlaydi. `AbortController` standart `AbortSignal` beradi, uni bekor qilinadigan API'lar (`fetch`, `fs`, ko'p kutubxonalar) kuzatib, erta to'xtaydi.

```ts
const ac = new AbortController();
const t = setTimeout(() => ac.abort(), 5000);   // 5s dan keyin bekor qil
try {
  const res = await fetch(url, { signal: ac.signal });
  return await res.json();
} catch (e) {
  if ((e as Error).name === 'AbortError') return null; // abort'da kutilgan holat
  throw e;
} finally {
  clearTimeout(t);
}
```

Signal'larni zanjirlash (`AbortSignal.any([userCancel, timeout])`) va abort'ni `signal.addEventListener('abort', ...)` orqali kuzatish mumkin.

**⚠️ Ehtiyot bo'l:** Abort faqat operatsiya signalni **hurmat qilsa** ishlaydi. Toza CPU loop yoki signalni e'tiborsiz qoldiruvchi kutubxona davom etaveradi. Timeout taymerni `finally` da `clearTimeout` qiling — aks holda taymer leak bo'ladi va event loop'ni tirik ushlab turadi.

---

## Timeout

**💡 Tushuncha:** Operatsiyani taymerga qarshi qo'ying — osilgan bog'liqlik abadiy bloklamasligi uchun. Signal-asoslangan timeout (`AbortSignal.timeout(ms)`) afzal, chunki ish chinakam bekor qilinadi, shunchaki e'tiborsiz qoldirilmaydi.

```ts
// Afzal: fetch'ni bekor qiladi
await fetch(url, { signal: AbortSignal.timeout(3000) });

// Umumiy race o'rami (DIQQAT: ostidagi ishni bekor QILMAYDI)
function withTimeout<T>(p: Promise<T>, ms: number): Promise<T> {
  return Promise.race([
    p,
    new Promise<T>((_, rej) =>
      setTimeout(() => rej(new Error('timeout')), ms)
    ),
  ]);
}
```

**⚠️ Ehtiyot bo'l:** `Promise.race` timeout caller'ni davom ettiradi, lekin sekin operatsiya fonda ishlashda davom etadi — resurslarni ushlab turadi va keyinroq bo'shliqqa resolve bo'lishi mumkin. Chinakam cancellation uchun `AbortController` ishlating.

---

## Retry va exponential backoff

**💡 Tushuncha:** Vaqtinchalik xatolar (tarmoq uzilishi, 503, deadlock) ko'pincha qayta urinishda muvaffaqiyatli bo'ladi. **Exponential backoff** urinishlar orasidagi kutishni oshiradi (`base * 2^attempt`); **jitter** uni tasodifiylashtirib, sinxron retry bo'roni (thundering herd)ning oldini oladi.

```ts
async function retry<T>(
  fn: () => Promise<T>,
  { retries = 5, base = 100, factor = 2, maxDelay = 5000 } = {}
): Promise<T> {
  let attempt = 0;
  for (;;) {
    try {
      return await fn();
    } catch (err) {
      if (attempt >= retries || !isRetryable(err)) throw err;
      const backoff = Math.min(base * factor ** attempt, maxDelay);
      const jitter = Math.random() * backoff;   // "full jitter"
      await new Promise(r => setTimeout(r, jitter));
      attempt++;
    }
  }
}
```

**⚠️ Ehtiyot bo'l:** Faqat **idempotent** yoki retry-xavfsiz operatsiyalarni qayta urinib ko'ring. `POST /charge` ni ko'r-ko'rona retry qilish mijozdan ikki marta pul yechishi mumkin — retry'ni **idempotency key** bilan birga ishlating. `Retry-After` header'ni hurmat qiling va umumiy urinishlarni cheklang; cheksiz retry qisqa uzilishni o'z-o'zini DDoS'ga aylantiradi. 4xx mijoz xatolarini retry qilmang (429 dan tashqari).

---

## unhandledRejection

**💡 Tushuncha:** Rejected promise garbage-collect bo'lguncha `.catch` (yoki muvaffaqiyatsiz `await`) bo'lmasa, Node `process.on('unhandledRejection')` event chiqaradi. Zamonaviy Node'da default — jarayonni **crash** qilish; bularni shovqin emas, **bug** deb hisoblang.

```ts
process.on('unhandledRejection', (reason, promise) => {
  logger.fatal({ reason }, 'unhandled rejection');
  // log, flush, keyin exit — supervisor toza jarayon qayta ishga tushiradi
  process.exit(1);
});
```

**⚠️ Ehtiyot bo'l:** Bu handler'ni rejection'larni **yutib** ishlashda davom etish uchun ishlatmang — jarayon buzilgan holatda bo'lishi mumkin. `uncaughtException` ham shunday. Eng yaxshi amaliyot: log qil, graceful shutdown'ga urin, exit qil va orkestrator (PM2, systemd, Kubernetes) qayta ishga tushirsin.

---

## Intervyu Q&A

### ❓ Async va non-blocking farqi nima?

**✅ Javob:** Async — natija **qachon** keladi (keyinroq, callback/promise/event orqali). Non-blocking — kutish paytida **thread bo'shmi**. Node I/O'ning aksariyati non-blocking va async, lekin nazariy jihatdan ular alohida o'qlar. Node kodida amalda: async API'lar non-blocking va bitta event-loop thread'ni boshqa ishlarga xizmat qilishga qo'yib yuboradi.

### ❓ Promise'ning uch holatini tushuntiring.

**✅ Javob:** `pending` (settle bo'lmagan), `fulfilled` (qiymat bilan settle), `rejected` (sabab bilan settle). Settle bir tomonlama va abadiy. `.then` fulfillment'ni, `.catch` rejection'ni, `.finally` har ikkalasini boshqaradi. "Thenable" — `.then` metodli har qanday obyekt, uni `await` va `Promise.resolve` qabul qiladi.

### ❓ `Promise.all` va `Promise.allSettled` — qachon qaysi?

**✅ Javob:** `all` — har bir natija kerak bo'lib, biror xato butunni bekor qilishi kerak bo'lganda (fail-fast, masalan sahifaning barcha qismlarini yuklash). `allSettled` — qisman muvaffaqiyat kerak va har natijani tekshirish kerak bo'lganda (masalan 100 ta bildirishnoma — bittasining xatosi qolgan 99 tasini to'xtatmasligi kerak). `all` birinchi xatoda reject bo'ladi; `allSettled` hech qachon reject bo'lmaydi.

### ❓ Uchta mustaqil API chaqiruvim bor va endpoint sekin. Nima xato?

**✅ Javob:** Ular ehtimol ketma-ket await qilingan (`await a; await b; await c`), latency'ni **yig'indi** qilgan. Mustaqil bo'lgani uchun ularni `Promise.all([a(), b(), c()])` bilan birga otib, latency'ni **max** ga aylantirish kerak. Bu code review'dagi eng keng tarqalgan async perf xato.

### ❓ Nega `array.forEach(async fn)` await qilmaydi?

**✅ Javob:** `forEach` callback'ining qaytargan qiymatini e'tiborsiz qoldiradi, shuning uchun qaytgan promise'lar yo'qoladi. Loop sinxron tugaydi, async ish esa await'siz ishlaydi. Ketma-ket uchun `for...of` + `await`, parallel uchun `await Promise.all(array.map(asyncFn))` ishlating.

### ❓ Ishlayotgan async operatsiyani qanday bekor qilasiz?

**✅ Javob:** Promise'lar bekor qilinmaydi, shuning uchun `AbortController` ishlating. Controller yarating, `controller.signal` ni signal-aware API'ga (`fetch`, `fs`) bering, `controller.abort()` chaqiring. Operatsiya `AbortError` bilan reject bo'ladi. Timeout uchun `AbortSignal.timeout(ms)`. Signal'larni `AbortSignal.any([...])` bilan birlashtiring.

### ❓ Exponential backoff bilan retry qiluvchi funksiya yozing.

**✅ Javob:** Yuqoridagi `retry` implementatsiyasiga qarang. Aytib o'tish kerak bo'lgan nuqtalar: urinishlar sonini va max delay'ni cheklash, sinxron retry'ni oldini olish uchun jitter qo'shish, faqat retry-xavfsiz/idempotent xatolarni retry qilish, `Retry-After` ni hurmat qilish. Non-idempotent operatsiyalar uchun idempotency key bilan birga ishlatish.

### ❓ `promisify` ni implement qiling.

**✅ Javob:** Original funksiyani oxiriga error-first callback qo'shib chaqiruvchi, `err -> reject`, `result -> resolve` qiladigan `new Promise` qaytaruvchi funksiya. Yuqoridagi implementatsiyaga qarang. Cheklov: oxirgi argument error-first callback va bitta success qiymati deb hisoblaydi.

### ❓ 10 ming elementni qayta ishlashda concurrency'ni N ga qanday cheklaysiz?

**✅ Javob:** Semaphore/limiter (`p-limit` kabi): aktiv tasklar sonini hisoblang; `active < N` bo'lsa darhol boshlang, aks holda navbatga qo'ying; task tugaganda keyingisini navbatdan oling. Bu aniq N tani in-flight ushlaydigan siljiydigan oyna beradi — bu belgilangan o'lchamli batching'dan samaraliroq (unda bitta sekin element butun batch'ni to'xtatadi). N ni eng kichik downstream limitga (DB pool, rate limit, xotira) sozlang.

### ❓ Explicit promise construction antipattern nima?

**✅ Javob:** Allaqachon promise qaytaradigan narsani `new Promise((resolve, reject) => {...})` ichiga o'rash. Bu hech narsa qo'shmaydi, xato qilish oson (biror yo'lda reject'ni unutish), xatolarni yashiradi. Mavjud promise'ni shunchaki return/await qiling. Constructor'ni faqat haqiqatan callback- yoki event-asoslangan API'ni ko'priklash uchun ishlating.

### ❓ `Promise.race` va `Promise.any` farqi?

**✅ Javob:** `race` birinchi promise settle bo'lishi bilan settle bo'ladi — fulfilled **yoki** rejected. `any` esa birinchi **fulfilled**'da settle bo'ladi va faqat **hammasi** reject bo'lsa reject qiladi (`AggregateError` bilan). `race` ni timeout uchun, `any` ni "birinchi muvaffaqiyatli mirror" uchun ishlating.

### ❓ Zamonaviy Node'da unhandled promise rejection'da nima bo'ladi?

**✅ Javob:** Node `unhandledRejection` event chiqaradi; default (v15+) jarayon crash bo'ladi. To'g'ri pattern — log qilish, flush va exit, supervisor toza jarayon qayta ishga tushiradi — yutib yuborish emas. Bu odatda biror joyda yo'qolgan `await`/`.catch` ni bildiradi.

### ❓ Nega `Promise.all` parallelizm bilan bir xil emas?

**✅ Javob:** `Promise.all` allaqachon boshlangan promise'lar settle bo'lishini kutadi; o'zi parallelizm **yaratmaydi**. Parallelizm operatsiyalarni (I/O ni) await'dan oldin **boshlash**dan keladi. Agar operatsiyalar CPU-bound JS bo'lsa, `Promise.all` ularni tezlashtirmaydi — ular baribir bitta thread'da navbatma-navbat ishlaydi. CPU parallelizmi uchun worker threads.

### ❓ Qachon ataylab ketma-ket await tanlaysiz?

**✅ Javob:** Haqiqiy data bog'liqlik bo'lganda (2-qadam 1-qadam natijasini talab qilsa), tartib/rate limit'ni majburlash kerak bo'lganda yoki hammasini birdan ishga tushirish resursni bosib ketadigan bo'lganda. Aks holda parallel afzal. Mahorat — bog'liqlikni tasodifiy serializatsiyadan ajrata bilish.

### ❓ Error-first callback'da eng keng xato nima?

**✅ Javob:** Xatoni tekshirgandan keyin `return` qilmaslik — success yo'li `undefined` data bilan davom etadi. Yana: async callback ichida sync `throw` qilish caller'ga tarqalmaydi, `uncaughtException` ga aylanadi. Doim avval `if (err) return ...` qiling.

---

## Masalalar

> Yechimlar: [solutions/backend/06-async-patterns.md](../solutions/backend/06-async-patterns.md)

1. **`promisify` yozing.** Error-first callback API'sini promise qaytaradigan qilib o'raydigan generic `promisify(fn)` funksiyasini yozing. Uni `fs.readFile` bilan sinab ko'ring.

2. **`retry` + exponential backoff yozing.** `retry(fn, { retries, base, factor, maxDelay })` funksiyasini yozing: xatoda exponential backoff + jitter bilan qayta uradi, urinishlar tugaganda yoki retry-xavfsiz bo'lmagan xatoda tashlaydi.

3. **Concurrency limiter yozing.** `pLimit(n)` funksiyasini yozing: u funksiya qaytaradi, va bir vaqtda ko'pi bilan `n` ta promise ishlashini ta'minlaydi (siljiydigan oyna). 100 ta soxta task bilan tekshiring.

4. **Sequential'ni parallel'ga aylantiring.** Berilgan `id`'lar massivi uchun har bir `fetchOne(id)` ni ketma-ket await qilgan kodni mustaqillik bo'lsa parallel'ga aylantiring, lekin concurrency'ni 5 ga cheklang (2 va 3-masalalarni birlashtiring).

5. **`withTimeout` yozing.** Promise'ni `ms` millisekundga cheklaydigan `withTimeout(promise, ms)` yozing: vaqt o'tsa `timeout` xatosi bilan reject bo'ladi. Keyin uni `AbortController` versiyasiga (chinakam cancellation) takomillashtiring.

6. **`mapWithConcurrency` yozing.** `mapWithConcurrency(items, n, fn)` yozing: `items` ustidan `fn` ni qo'llaydi, lekin ko'pi bilan `n` ta parallel ishlaydi va natijalarni **kirish tartibida** qaytaradi.

7. **Callback hell'ni async/await'ga aylantiring.** Uch bosqichli ichma-ich callback kodini (`getUser → getOrders → getShipping`) async/await va markazlashgan `try/catch` bilan qayta yozing.

8. **`allSettled` ni qo'lda implement qiling.** `Promise.allSettled` ni o'zingiz yozing: promise'lar massivini olib, hech qachon reject bo'lmasdan, har biri uchun `{ status, value }` yoki `{ status, reason }` massivini qaytaring.

← [Backend bo'limiga qaytish](./README.md)
