# Memory Management — Masalalar Yechimlari

Bu fayl `03-memory-management.md` hujjatidagi masalalar yechimlarini o'z ichiga oladi. Har bir yechimda nafaqat kod, balki "nega aynan shunday" tushuntirilgan.

## Yechim 1: Reference tushunchasi

```js
let a = { n: 1 };
let b = a;        // b va a bir obyektni ko'rsatadi
b.n = 2;          // bir obyekt → a.n ham 2 bo'ladi
let c = b;        // c ham o'sha obyektni ko'rsatadi
c = { n: 9 };     // c QAYTA TAYINLANDI → endi yangi obyekt
console.log(a.n, b.n, c.n); // 2 2 9
```

**Tushuntirish:** `b = a` va `c = b` — havola nusxalash, yangi obyekt yaratmaydi. `b.n = 2` mutatsiya bo'lgani uchun bir obyektni ko'rsatayotgan `a.n` ham 2 bo'ladi. Ammo `c = {n: 9}` — qayta tayinlash; `c` endi butunlay yangi obyektni ko'rsatadi va eski obyektga (a, b) ta'sir qilmaydi. Natija: `2 2 9`.

## Yechim 2: Value vs reference funksiyada

```js
// ISHLAMAYDI — primitive value bo'yicha uzatiladi
function swap(x, y) {
  let t = x;
  x = y;
  y = t;
  // faqat lokal nusxalar almashdi, tashqaridagilarga ta'sir yo'q
}
let a = 1, b = 2;
swap(a, b);
console.log(a, b); // 1 2 — o'zgarmadi
```

**Nega:** Primitive funksiyaga value (nusxa) bo'yicha uzatiladi. Funksiya ichidagi `x`, `y` — original'larning nusxasi. Ularni almashtirish lokal nusxalarni almashtiradi, original o'zgarmaydi.

```js
// ISHLAYDI — obyekt mutatsiyasi orqali
function swap(obj) {
  let t = obj.a;
  obj.a = obj.b;
  obj.b = t;
}
let data = { a: 1, b: 2 };
swap(data);
console.log(data.a, data.b); // 2 1 — o'zgardi

// yoki return + destructuring (idiomatik JS)
let [x, y] = [1, 2];
[x, y] = [y, x];
console.log(x, y); // 2 1
```

**Tushuntirish:** Obyekt reference bo'yicha uzatilgani uchun uning xususiyatlarini mutatsiya qilish original'ga ta'sir qiladi. Zamonaviy JS'da array destructuring (`[x, y] = [y, x]`) eng toza yo'l.

## Yechim 3: Cycle va GC

```js
let a = {};
let b = {};
a.ref = b;
b.ref = a;
a = null;
b = null;
```

**Reference counting:** Bu algoritm obyektlarni **tozalay olmaydi**. `a = null` dan keyin ham `b.ref` hali birinchi obyektni ko'rsatadi (refcount = 1), `b = null` dan keyin ham `a.ref` ikkinchisini ko'rsatadi (refcount = 1). Ikkalasi bir-birini ushlab turadi → refcount hech qachon 0 ga tushmaydi → leak.

**Mark-and-sweep:** Bu algoritm obyektlarni **tozalaydi**. Mark bosqichida root'lardan (global'lar, stack) boshlab tekshiriladi. `a` va `b` `null` bo'lgach, bu ikki obyektga root'lardan **yetib bo'lmaydi** — garchi ular bir-birini ko'rsatsa ham. Shuning uchun ular belgilanmaydi va sweep bosqichida o'chiriladi.

**Xulosa:** Aynan shu farq tufayli V8 reference counting emas, mark-and-sweep ishlatadi — u cycle muammosidan xoli.

## Yechim 4: Memory leak topish

**Muammo:** Har `process` chaqiruvida yangi `setInterval` yaratiladi, lekin hech qachon `clearInterval` qilinmaydi. Har bir interval `cache[id]` ni (demak `data` ni) abadiy ushlab turadi. `cache`'dan ham hech narsa o'chirilmaydi. Ikki tomonlama leak.

```js
const cache = new Map();
const timers = new Map();

function process(id, data) {
  cache.set(id, data);
  const timer = setInterval(() => console.log(cache.get(id)), 1000);
  timers.set(id, timer);
}

// kerak bo'lmaganda tozalash funksiyasi
function release(id) {
  clearInterval(timers.get(id));
  timers.delete(id);
  cache.delete(id);
}
```

**Tushuntirish:** (1) Timer'ni saqlab, `release` da `clearInterval` qilamiz — endi interval va u ushlab turgan `data` GC'ga ochiq. (2) `cache.delete(id)` bilan ma'lumotni ham bo'shatamiz. `Map` ishlatish `delete` ni qulaylashtiradi. Agar kalitlar obyekt bo'lganida `WeakMap` yanada yaxshi yechim bo'lardi.

## Yechim 5: Closure leak

```js
// LEAK — closure katta massivni keraksiz ushlaydi
function createHandler() {
  const hugeData = new Array(1000000).fill("x");
  return function () {
    // hugeData ishlatilmasa ham, closure uni ushlab turadi
    return "natija";
  };
}
const handler = createHandler(); // hugeData abadiy xotirada
```

**Tuzatilgan:**

```js
function createHandler() {
  const hugeData = new Array(1000000).fill("x");
  const summary = hugeData.length; // faqat kerakli qiymatni olamiz
  // hugeData endi closure'da ushlanmaydi, GC tozalaydi
  return function () {
    return `natija: ${summary}`;
  };
}
const handler = createHandler();
```

