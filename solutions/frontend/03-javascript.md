# JavaScript — Yechimlar

Bu fayl [`frontend/03-javascript.md`](../../frontend/03-javascript.md) dagi **Masalalar** bo'limining to'liq yechimlarini, kod va izohlar bilan, oson→qiyin tartibida saqlaydi.

---

## 1. `once`

Funksiyani ko'pi bilan bir marta chaqiradi; keyingi chaqiruvlar birinchi natijani qaytaradi. Closure orqali `called` va `result` ushlanadi.

```js
function once(fn) {
  let called = false;
  let result;
  return function (...args) {
    if (!called) {
      called = true;
      result = fn.apply(this, args); // `this` va argumentlarni uzatamiz
    }
    return result;
  };
}

// Sinov
const init = once(() => {
  console.log("faqat bir marta ishga tushadi");
  return 42;
});
init(); // => "faqat bir marta..." log, 42 qaytaradi
init(); // log YO'Q, 42 qaytaradi (keshlangan)
```

**Izoh:** `fn.apply(this, args)` muhim — `obj.method = once(fn)` holatida `this` to'g'ri uzatiladi.

---

## 2. `flatten`

Istalgan chuqurlikdagi massivni tekislaydi. `reduce` har bir elementni tekshiradi: agar massiv bo'lsa va chuqurlik qolsa — rekursiya, aks holda `concat`.

```js
function flatten(arr, depth = Infinity) {
  return arr.reduce(
    (acc, x) =>
      Array.isArray(x) && depth > 0
        ? acc.concat(flatten(x, depth - 1)) // chuqurlikni kamaytirib rekursiya
        : acc.concat(x),
    []
  );
}

flatten([1, [2, [3, [4]]]]);        // => [1, 2, 3, 4]
flatten([1, [2, [3, [4]]]], 1);     // => [1, 2, [3, [4]]] (faqat 1 daraja)
```

**Izoh:** `depth` parametri har rekursiv chaqiruvda `1` ga kamayadi, shuning uchun cheklangan chuqurlik ham ishlaydi. Native `Array.prototype.flat(depth)` aynan shunday ishlaydi.

---

## 3. `curry`

`fn.length` (kutilayotgan argumentlar soni) yetarli yig'ilgunicha argumentlarni to'playdi, keyin chaqiradi.

```js
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) {
      return fn.apply(this, args);          // yetarli argument — chaqiramiz
    }
    return (...rest) => curried.apply(this, [...args, ...rest]); // yana to'playmiz
  };
}

const add = curry((a, b, c) => a + b + c);
add(1)(2)(3);   // => 6
add(1, 2)(3);   // => 6
add(1)(2, 3);   // => 6
add(1, 2, 3);   // => 6
```

**Izoh:** `fn.length` faqat rest/default'siz oddiy parametrlarni sanaydi. `(a, b = 1) => ...` da `length` `1` bo'ladi — currying buni hisobga olmaydi.

---

## 4. `debounce` (leading + trailing)

Faollik to'xtaguncha kechiktiradi. `leading` — birinchi chaqiruvda darhol ishga tushadi; `trailing` — jim davrdan keyin ishga tushadi.

```js
function debounce(fn, delay, { leading = false, trailing = true } = {}) {
  let timer = null;
  return function (...args) {
    const callNow = leading && timer === null; // taymer yo'q bo'lsa — birinchi chaqiruv

    clearTimeout(timer);
    timer = setTimeout(() => {
      timer = null;
      if (trailing && !callNow) fn.apply(this, args); // jim davrdan keyin
    }, delay);

    if (callNow) fn.apply(this, args); // leading: darhol
  };
}

// Sinov: tez yozilgan input'da faqat to'xtagandan keyin search ishga tushadi
const search = debounce((q) => console.log("qidiruv:", q), 300);
```

**Izoh:** `callNow` `leading` yoqilgan va taymer faol bo'lmaganda `true` bo'ladi. `trailing && !callNow` — agar `leading` allaqachon ishga tushgan bo'lsa, oxirida takror ishga tushmasligini ta'minlaydi.

---

## 5. `throttle` (trailing saqlanadi)

