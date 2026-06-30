# Async Patternlar — Masalalar Yechimlari

Bu fayl [`backend/06-async-patterns.md`](../../backend/06-async-patterns.md) dagi "Masalalar" bo'limining yechimlarini o'z ichiga oladi. Har bir yechim izohlangan.

## Mundarija

- [1. `promisify` yozing](#1-promisify-yozing)
- [2. `retry` + exponential backoff](#2-retry--exponential-backoff)
- [3. Concurrency limiter (`pLimit`)](#3-concurrency-limiter-plimit)
- [4. Sequential'ni parallel'ga aylantirish](#4-sequentialni-parallelga-aylantirish)
- [5. `withTimeout`](#5-withtimeout)
- [6. `mapWithConcurrency`](#6-mapwithconcurrency)
- [7. Callback hell → async/await](#7-callback-hell--asyncawait)
- [8. `allSettled` ni qo'lda implement qilish](#8-allsettled-ni-qolda-implement-qilish)

---

## 1. `promisify` yozing

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

// Tekshirish
import fs from 'node:fs';
const readFile = promisify(fs.readFile);
const data = await readFile('package.json', 'utf8');
```

**Asosiy nuqtalar:** Original funksiyaning **oxiriga** error-first callback qo'shamiz: `err` bo'lsa `reject`, aks holda `resolve`. `this` muhim bo'lgan metodlar uchun `fn.apply(thisArg, ...)` kerak bo'lishi mumkin. Bu versiya bitta success qiymatini qabul qiladi — `(err, a, b)` kabi API'lar uchun massiv qaytarishga moslash kerak.

---

## 2. `retry` + exponential backoff

```ts
function isRetryable(err: any): boolean {
  // misol: tarmoq xatolari va 5xx/429 ni retry qilamiz
  const code = err?.code;
  const status = err?.status ?? err?.response?.status;
  if (['ECONNRESET', 'ETIMEDOUT', 'ECONNREFUSED'].includes(code)) return true;
  if (status === 429 || (status >= 500 && status < 600)) return true;
  return false;
}

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
      const jitter = Math.random() * backoff;        // full jitter
      await new Promise((r) => setTimeout(r, jitter));
      attempt++;
    }
  }
}

// Tekshirish: 3-urinishda muvaffaqiyat
let n = 0;
await retry(async () => {
  if (++n < 3) throw Object.assign(new Error('flaky'), { code: 'ETIMEDOUT' });
  return 'ok';
});
```

**Asosiy nuqtalar:** Backoff `base * factor^attempt`, `maxDelay` bilan cheklangan. Jitter sinxron "thundering herd" retry'ning oldini oladi. Faqat `isRetryable` xatolarni retry qilamiz — 4xx (429 dan tashqari) qaytarilmaydi. Non-idempotent operatsiyalarni retry qilishdan oldin idempotency key ishlatish kerak.

---

## 3. Concurrency limiter (`pLimit`)

```ts
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

// Tekshirish: bir vaqtda ko'pi bilan 3 ta
const limit = pLimit(3);
let running = 0, maxRunning = 0;
const tasks = Array.from({ length: 100 }, (_, i) =>
  limit(async () => {
    running++; maxRunning = Math.max(maxRunning, running);
    await new Promise((r) => setTimeout(r, 10));
    running--;
    return i;
  })
);
await Promise.all(tasks);
console.log('maxRunning:', maxRunning); // 3
```

**Asosiy nuqtalar:** `active` aktiv tasklar sonini hisoblaydi. `active < concurrency` bo'lsa darhol `run()`, aks holda `queue` ga. Task tugaganda (`finally(next)`) navbatdan keyingisini olamiz. Bu **siljiydigan oyna** — har doim aniq `concurrency` ta in-flight, batching'dan samaraliroq.

---

## 4. Sequential'ni parallel'ga aylantirish

```ts
// ❌ Oldingi kod: ketma-ket, sekin
async function before(ids: string[]) {
  const results = [];
  for (const id of ids) results.push(await fetchOne(id));
  return results;
}

// ✅ Parallel, lekin concurrency 5 ga cheklangan
async function after(ids: string[]) {
  const limit = pLimit(5);                       // 3-masaladagi pLimit
  return Promise.all(ids.map((id) => limit(() => fetchOne(id))));
}
```

**Asosiy nuqtalar:** `for...of` + `await` har bir chaqiruvni serializatsiya qiladi (vaqt = yig'indi). `map` + `Promise.all` ularni parallel qiladi (vaqt = max), lekin **cheksiz fan-out** xavfli — shuning uchun `pLimit(5)` bilan bir vaqtda ko'pi bilan 5 tani ushlaymiz. `Promise.all` natijalar tartibini saqlaydi.

---

## 5. `withTimeout`

```ts
// Versiya 1: race (ostidagi ishni bekor QILMAYDI)
function withTimeout<T>(p: Promise<T>, ms: number): Promise<T> {
  let timer: NodeJS.Timeout;
  const timeout = new Promise<never>((_, reject) => {
    timer = setTimeout(() => reject(new Error(`timeout ${ms}ms`)), ms);
  });
  return Promise.race([p, timeout]).finally(() => clearTimeout(timer));
}

// Versiya 2: AbortController (chinakam cancellation)
async function fetchWithTimeout(url: string, ms: number) {
  const ac = new AbortController();
  const timer = setTimeout(() => ac.abort(), ms);
  try {
    return await fetch(url, { signal: ac.signal });
  } finally {
    clearTimeout(timer);
  }
  // yoki shunchaki: fetch(url, { signal: AbortSignal.timeout(ms) })
}
```

**Asosiy nuqtalar:** Versiya 1 caller'ni davom ettiradi, lekin sekin promise fonda ishlashda davom etadi (resurs ushlaydi). `clearTimeout` ni `finally` da chaqirish taymer leak'ining oldini oladi. Versiya 2 esa `AbortController` bilan ishni **chinakam bekor** qiladi — afzal usul.

---

## 6. `mapWithConcurrency`

```ts
async function mapWithConcurrency<T, R>(
  items: T[],
  concurrency: number,
  fn: (item: T, index: number) => Promise<R>
): Promise<R[]> {
  const results: R[] = new Array(items.length);
  let nextIndex = 0;

  async function worker() {
    while (nextIndex < items.length) {
      const i = nextIndex++;              // o'z indeksini "egallaydi"
      results[i] = await fn(items[i], i); // natijani to'g'ri pozitsiyaga
    }
  }

  // concurrency ta worker'ni parallel ishga tushiramiz
  const workers = Array.from(
    { length: Math.min(concurrency, items.length) },
    () => worker()
  );
  await Promise.all(workers);
  return results;
}

// Tekshirish
const out = await mapWithConcurrency([1, 2, 3, 4, 5], 2, async (x) => x * 2);
console.log(out); // [2, 4, 6, 8, 10] — kirish tartibida
```

**Asosiy nuqtalar:** `concurrency` ta "worker" yaratamiz; har biri umumiy `nextIndex` dan keyingi elementni egallab ishlaydi. Natija `results[i]` ga **kirish indeksi** bo'yicha yoziladi, shuning uchun tartib saqlanadi (qaysi worker oldin tugashidan qat'i nazar). Bu pLimit'ning muqobil, ravon implementatsiyasi.

---

## 7. Callback hell → async/await

```ts
import { promisify } from 'node:util';

// Avval callback API'larni promisify qilamiz (yoki promise versiyasidan foydalanamiz)
const pGetUser = promisify(getUser);
const pGetOrders = promisify(getOrders);
const pGetShipping = promisify(getShipping);

async function getShippingForUser(id: string) {
  try {
    const user = await pGetUser(id);
    const orders = await pGetOrders(user.id);   // user.id ga bog'liq
    const shipping = await pGetShipping(orders[0].id); // orders ga bog'liq
    return shipping;
  } catch (err) {
    // bitta markazlashgan catch — uch bosqichning istalganidagi xato
    logger.error({ err }, 'shipping olish xatosi');
    throw err;
  }
}
```

**Asosiy nuqtalar:** Ichma-ich callback'lar (pyramid of doom) chiziqli kodga aylandi. Bu yerda bosqichlar **bog'liq** (user → orders → shipping), shuning uchun ketma-ket await **to'g'ri** — mustaqil bo'lganda `Promise.all` ishlatardik. Xato boshqaruvi takrorlanmaydi, bitta `try/catch` barcha bosqichlarni qoplaydi.

---

## 8. `allSettled` ni qo'lda implement qilish

```ts
function allSettled<T>(
  promises: Promise<T>[]
): Promise<Array<
  | { status: 'fulfilled'; value: T }
  | { status: 'rejected'; reason: unknown }
>> {
  return Promise.all(
    promises.map((p) =>
      Promise.resolve(p).then(
        (value) => ({ status: 'fulfilled' as const, value }),
        (reason) => ({ status: 'rejected' as const, reason })
      )
    )
  );
}

// Tekshirish
const res = await allSettled([
  Promise.resolve(1),
  Promise.reject(new Error('xato')),
  Promise.resolve(3),
]);
console.log(res);
// [
//   { status: 'fulfilled', value: 1 },
//   { status: 'rejected', reason: Error('xato') },
//   { status: 'fulfilled', value: 3 },
// ]
```

**Asosiy nuqtalar:** Asosiy hiyla — har bir promise'ga `.then(onFulfilled, onRejected)` qo'shib, **ikkala** holatni ham fulfilled natijaga aylantirish. Shu sabab `Promise.all` hech qachon reject bo'lmaydi (chunki ichki promise'lar doim resolve bo'ladi). `Promise.resolve(p)` non-promise qiymatlarni ham qo'llab-quvvatlaydi. `as const` TypeScript'da literal tipini saqlaydi.

---

← [Backend bo'limiga qaytish](../../backend/README.md)