**Tushuntirish:** Closure o'zining tashqi scope'idagi o'zgaruvchilarni ushlab turadi. Agar `hugeData` butun kerak bo'lmasa, undan faqat zarur qismni (`summary`) ajratib oling — shunda butun massivga reference qolmaydi va GC uni tozalaydi.

## Yechim 6: Stack overflow

```js
// LEAK / xato — base case yo'q, cheksiz rekursiya
function countdown(n) {
  console.log(n);
  return countdown(n - 1); // hech qachon to'xtamaydi
}
// countdown(5); → RangeError: Maximum call stack size exceeded
```

**Tuzatilgan:**

```js
function countdown(n) {
  if (n < 0) return; // base case — rekursiyani to'xtatadi
  console.log(n);
  return countdown(n - 1);
}
countdown(5); // 5 4 3 2 1 0
```

**Tushuntirish:** Stack overflow xatosi **stack segmentiga** tegishli. Har bir rekursiv chaqiruv yangi stack frame qo'shadi; base case bo'lmasa frame'lar to'planib, stack sig'imi (~1 MB) oshib ketadi. Yechim — har doim to'xtatuvchi shart (base case) qo'yish.

## Yechim 7: Detached DOM

```js
// LEAK — DOM'dan olib tashlandi, lekin JS reference tutadi
let detached = document.getElementById("item");
document.body.removeChild(detached);
// detached o'zgaruvchisi hali tugunni (va bolalarini) ushlaydi → leak
```

**Tuzatilgan:**

```js
let node = document.getElementById("item");
document.body.removeChild(node);
node = null; // reference uzildi → GC tugunni tozalashi mumkin
```

**Tushuntirish:** DOM tugunni daraxtdan olib tashlash GC uchun yetarli emas — JS o'zgaruvchi hali reference tutsa, tugun xotirada qoladi (detached node). Ishlatib bo'lgach o'zgaruvchini `null` qiling. Event listener'lar ham shunday tutib turishi mumkin — ularni `removeEventListener` bilan tozalang.

## Yechim 8: Deep clone

```js
const original = { a: 1, b: { c: 2 } };

// SHALLOW — yetarli emas
const shallow = { ...original };
shallow.b.c = 99;
console.log(original.b.c); // 99 — original buzildi!

// DEEP — to'g'ri
const deep = structuredClone(original); // zamonaviy, tavsiya etiladi
deep.b.c = 99;
console.log(original.b.c); // 2 — saqlanib qoldi
```

**Nega shallow yetarli emas:** `{...original}` faqat birinchi darajani nusxalaydi. Ichki obyekt `b` esa **reference** bo'yicha nusxalanadi — `shallow.b` va `original.b` bir xil obyektni ko'rsatadi. `shallow.b.c` ni o'zgartirsangiz, original'ning `b.c` ham o'zgaradi.

**Yechim variantlari:** `structuredClone(obj)` (eng yaxshi, native, cycle'larni qo'llab-quvvatlaydi); `JSON.parse(JSON.stringify(obj))` (sodda, lekin funksiya, `undefined`, `Date`, cycle bilan ishlamaydi); yoki rekursiv qo'lda yozish.

## Yechim 9: Segmentlarni aniqlash

```c
int g = 5;              // DATA segment — global, initsializatsiya qilingan
static int s;           // BSS segment  — static, init qilinmagan (0 ga to'la)
int main() {
  int local = 1;        // STACK — lokal o'zgaruvchi
  int *p = malloc(4);   // p (pointer) STACK'da, p ko'rsatgan 4 bayt HEAP'da
}
```

**Tushuntirish:**
- `g = 5` — qiymat berilgan global → **Data**.
- `static int s` — qiymat berilmagan static → **BSS** (dastur boshida 0).
- `local` — funksiya ichidagi oddiy o'zgaruvchi → **Stack** (frame ichida).
- `p` — pointer o'zgaruvchisining o'zi stack'da, lekin `malloc` qaytargan 4 bayt **Heap**'da. Pointer heap'dagi blokka ko'rsatadi.

## Yechim 10: GC trigger

```js
let data = { huge: new Array(1e6) };
window.handler = () => console.log("hi");
data = null;
```

**Tozalanadimi?** Ha, `data` ko'rsatgan obyekt **tozalanadi**. `data = null` dan keyin `{ huge: ... }` obyektiga root'lardan boshqa reference qolmaydi → GC uni o'chiradi. `window.handler` arrow funksiyasi esa `huge` ni ushlamaydi (uni umuman ishlatmaydi), shuning uchun u leak'ga sabab bo'lmaydi.

**Diqqat:** Agar `handler` ichida `data.huge` ishlatilganida, closure orqali `data` obyektini ushlab turardi va `data = null` qilsangiz ham `window.handler` (global root) tufayli tozalanmasdi. U holda tozalash uchun `window.handler = null` ham qilish kerak bo'lardi.

```js
// leak holati (taqqoslash uchun):
let data = { huge: new Array(1e6) };
window.handler = () => console.log(data.huge.length); // data'ni ushlaydi
data = null; // YETARLI EMAS — handler hali ushlaydi
// to'liq tozalash:
window.handler = null;
```

← [CS Asoslari bo'limiga qaytish](../../cs-fundamentals/README.md)