Har interval'da ko'pi bilan bir marta ishga tushadi, lekin oxirgi chaqiruvni yo'qotmaydi.

```js
function throttle(fn, limit) {
  let inThrottle = false;
  let lastArgs = null; // interval ichida kelgan oxirgi argumentlar

  return function (...args) {
    if (!inThrottle) {
      fn.apply(this, args);
      inThrottle = true;
      setTimeout(function tick() {
        if (lastArgs) {            // interval ichida chaqiruv bo'lgan bo'lsa
          fn.apply(this, lastArgs);
          lastArgs = null;
          setTimeout(tick, limit); // yana bir interval kutamiz
        } else {
          inThrottle = false;
        }
      }, limit);
    } else {
      lastArgs = args;             // saqlab qo'yamiz, trailing'da ishlatamiz
    }
  };
}
```

**Izoh:** `lastArgs` interval davomida kelgan oxirgi chaqiruvni saqlaydi. Interval tugaganda agar saqlangan chaqiruv bo'lsa, uni ishga tushiramiz va yangi interval boshlaymiz — shunday qilib trailing yo'qolmaydi.

---

## 6. `deepClone`

Siklik havolalarni `WeakMap` orqali, `Date`/`Map`/`Set` ni maxsus boshqaradi.

```js
function deepClone(value, seen = new WeakMap()) {
  // primitive va null — to'g'ridan-to'g'ri qaytaramiz
  if (value === null || typeof value !== "object") return value;

  // sikl: allaqachon nusxalangan bo'lsa, o'sha nusxani qaytaramiz
  if (seen.has(value)) return seen.get(value);

  // maxsus tiplar
  if (value instanceof Date) return new Date(value);
  if (value instanceof Map) {
    const m = new Map();
    seen.set(value, m);
    value.forEach((v, k) => m.set(deepClone(k, seen), deepClone(v, seen)));
    return m;
  }
  if (value instanceof Set) {
    const s = new Set();
    seen.set(value, s);
    value.forEach((v) => s.add(deepClone(v, seen)));
    return s;
  }

  const copy = Array.isArray(value) ? [] : {};
  seen.set(value, copy);               // rekursiyadan OLDIN ro'yxatga olamiz (sikl uchun)
  for (const key of Reflect.ownKeys(value)) {
    copy[key] = deepClone(value[key], seen);
  }
  return copy;
}

// Sinov: siklik havola
const a = { name: "a" };
a.self = a;
const c = deepClone(a);
c.self === c;        // => true (sikl saqlandi, cheksiz rekursiya yo'q)
c !== a;             // => true
```

**Izoh:** Eng muhim qism — `seen.set(value, copy)` ni rekursiyadan **oldin** chaqirish. Aks holda siklik havola cheksiz rekursiyaga olib keladi. `Reflect.ownKeys` Symbol key'larni ham qoplaydi.

---

## 7. `Promise.all`

Tartibni saqlovchi natijalar massivi bilan resolve bo'ladi; birinchi rejection'da darhol reject qiladi.

```js
function promiseAll(promises) {
  return new Promise((resolve, reject) => {
    const results = [];
    let remaining = promises.length;

    if (remaining === 0) return resolve(results); // bo'sh holat

    promises.forEach((p, i) => {
      Promise.resolve(p).then(            // p promise bo'lmasligi mumkin
        (val) => {
          results[i] = val;               // INDEX bo'yicha — tartib saqlanadi
          remaining -= 1;
          if (remaining === 0) resolve(results);
        },
        reject                            // birinchi rejection g'olib (fail-fast)
      );
    });
  });
}

// Sinov
promiseAll([Promise.resolve(1), 2, Promise.resolve(3)])
  .then((r) => console.log(r)); // => [1, 2, 3]
```

**Izoh:** `results[i] = val` — natijalarni settle bo'lish tartibida emas, asl index bo'yicha joylaydi. `Promise.resolve(p)` oddiy qiymatlarni ham promise'ga o'raydi.

---

## 8. `Promise.allSettled`

Hech qachon reject qilmaydi; har biri uchun `{status, value}` yoki `{status, reason}` qaytaradi.

```js
function allSettled(promises) {
  return Promise.all(
    promises.map((p) =>
      Promise.resolve(p).then(
        (value) => ({ status: "fulfilled", value }),
        (reason) => ({ status: "rejected", reason }) // rejection'ni qiymatga aylantiramiz
      )
    )
  );
}

// Sinov
allSettled([Promise.resolve(1), Promise.reject("xato")])
  .then((r) => console.log(r));
// => [{status:'fulfilled', value:1}, {status:'rejected', reason:'xato'}]
```

**Izoh:** Hiyla — har bir rejection'ni `.then` ning ikkinchi argumenti orqali **fulfilled** qiymatga aylantiramiz. Shuning uchun `Promise.all` hech qachon reject qilmaydi va biz hammasini kuta olamiz.

---

## 9. `MyPromise`

Holat, qiymat va navbatdagi callback'larni kuzatadi; settle'da ularni microtask orqali asinxron bo'shatadi. `then` chaining'ni qo'llab-quvvatlaydi.

```js
class MyPromise {
  constructor(executor) {
    this.state = "pending";
    this.value = undefined;
    this.cbs = []; // settle bo'lguncha kutayotgan callback'lar

    const settle = (state) => (value) => {
      if (this.state !== "pending") return; // faqat bir marta settle
      this.state = state;
      this.value = value;
      this.cbs.forEach((cb) => cb());        // kutayotganlarni bo'shatamiz
    };

    try {
      executor(settle("fulfilled"), settle("rejected"));
    } catch (e) {
      settle("rejected")(e);
    }
  }

  then(onF, onR) {
    return new MyPromise((resolve, reject) => {
      const handle = () => {
        queueMicrotask(() => {              // asinxron — microtask
          try {
            if (this.state === "fulfilled") {
              resolve(onF ? onF(this.value) : this.value);
            } else {
              onR ? resolve(onR(this.value)) : reject(this.value);
            }
          } catch (e) {
            reject(e);
          }
        });
      };
      // pending bo'lsa — navbatga, aks holda darhol
      this.state === "pending" ? this.cbs.push(handle) : handle();
    });
  }

  catch(onR) {
    return this.then(null, onR);
  }
}

// Sinov
new MyPromise((res) => setTimeout(() => res(1), 10))
  .then((v) => v + 1)
  .then((v) => console.log(v)); // => 2 (microtask orqali)
```

**Izoh:** `queueMicrotask` — `.then` callback'lari har doim asinxron ishlashini ta'minlaydi (Promises/A+ talabi). `catch` shunchaki `then(null, onR)`.

---

## 10. `memoize`

Bitta obyekt argument uchun `WeakMap`, ko'p/primitive argumentlar uchun `Map` + serializatsiya.

```js
function memoize(fn) {
  const primitiveCache = new Map();   // ko'p yoki primitive argumentlar
  const objectCache = new WeakMap();  // bitta obyekt argument (leak'siz)

  return function (...args) {
    // optimal yo'l: aniq bitta obyekt argument
    if (args.length === 1 && typeof args[0] === "object" && args[0] !== null) {
      const key = args[0];
      if (objectCache.has(key)) return objectCache.get(key);
      const result = fn.apply(this, args);
      objectCache.set(key, result);
      return result;
    }

    // umumiy yo'l: serializatsiya
    const key = JSON.stringify(args);
    if (primitiveCache.has(key)) return primitiveCache.get(key);
    const result = fn.apply(this, args);
    primitiveCache.set(key, result);
    return result;
  };
}

// Sinov
const square = memoize((n) => (console.log("hisoblandi"), n * n));
square(4); // "hisoblandi", => 16
square(4); // keshdan, => 16

const obj = { x: 1 };
const byObj = memoize((o) => o.x * 2);
byObj(obj); // hisoblanadi
byObj(obj); // keshdan (WeakMap orqali — obj GC qilinsa avto-tozalanadi)
```

**Izoh:** `WeakMap` yo'li memory leak'ni oldini oladi — kesh kaliti GC qilinganda yozuv ham yo'qoladi. `JSON.stringify` yo'li faqat serializatsiyalanadigan argumentlar uchun ishlaydi va key tartibiga sezgir.

---

← [JavaScript mavzusiga qaytish](../../frontend/03-javascript.md)
