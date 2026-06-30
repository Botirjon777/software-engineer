# JavaScript — Chuqur

JavaScript intervyularda eng ko'p so'raladigan mavzu. Bu fayl tilning ichki mexanizmlarini — tiplar, scope, closure, `this`, prototype, event loop, async va functional toolkit — chuqur, savol-javob ko'rinishida yoritadi. Maqsad: nafaqat "nima", balki "nega shunday" degan savolga javob bera olish.

## Mundarija

- [Primitive vs Reference](#primitive-vs-reference)
- [Type coercion va `==` vs `===`](#type-coercion-va--vs-)
- [Truthy va Falsy](#truthy-va-falsy)
- [`var` / `let` / `const` va TDZ](#var--let--const-va-tdz)
- [Hoisting](#hoisting)
- [Scope](#scope)
- [Closure](#closure)
- [`this` kalit so'zi](#this-kalit-sozi)
- [Prototype va prototypal inheritance](#prototype-va-prototypal-inheritance)
- [`new` kalit so'zi](#new-kalit-sozi)
- [ES6 class va inheritance](#es6-class-va-inheritance)
- [Event loop](#event-loop)
- [Callback va callback hell](#callback-va-callback-hell)
- [Promises](#promises)
- [async / await](#async--await)
- [Generator va iterator](#generator-va-iterator)
- [Higher-order functions va map/filter/reduce](#higher-order-functions-va-mapfilterreduce)
- [Currying](#currying)
- [Debounce va throttle](#debounce-va-throttle)
- [Destructuring, spread va rest](#destructuring-spread-va-rest)
- [Modullar: ESM vs CommonJS](#modullar-esm-vs-commonjs)
- [Map / Set / WeakMap](#map--set--weakmap)
- [Deep vs shallow copy](#deep-vs-shallow-copy)
- [Event delegation](#event-delegation)
- [`bind` / `call` / `apply` implementatsiyasi](#bind--call--apply-implementatsiyasi)
- [Memoization](#memoization)
- [Immutability va pure functions](#immutability-va-pure-functions)
- [Masalalar](#masalalar)

---

## Primitive vs Reference

**💡 Tushuncha:** JavaScriptda qiymatlar ikki turkumga bo'linadi. **Primitive** tiplar (`string`, `number`, `boolean`, `null`, `undefined`, `symbol`, `bigint`) o'zgarmas (immutable) va **qiymat bo'yicha** (by value) nusxalanadi. **Reference** tiplar (`object`, `array`, `function`) heap'da saqlanadi va **havola bo'yicha** (by reference) nusxalanadi — o'zgaruvchi pointer'ni saqlaydi.

```js
let a = 10;
let b = a;      // b qiymatning nusxasini oladi
b = 20;
console.log(a); // => 10  (a o'zgarmadi)

let o1 = { n: 10 };
let o2 = o1;     // o2 AYNAN SHU obyektga ishora qiladi
o2.n = 20;
console.log(o1.n); // => 20  (ikkalasi ham o'zgarishni ko'radi)
```

### ❓ Value va reference semantikasi farqi nimada?

**✅ Javob:** Primitive'lar **qiymat bo'yicha** beriladi: har bir o'zgaruvchi mustaqil nusxa oladi, shuning uchun birini o'zgartirish boshqasiga ta'sir qilmaydi. Obyektlar **havola bo'yicha** beriladi: o'zgaruvchi heap'dagi obyektga pointer saqlaydi, shuning uchun ikki o'zgaruvchi bitta obyektga "alias" bo'lishi mumkin. Funksiya argumentlari ham shu qoidaga bo'ysunadi — JavaScript har doim **pass-by-value**, lekin obyektlar uchun *uzatilayotgan qiymat — bu havolaning o'zi*.

```js
function mutate(obj) { obj.x = 99; }       // umumiy obyektni o'zgartiradi
function reassign(obj) { obj = { x: 0 }; } // faqat LOKAL nusxani qayta bog'laydi

const t = { x: 1 };
mutate(t);   console.log(t.x); // => 99
reassign(t); console.log(t.x); // => 99  (chaqiruvchining havolasi o'zgarmadi)
```

**⚠️ Ehtiyot bo'l:** Ko'pchilik "JS pass by reference" deydi. Bu noto'g'ri — bu **havolaning qiymati bo'yicha uzatish** (ba'zan "call by sharing" deyiladi). Funksiya ichida parametrni qayta tayinlash chaqiruvchiga ta'sir qilmaydi; faqat ko'rsatilgan obyektni mutatsiya qilish ko'rinadi.

### ❓ Qiymatning tipini qanday tekshirasiz?

**✅ Javob:** Primitive'lar va funksiyalar uchun `typeof`; massivlar uchun `Array.isArray()`; obyektlarni aniqroq ajratish uchun `instanceof` yoki `Object.prototype.toString.call()`.

```js
typeof 42;            // => "number"
typeof "x";           // => "string"
typeof undefined;     // => "undefined"
typeof null;          // => "object"   ⚠️ tarixiy bug
typeof function(){};  // => "function"
typeof Symbol();      // => "symbol"
typeof 10n;           // => "bigint"
Array.isArray([]);    // => true
Object.prototype.toString.call([]); // => "[object Array]"
```

**⚠️ Ehtiyot bo'l:** `typeof null === "object"` — bu eski bug bo'lib, orqaga moslik uchun saqlab qolingan. `null`ni tekshirish uchun `value === null` ishlating.

---

## Type coercion va `==` vs `===`

**💡 Tushuncha:** **Coercion** — bu yashirin (implicit) tip konvertatsiyasi. `==` (loose equality) operandlarni umumiy tipga keltirib solishtiradi; `===` (strict equality) tip **va** qiymatni hech qanday coercion'siz solishtiradi.

### ❓ `==` va `===` farqini tushuntiring.

**✅ Javob:** `===` faqat tip va qiymat mos kelganda `true` qaytaradi. `==` esa Abstract Equality algoritmini qo'llaydi: agar tiplar farq qilsa, coercion qiladi (number va string → number, boolean → number, obyekt → primitive `valueOf`/`toString` orqali). Kutilmagan coercion'lardan qochish uchun doim `===` afzal. `==` ning yagona idiomatik foydali holati — `x == null`, bu `null` va `undefined` ikkalasi uchun ham `true`.

```js
0 == "";       // => true   ("" → 0)
0 == "0";      // => true   ("0" → 0)
"" == "0";     // => false  (ikkalasi string, coercion yo'q, "" !== "0")
0 == false;    // => true   (false → 0)
null == undefined; // => true
null == 0;     // => false  (null faqat null/undefined bilan loosely teng)
[] == ![];     // => true   (![] → false → 0; [] → "" → 0)
NaN === NaN;   // => false  (Number.isNaN ishlating)
```

**⚠️ Ehtiyot bo'l:** `[] == ![]` — klassik tuzoq. `![]` avval `false` ga aylanadi (massivlar truthy). Keyin `[] == false` → `[] == 0` → `"" == 0` → `0 == 0` → `true`.

### ❓ Nega `0.1 + 0.2 !== 0.3`?

**✅ Javob:** Sonlar IEEE-754 64-bit floating point bilan saqlanadi, bu `0.1` yoki `0.2` ni binar tizimda aniq ifodalay olmaydi. Natija `0.30000000000000004`. Float'larni tolerantlik (epsilon) bilan solishtiring: `Math.abs(a - b) < Number.EPSILON`.

---

## Truthy va Falsy

**💡 Tushuncha:** Boolean kontekstda har bir qiymat yo truthy, yo falsy. Aniq **sakkizta** falsy qiymat mavjud:

```js
false, 0, -0, 0n, "", null, undefined, NaN
```

Qolgan hammasi truthy — shu jumladan `"0"`, `"false"`, `[]`, `{}` va barcha funksiyalar.

### ❓ Falsy qiymatlarni sanang. `[]` truthymi?

**✅ Javob:** Falsy qiymatlar: `false`, `0`, `-0`, `0n` (BigInt nol), `""`, `null`, `undefined` va `NaN`. Bo'sh massiv `[]` va bo'sh obyekt `{}` — **truthy**, chunki ular obyekt, va barcha obyektlar truthy. Bu ko'pchilikni adashtiradi, chunki `[] == false` `true` (loose-equality coercion), lekin `Boolean([])` `true`.

```js
if ([]) console.log("ishlaydi");   // ishlaydi (truthy)
console.log([] == false);           // => true (coercion, boshqa yo'l)
```

**⚠️ Ehtiyot bo'l:** `0` yoki `""` haqiqiy qiymat bo'lishi mumkin bo'lganda `||` o'rniga `??` (nullish coalescing) ishlating:

```js
const count = 0;
count || 10;   // => 10  ⚠️ noto'g'ri, 0 falsy
count ?? 10;   // => 0   ✅ faqat null/undefined fallback'ni ishga tushiradi
```

---

## `var` / `let` / `const` va TDZ

**💡 Tushuncha:** `var` — function-scoped va `undefined` boshlang'ich qiymat bilan hoist qilinadi. `let` va `const` — block-scoped va hoist qilinadi, lekin **initsializatsiyalanmaydi** — deklaratsiyadan oldin kirish xato beradi. `const` **qayta tayinlashni** taqiqlaydi (mutatsiyani emas).

### ❓ Temporal Dead Zone (TDZ) nima?

**✅ Javob:** TDZ — bu blok boshlanishidan `let`/`const` deklaratsiyasi initsializatsiyalangunigacha bo'lgan soha. Binding mavjud (u hoist qilingan), lekin unga kirib bo'lmaydi; o'qishga urinish `ReferenceError` beradi. Bu use-before-declare holatini sokin `undefined` o'rniga aniq xatoga aylantiradi.

```js
console.log(x); // => undefined (var hoist va init qilingan)
var x = 1;

console.log(y); // ❌ ReferenceError: Cannot access 'y' before initialization
let y = 1;      // y bu satrdan yuqorida TDZ'da
```

### ❓ `const` obyektni immutable qiladimi?

**✅ Javob:** Yo'q. `const` faqat **binding'ni qayta tayinlashni** taqiqlaydi, qiymatni mutatsiya qilishni emas. `const` obyektning xususiyatlari hali ham o'zgartirilishi mumkin. Yuza (shallow) immutability uchun `Object.freeze()`; chuqur immutability uchun rekursiv freeze yoki kutubxona kerak.

```js
const obj = { a: 1 };
obj.a = 2;        // ✅ ruxsat (mutatsiya)
// obj = {};      // ❌ TypeError (qayta tayinlash)

Object.freeze(obj);
obj.a = 3;        // sokin e'tiborsiz qoldiriladi (strict mode'da throw)
```

---

## Hoisting

**💡 Tushuncha:** Kompilyatsiya fazasida deklaratsiyalar kod ishga tushishidan oldin ro'yxatga olinadi. `var` va function **deklaratsiyalari** hoist qilinadi; `let`/`const` TDZ ichiga hoist qilinadi; function **expression** va arrow funksiyalar callable sifatida hoist qilinmaydi (faqat ularning binding'i).

### ❓ Nima hoist qilinadi va qanday?

**✅ Javob:**
- **Function deklaratsiyalari** to'liq hoist qilinadi — ularni paydo bo'lishidan oldin chaqirsa bo'ladi.
- **`var`** deklaratsiyalari hoist qilinib `undefined` ga initsializatsiyalanadi.
- **`let`/`const`** hoist qilinadi, lekin initsializatsiyalanmaydi (TDZ).
- **Function/arrow expression** faqat o'zgaruvchi binding'ini hoist qiladi, shuning uchun ularni erta chaqirsa xato.

```js
foo();             // => "ishlaydi" (deklaratsiya hoist qilingan)
function foo() { console.log("ishlaydi"); }

bar();             // ❌ TypeError: bar is not a function
var bar = () => {};// faqat `var bar` (=> undefined) hoist qilingan
```

**⚠️ Ehtiyot bo'l:** `var` va function deklaratsiyasi bir xil nomga ega bo'lsa, hoisting paytida function deklaratsiyasi g'olib chiqadi, lekin keyingi tayinlash runtime'da uni qayta yozadi.

---

## Scope

**💡 Tushuncha:** Scope o'zgaruvchining ko'rinish sohasini belgilaydi. JS'da **global**, **function** va (ES6'dan beri) **block** scope bor. Lexical scoping — ichki scope'lar tashqi scope o'zgaruvchilarini o'qiy oladi, va bu kod **qayerda yozilganiga** qarab hal qilinadi, qayerda chaqirilganiga emas.

### ❓ Lexical scope nima?

**✅ Javob:** Lexical (statik) scope — funksiyaning kirish mumkin bo'lgan o'zgaruvchilari uning manba kodidagi fizik joylashuviga qarab belgilanadi. O'zgaruvchi joriy scope'da topilmasa, engine **scope chain** bo'ylab tashqi scope'larga ko'tariladi, global'gacha; topilmasa `ReferenceError`. Bu closure'lar asosini tashkil etadi.

```js
const x = "tashqi";
function outer() {
  const y = "o'rta";
  function inner() {
    console.log(x, y); // ikkalasini scope chain orqali ko'radi
  }
  return inner;
}
outer()(); // => "tashqi o'rta"
```

---

## Closure

**💡 Tushuncha:** **Closure** — bu funksiya va uni o'rab turgan lexical environment'ga havolalar birikmasi. Ichki funksiya tashqi o'zgaruvchilarni tashqi funksiya tugaganidan keyin ham "eslab qoladi", chunki closure ularga havola qilgancha ular tirik qoladi.

### ❓ Closure nima? Amaliy misol bering.

**✅ Javob:** Closure — funksiya o'rab turuvchi scope tugaganidan keyin ham uning o'zgaruvchilariga kirishni saqlab qolganda hosil bo'ladi. Amaliy holatlar: ma'lumot maxfiyligi (module pattern), function factory, callback'larda holat saqlash, memoization va currying.

```js
function makeCounter() {
  let count = 0;                 // maxfiy holat
  return {
    inc: () => ++count,
    get: () => count,
  };
}
const c = makeCounter();
c.inc(); c.inc();
console.log(c.get()); // => 2   (count tashqaridan kirib bo'lmaydi)
```

### ❓ Quyidagi klassik loop bug'ni tuzating.

**✅ Javob:** `var` bilan barcha callback'lar **bitta** function-scoped `i` ni baham ko'radi, u ishga tushganda `3` bo'ladi. Har iteratsiyada yangi block-scoped binding hosil qilish uchun `let` ishlating (yoki pre-ES6'da IIFE).

```js
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0); // => 3, 3, 3
}

for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0); // => 0, 1, 2 ✅
}
```

**⚠️ Ehtiyot bo'l:** Closure'lar memory leak'ga sabab bo'lishi mumkin — uzoq yashovchi closure katta obyektlarni (masalan DOM tugunlari) ushlab tursa va ular hech qachon bo'shatilmasa. Kerak bo'lmagan havolalarni `null` qiling.

---

## `this` kalit so'zi

**💡 Tushuncha:** `this` funksiya **qanday chaqirilganiga** qarab belgilanadi, qayerda yozilganiga emas — bundan arrow funksiyalar mustasno, ular `this` ni lexical tarzda oladi. Beshta binding qoidasi bor: default, implicit, explicit, `new` va lexical (arrow).

### ❓ `this` ni barcha kontekstlarda tushuntiring.

**✅ Javob:**
1. **Global / mustaqil chaqiruv:** non-strict mode'da `this` — global obyekt (`window`/`globalThis`); strict mode va ES module'da `undefined`.
2. **Method chaqiruvi** (`obj.fn()`): `this` — nuqtadan oldingi obyekt (implicit binding).
3. **`call` / `apply` / `bind`:** `this` ni aniq belgilaydi (explicit binding).
4. **`new Fn()`:** `this` — yangi yaratilgan instance.
5. **Arrow function:** o'zining `this` i yo'q; o'rab turuvchi lexical scope'dan oladi.

```js
const obj = {
  val: 42,
  regular() { return this.val; },
  arrow: () => this?.val,        // `this` tashqi scope, obj emas
};
obj.regular(); // => 42
obj.arrow();   // => undefined (arrow obj'ga bog'lanmaydi)

const fn = obj.regular;
fn();          // => undefined / TypeError — `this` yo'qoldi
```

### ❓ Nega arrow funksiyalar callback'larda `this` muammosini hal qiladi?

**✅ Javob:** Arrow funksiyalar o'z `this` ini yaratmaydi; ularni aniqlash vaqtidagi o'rab turuvchi scope'dan oladi. Bu `setTimeout`, event handler yoki array callback ichidagi klassik "lost `this`" muammosini hal qiladi, u yerda oddiy funksiya default-bind bo'lib qolardi.

```js
class Timer {
  constructor() { this.secs = 0; }
  start() {
    setInterval(() => { this.secs++; }, 1000); // arrow `this` = Timer instance'ni saqlaydi
  }
}
```

### ❓ `call`, `apply`, `bind` farqi nimada?

**✅ Javob:** Uchchalasi ham `this` ni aniq belgilaydi. `call(ctx, a, b)` — argumentlarni alohida beradi va darhol chaqiradi. `apply(ctx, [a, b])` — argumentlarni massiv sifatida beradi va darhol chaqiradi. `bind(ctx, a)` — chaqirmaydi, balki `this` (va ixtiyoriy oldindan berilgan argumentlar) ga doimiy bog'langan **yangi funksiya** qaytaradi.

```js
function greet(g) { return `${g}, ${this.name}`; }
const u = { name: "Ali" };
greet.call(u, "Salom");     // => "Salom, Ali"
greet.apply(u, ["Salom"]);  // => "Salom, Ali"
const bound = greet.bind(u);
bound("Salom");             // => "Salom, Ali"
```

**⚠️ Ehtiyot bo'l:** DOM event handler oddiy `function` bo'lsa, `this` — eventni yuborgan element. Arrow handler'da `this` — lexical scope (element emas); buning o'rniga `event.currentTarget` ishlating.

---

## Prototype va prototypal inheritance

**💡 Tushuncha:** Har bir obyektda ichki `[[Prototype]]` havolasi bor (`__proto__` orqali ochiq, `Object.getPrototypeOf` orqali olinadi). Obyektda topilmagan xususiyat qidiruvi shu **prototype chain** bo'ylab yuqoriga ko'tariladi, topilgunicha yoki `null` ga yetgunicha. Funksiyalarda `prototype` xususiyati bor, u `new` bilan chaqirilganda ishlatiladi.

### ❓ Prototype chain qanday ishlaydi?

**✅ Javob:** `obj.prop` ga kirganda, engine avval `obj` ning o'z xususiyatlarini tekshiradi; topilmasa `obj.__proto__` ga, keyin uning prototype'iga va hokazo `Object.prototype` gacha (uning prototype'i `null`). Inheritance va method ulashish shu tarzda ishlaydi — instance method'lar prototype'da bir marta yashaydi, har instance'da emas.

```js
function Animal(name) { this.name = name; }
Animal.prototype.speak = function () { return `${this.name} ovoz chiqaradi`; };

const dog = new Animal("Rex");
dog.speak();                          // => "Rex ovoz chiqaradi"
dog.hasOwnProperty("speak");          // => false (u prototype'da)
Object.getPrototypeOf(dog) === Animal.prototype; // => true
```

### ❓ `__proto__` va `prototype` farqi?

**✅ Javob:** `prototype` — **constructor funksiyalardagi** xususiyat; u `new` bilan yaratilgan instance'larning `[[Prototype]]` iga aylanadi. `__proto__` — **har bir obyektdagi** accessor, uning haqiqiy prototype'iga ishora qiladi. Qisqasi: `instance.__proto__ === Constructor.prototype`.

**⚠️ Ehtiyot bo'l:** `Object.create(null)` prototype'siz obyekt yaratadi — toza dictionary uchun foydali (inherited `toString` yo'q, prototype-pollution xavfi yo'q), lekin unda `Object` method'lari bo'lmaydi.

---

## `new` kalit so'zi

**💡 Tushuncha:** `new` funksiya chaqiruvini obyekt qurishga aylantiradi.

### ❓ `new` qadam-baqadam nima qiladi?

**✅ Javob:** `new Fn(args)`:
1. Yangi bo'sh obyekt yaratadi.
2. Uning `[[Prototype]]` ini `Fn.prototype` ga o'rnatadi.
3. `Fn` ni `this` yangi obyektga bog'langan holda chaqiradi.
4. Yangi obyektni qaytaradi — agar constructor aniq non-primitive obyekt qaytarmasa; aks holda o'sha obyekt qaytariladi.

```js
function myNew(Ctor, ...args) {
  const obj = Object.create(Ctor.prototype); // 1-2 qadam
  const result = Ctor.apply(obj, args);      // 3 qadam
  return (result !== null && typeof result === "object") ? result : obj; // 4 qadam
}
```

**⚠️ Ehtiyot bo'l:** Non-arrow constructor'da `new` ni unutish non-strict mode'da `this` ni global obyekt qiladi va sokin ravishda global'larni ifloslantiradi. Class'lar `new` siz chaqirilsa throw qiladi va buni oldini oladi.

---

## ES6 class va inheritance

**💡 Tushuncha:** `class` — prototype ustidagi syntactic sugar — method'lar `Class.prototype` ga tushadi. `extends` prototype chain'ni o'rnatadi; `super` parent constructor/method'ni chaqiradi.

### ❓ Class'lar "shunchaki" syntactic sugar'mi?

**✅ Javob:** Asosan ha. Method'lar hali ham prototype'da saqlanadi va inheritance hali ham prototype chain'ni ishlatadi. Lekin class'lar haqiqiy semantika qo'shadi: tanasi strict mode'da ishlaydi, hoist qilinmaydi (callable emas), `new` bilan chaqirilishi shart, va `super`, private field (`#x`), static member, getter/setter'ni toza qo'llab-quvvatlaydi.

```js
class Animal {
  #id = Math.random();          // private field
  constructor(name) { this.name = name; }
  speak() { return `${this.name} ovoz chiqaradi`; }
  static create(n) { return new Animal(n); }
}

class Dog extends Animal {
  speak() { return `${super.speak()} — vov`; } // super chaqiruvi
}
new Dog("Rex").speak(); // => "Rex ovoz chiqaradi — vov"
```

**⚠️ Ehtiyot bo'l:** Private field (`#id`) haqiqatan ham private — class tashqarisida kirib bo'lmaydi va enumerable emas. Tashqarida `obj.#id` ga kirish syntax error beradi.

---

## Event loop

**💡 Tushuncha:** JS single-threaded. **Call stack** sinxron kodni ishlatadi. Async callback'lar yoki **macrotask queue** (`setTimeout`, `setInterval`, I/O, UI event) yoki **microtask queue** (Promise `.then`, `queueMicrotask`, `await` davomi, `MutationObserver`) ga navbatga turadi. Har macrotask'dan keyin loop render yoki keyingi macrotask'dan oldin **butun microtask queue'ni bo'shatadi**.

### ❓ Quyidagi kod qaysi tartibda log qiladi?

**✅ Javob:** Avval sinxron kod, keyin **barcha** microtask'lar, keyin macrotask'lar. Shuning uchun Promise `0` kechikishli `setTimeout` dan ham oldin.

```js
console.log("1");
setTimeout(() => console.log("2"), 0);
Promise.resolve().then(() => console.log("3"));
console.log("4");
// => 1, 4, 3, 2
```

### ❓ Qiyin tartib — buni iz qilib chiqing.

**✅ Javob:** `1` va `5` sinxron. `4` (microtask) `2` (macrotask) dan oldin. `setTimeout` ichida `2` log qilinib `3` rejalashtirilgandan so'ng, loop keyingi macrotask'dan oldin microtask (`3`) ni bo'shatadi.

```js
console.log("1");                                    // sinxron
setTimeout(() => {                                   // macrotask
  console.log("2");
  Promise.resolve().then(() => console.log("3"));    // macrotask ichidagi microtask
}, 0);
Promise.resolve().then(() => console.log("4"));      // microtask
console.log("5");                                    // sinxron
// => 1, 5, 4, 2, 3
```

### ❓ Yanada qiyin — `async/await` aralashgan tartib.

**✅ Javob:** `await` dan keyingi qism microtask sifatida rejalashtiriladi. `async1` `await async2()` ga yetganda, `async2` ichi sinxron ishlaydi, keyin `async1` ning qolgani microtask'ga qo'yiladi.

```js
async function async1() {
  console.log("A");
  await async2();
  console.log("B");        // await'dan keyin → microtask
}
async function async2() { console.log("C"); }

console.log("D");
async1();
Promise.resolve().then(() => console.log("E"));
console.log("F");
// => D, A, C, F, B, E
```

`D` (sinxron) → `async1()` chaqirilib `A`, keyin `async2()` `C` chiqaradi va `await` `B` ni microtask'ga qo'yadi → `F` (sinxron) → microtask'lar tartibi: `B` (avval qo'yilgan), keyin `E`.

**⚠️ Ehtiyot bo'l:** Uzoq ishlovchi sinxron task hamma narsani bloklaydi — taymerlar, render, click. Va cheksiz microtask loop (har doim yangi microtask rejalashtiruvchi `.then`) macrotask'larni **ochlikka** uchratib (starve) sahifani muzlatishi mumkin, chunki loop microtask'lar tugaguncha keyingisiga o'tmaydi.

---

## Callback va callback hell

**💡 Tushuncha:** **Callback** — keyinroq chaqirish uchun uzatiladigan funksiya. Ko'p async callback'larni ichma-ich joylash chuqur indentatsiyali "callback hell" (pyramid of doom) hosil qiladi, uni o'qish va xato boshqarish qiyin.

### ❓ Callback hell nima va undan qanday qochasiz?

**✅ Javob:** Callback hell — chuqur joylashgan callback'lar, har biri oldingisiga bog'liq, natijada o'qib bo'lmaydigan, xatoga moyil, takroriy error handling'li kod. Yechimlar: callback'larni nomlash va tekislash, modullash, yoki — yaxshirog'i — Promise (chaining) yoki `async/await` bilan chiziqli, `try/catch` qilinadigan oqim.

```js
// 😖 callback hell
getUser(id, (u) => {
  getPosts(u, (p) => {
    getComments(p, (c) => { /* ... */ });
  });
});

// ✅ async/await
const u = await getUser(id);
const p = await getPosts(u);
const c = await getComments(p);
```

---

## Promises

**💡 Tushuncha:** **Promise** — kelajakdagi qiymatni ifodalaydi, uch holati bor: **pending**, **fulfilled** yoki **rejected**. Bir marta settle bo'lgach o'zgarmas. `.then`/`.catch`/`.finally` reaksiyalarni ro'yxatga oladi va yangi promise qaytaradi, bu chaining'ga imkon beradi.

### ❓ Promise holatlari va chaining'ni tushuntiring.

**✅ Javob:** Promise **pending** dan boshlanadi, keyin aniq bir marta **fulfilled** (qiymat bilan) yoki **rejected** (sabab bilan) ga settle bo'ladi. `.then(onFulfilled, onRejected)` yangi promise qaytaradi: qiymat qaytarsa — pastga uzatadi; promise qaytarsa — kutadi; throw qilsa — keyingisini reject qiladi. Oxiridagi yagona `.catch` zanjirdagi har qanday rejection'ni ushlaydi.

```js
fetch(url)
  .then((res) => res.json())
  .then((data) => transform(data))
  .catch((err) => console.error(err))   // har qanday oldingi rejection'ni ushlaydi
  .finally(() => hideSpinner());         // natijadan qat'i nazar ishlaydi
```

### ❓ `Promise.all`, `race`, `allSettled`, `any` ni solishtiring.

**✅ Javob:**
- **`Promise.all`** — barcha qiymatlar massivi bilan resolve bo'ladi; birinchi rejection'da **tez reject** qiladi (fail-fast).
- **`Promise.race`** — birinchi settle bo'lgan promise bilan settle bo'ladi (fulfilled *yoki* rejected).
- **`Promise.allSettled`** — hech qachon reject qilmaydi; har biri uchun `{status, value|reason}` bilan resolve bo'ladi.
- **`Promise.any`** — birinchi **fulfilled** qiymat bilan resolve bo'ladi; **hammasi** reject qilsagina `AggregateError` bilan reject qiladi.

```js
await Promise.all([a, b]);        // [aVal, bVal] yoki birinchi rejection
await Promise.race([a, b]);       // birinchi settle bo'lgan
await Promise.allSettled([a, b]); // [{status:'fulfilled',value}, {status:'rejected',reason}]
await Promise.any([a, b]);        // birinchi fulfilled
```

**⚠️ Ehtiyot bo'l:** Ushlanmagan rejection (`.catch` yo'q, `await` atrofida `try/catch` yo'q) `unhandledrejection` event'ini ishga tushiradi va Node'ni crash qilishi mumkin. Rejection'larni doim boshqaring.

---

## async / await

**💡 Tushuncha:** `async` funksiyalar har doim promise qaytaradi. `await` funksiyani promise settle bo'lguncha to'xtatadi va uning qiymatini ochadi — main thread'ni bloklamasdan (davomi microtask sifatida rejalashtiriladi).

### ❓ async/await bilan xatolarni qanday boshqarasiz?

**✅ Javob:** `await` chaqiruvlarini `try/catch` ga o'rang; rejected promise `try` ichida throw qilingan xatoga aylanadi. Mustaqil operatsiyalar uchun ularni ketma-ket emas, parallel ishlatish maqsadida `Promise.all` ni await qiling.

```js
async function load() {
  try {
    const [user, posts] = await Promise.all([getUser(), getPosts()]); // parallel
    return { user, posts };
  } catch (err) {
    console.error("load failed:", err);
    throw err; // qayta throw yoki fallback qaytaring
  }
}
```

**⚠️ Ehtiyot bo'l:** Loop ichida await qilish so'rovlarni **ketma-ket** ishlatadi. Parallel qilish uchun avval promise'larga map qiling, keyin `await Promise.all(promises)`. Yana, unutilgan `await` qiymat o'rniga pending promise'ning o'zini qaytaradi.

---

## Generator va iterator

**💡 Tushuncha:** **Iterator** — `next()` ga ega bo'lib `{value, done}` qaytaradi. **Iterable** — `[Symbol.iterator]` ni amalga oshiradi. **Generator** (`function*`) — to'xtab (`yield`) va davom etib turadigan funksiya, avtomatik iterator hosil qiladi.

### ❓ Generator nimaga kerak?

**✅ Javob:** Generator'lar qiymatlarni dangasalik bilan (talab bo'yicha, lazy) ishlab chiqaradi, bu cheksiz ketma-ketliklar, maxsus iteratsiya va kooperativ to'xtashga imkon beradi. `yield` ijroni to'xtatadi va qiymat qaytaradi; `next(v)` davom ettiradi, `v` esa `yield` ifodasining natijasi bo'ladi. `async/await` paydo bo'lishidan oldin async oqim boshqaruvida (masalan redux-saga) ishlatilgan.

```js
function* idGen() {
  let id = 1;
  while (true) yield id++;     // lazy, cheksiz
}
const gen = idGen();
gen.next().value; // => 1
gen.next().value; // => 2

// Maxsus iterable
const range = {
  from: 1, to: 3,
  *[Symbol.iterator]() { for (let i = this.from; i <= this.to; i++) yield i; },
};
[...range]; // => [1, 2, 3]
```

---

## Higher-order functions va map/filter/reduce

**💡 Tushuncha:** **Higher-order function** — funksiyani argument sifatida qabul qiladi va/yoki funksiya qaytaradi. `map`, `filter` va `reduce` — functional massiv ishlovining asosiy ishchi otlari.

### ❓ `reduce` orqali `map` ni amalga oshiring.

**✅ Javob:** `reduce` eng umumiy; `map`/`filter` uning maxsus holatlari.

```js
const map = (arr, fn) => arr.reduce((acc, x, i) => [...acc, fn(x, i)], []);
const filter = (arr, fn) => arr.reduce((acc, x) => (fn(x) ? [...acc, x] : acc), []);

[1, 2, 3].reduce((sum, x) => sum + x, 0); // => 6
```

**⚠️ Ehtiyot bo'l:** Boshlang'ich qiymatsiz `reduce` birinchi elementni accumulator sifatida ishlatadi va index 1 dan boshlaydi — boshlang'ich qiymatsiz bo'sh massivda chaqirish `TypeError` beradi. Doim boshlang'ich qiymat bering.

---

## Currying

**💡 Tushuncha:** **Currying** N argumentli funksiyani N ta zanjirlangan unary (bir argumentli) funksiyaga aylantiradi. **Partial application** ba'zi argumentlarni belgilab, qolganini kutuvchi funksiya qaytaradi.

### ❓ Umumiy `curry` ni amalga oshiring.

**✅ Javob:** Yetarli argument yig'ilgunicha (`fn.length` orqali) ularni to'plang, keyin chaqiring.

```js
function curry(fn) {
  return function curried(...args) {
    return args.length >= fn.length
      ? fn.apply(this, args)
      : (...rest) => curried.apply(this, [...args, ...rest]);
  };
}

const add = curry((a, b, c) => a + b + c);
add(1)(2)(3);   // => 6
add(1, 2)(3);   // => 6
add(1)(2, 3);   // => 6
```

---

## Debounce va throttle

**💡 Tushuncha:** **Debounce** funksiyani faollik to'xtaguncha kechiktiradi (jim davrdan keyin bir marta ishga tushadi) — qidiruv input va resize uchun yaxshi. **Throttle** ijroni har interval'da ko'pi bilan bir martaga cheklaydi — scroll va mousemove uchun yaxshi.

### ❓ Debounce va throttle'ni amalga oshiring.

**✅ Javob:**

```js
function debounce(fn, delay) {
  let timer;
  return function (...args) {
    clearTimeout(timer);
    timer = setTimeout(() => fn.apply(this, args), delay);
  };
}

function throttle(fn, limit) {
  let inThrottle = false;
  return function (...args) {
    if (!inThrottle) {
      fn.apply(this, args);
      inThrottle = true;
      setTimeout(() => (inThrottle = false), limit);
    }
  };
}
```

**⚠️ Ehtiyot bo'l:** Debounce'ga `leading`/`trailing` opsiyasi qo'shilishi mumkin (birinchi chaqiruvda yoki oxirgisida ishga tushish). Sodda throttle trailing chaqiruvni yo'qotadi; ishlab chiqarish versiyalari (lodash) uni saqlaydi. Ikkalasi ham `timer`/`inThrottle` ustidagi closure'ga tayanadi.

---

## Destructuring, spread va rest

**💡 Tushuncha:** **Destructuring** massiv/obyektni o'zgaruvchilarga ochadi. **Spread** (`...`) iterable/obyektni elementlarga yoyadi. **Rest** (`...`) qolgan elementlarni massiv/obyektga yig'adi.

### ❓ Default va renaming bilan destructuring'ni ko'rsating.

**✅ Javob:**

```js
const { a = 1, b: bee, ...others } = { a: 10, b: 20, c: 30 };
// a => 10, bee => 20, others => { c: 30 }

const [first, , third = 99] = [1, 2];   // first=1, third=99 (default; index 1 tashlandi)

const merged = { ...obj1, ...obj2 };     // shallow merge (keyingisi g'olib)
const copy = [...arr];                    // shallow array nusxasi
function sum(...nums) { return nums.reduce((a, b) => a + b, 0); } // rest param
```

**⚠️ Ehtiyot bo'l:** Spread — **shallow** nusxa; ichki obyektlar hali ham havola bo'yicha ulashiladi. Va object spread faqat own enumerable xususiyatlarni nusxalaydi (prototype method'larni emas).

---

## Modullar: ESM vs CommonJS

**💡 Tushuncha:** **CommonJS** (`require`/`module.exports`) — Node'ning eski sinxron modul tizimi. **ES Modules** (`import`/`export`) — standart, statik tahlil qilinadigan, asinxron va tree-shakeable.

### ❓ ESM va CommonJS'ni solishtiring.

**✅ Javob:**
- **Sintaksis:** ESM `import`/`export`; CJS `require`/`module.exports`.
- **Yuklash:** ESM statik (import'lar hoist qilinib, ijrodan oldin hal qilinadi) — tree-shaking'ga imkon beradi; CJS dinamik va sinxron (`require` chaqiruv vaqtida ishlaydi).
- **Binding:** ESM import'lari **jonli, read-only binding**; CJS require vaqtidagi qiymat nusxasini export qiladi.
- **`this`:** ESM'da top-level `this` — `undefined`, CJS'da `module.exports`.
- ESM top-level `await` ni qo'llab-quvvatlaydi.

```js
// ESM
export const x = 1;
export default fn;
import fn, { x } from "./mod.js";

// CommonJS
module.exports = { x: 1 };
const { x } = require("./mod");
```

**⚠️ Ehtiyot bo'l:** ESM modulni to'g'ridan-to'g'ri `require()` qilib bo'lmaydi, va ESM'ning named import'lari statik hal qilinishi shart (shartli/lazy yuklash uchun dinamik `import()` ishlating).

---

## Map / Set / WeakMap

**💡 Tushuncha:** **`Map`** — key→value, **istalgan** key tipi bilan, qo'shish tartibini saqlaydi. **`Set`** — noyob qiymatlarni saqlaydi. **`WeakMap`/`WeakSet`** — key'larni zaif ushlaydi (faqat obyekt), shuning uchun garbage collection'ga to'sqinlik qilmaydi.

### ❓ Qachon oddiy obyekt o'rniga `Map`? `WeakMap` nimaga kerak?

**✅ Javob:** `Map` ni — key string/symbol bo'lmaganda, tartib kafolati kerakda, tez-tez qo'shish/o'chirishda yoki ishonchli `.size` kerakda ishlating. Oddiy obyektlar prototype key'larini ham olib yuradi (pollution xavfi). `WeakMap` ni — obyektlarga maxfiy/yordamchi ma'lumotni memory leak qilmasdan bog'lash uchun ishlating: key obyekt GC qilinganda yozuv yo'qoladi. `WeakMap` key'lari obyekt bo'lishi shart va enumerable emas.

```js
const m = new Map([["a", 1]]);
m.set(obj, "v").get(obj); // istalgan key tipi, tartibli, m.size

const cache = new WeakMap();        // key GC qilinganda yozuv avto-tozalanadi
cache.set(domNode, computedData);   // tugun o'chsa leak yo'q
```

**⚠️ Ehtiyot bo'l:** `WeakMap` iterable emas va `.size` i yo'q — uning ichidagilar ataylab kuzatib bo'lmaydigan qilingan (chunki GC vaqti nondeterministik).

---

## Deep vs shallow copy

**💡 Tushuncha:** **Shallow** nusxa faqat yuqori darajani dublikat qiladi, ichki havolalarni ulashadi. **Deep** nusxa hamma narsani rekursiv dublikat qiladi, to'liq mustaqil.

### ❓ Obyektni qanday deep clone qilasiz?

**✅ Javob:** Variantlar:
- `structuredClone(obj)` — o'rnatilgan, Date, Map, Set, siklik havolalarni boshqaradi (lekin funksiyalarni emas).
- `JSON.parse(JSON.stringify(obj))` — sodda, lekin funksiya, `undefined`, `Symbol` ni tashlaydi, `Date` ni buzadi (→ string), va sikllarda throw qiladi.
- To'liq nazorat uchun qo'lda rekursiv clone.

```js
const deep = structuredClone(original); // ✅ zamonaviy, ishonchli

function deepClone(value, seen = new WeakMap()) {
  if (value === null || typeof value !== "object") return value;
  if (seen.has(value)) return seen.get(value);          // sikllarni boshqaradi
  const copy = Array.isArray(value) ? [] : {};
  seen.set(value, copy);
  for (const key of Object.keys(value)) {
    copy[key] = deepClone(value[key], seen);
  }
  return copy;
}
```

**⚠️ Ehtiyot bo'l:** `JSON` cloning ma'lumotni sokin yo'qotadi (funksiya, `undefined`, `Symbol` key'lar), `Date` ni ISO string'ga aylantiradi va siklik havolalarda throw qiladi. Mavjud bo'lsa `structuredClone` afzal.

---

## Event delegation

**💡 Tushuncha:** **Event delegation** bitta listener'ni umumiy ajdod elementga biriktiradi va event **bubbling** hamda `event.target` orqali ko'plab bolalar (shu jumladan dinamik qo'shilganlar) eventlarini boshqaradi.

### ❓ Nega event delegation ishlatamiz?

**✅ Javob:** N ta bolaning har biriga handler bog'lash o'rniga (qimmat xotira, kelajakdagi elementlarni qoplamaydi), parent'ga bitta handler bog'lanadi. Event'lar yuqoriga bubble qiladi; qaysi bola ishga tushirganini topish uchun `event.target` ni tekshiring. Bu masshtablanadi, dinamik qo'shilgan elementlar uchun ishlaydi va tozalashni soddalashtiradi.

```js
document.querySelector("#list").addEventListener("click", (e) => {
  const item = e.target.closest("li");
  if (item) console.log("bosildi", item.dataset.id);
});
```

**⚠️ Ehtiyot bo'l:** Ba'zi event'lar bubble qilmaydi (`focus`, `blur`, `mouseenter`) — delegation uchun ularning bubbling muqobillarini (`focusin`, `mouseover`) ishlating. `event.target` (qayerdan kelib chiqqan) va `event.currentTarget` (listener qayerga bog'langan) ni to'g'ri ishlating.

---

## `bind` / `call` / `apply` implementatsiyasi

**💡 Tushuncha:** `call`/`apply` funksiyani aniq `this` (va argumentlar — `call` alohida, `apply` massiv) bilan chaqiradi. `bind` `this` (va ixtiyoriy oldindan berilgan argumentlar) ga doimiy bog'langan yangi funksiya qaytaradi.

### ❓ `call`, `apply`, `bind` ni amalga oshiring.

**✅ Javob:**

```js
Function.prototype.myCall = function (ctx, ...args) {
  ctx = ctx ?? globalThis;
  const key = Symbol("fn");
  ctx[key] = this;            // fn'ni biriktirib `this` ni ctx qilamiz
  const result = ctx[key](...args);
  delete ctx[key];
  return result;
};

Function.prototype.myApply = function (ctx, args = []) {
  return this.myCall(ctx, ...args);
};

Function.prototype.myBind = function (ctx, ...bound) {
  const fn = this;
  return function (...args) {
    // `new` bilan ishlatishni qo'llab-quvvatlash: new-langan chaqiruv ctx'ni e'tiborsiz qoldiradi
    const isNew = this instanceof fn;
    return fn.apply(isNew ? this : ctx, [...bound, ...args]);
  };
};
```

**⚠️ Ehtiyot bo'l:** To'g'ri `bind` `new` bilan chaqirilishni (bog'langan `this` e'tiborsiz qolishi kerak) va partial application'ni (oldindan berilgan argumentlar oldinga qo'shilishi) boshqarishi shart.

---

## Memoization

**💡 Tushuncha:** **Memoization** pure funksiyaning natijalarini argumentlari bo'yicha kesh qiladi, shuning uchun bir xil input bilan takroriy chaqiruvlar darhol qaytadi.

### ❓ `memoize` ni amalga oshiring.

**✅ Javob:**

```js
function memoize(fn) {
  const cache = new Map();
  return function (...args) {
    const key = JSON.stringify(args);     // sodda key; serializatsiyalanadigan arg'larni taxmin qiladi
    if (cache.has(key)) return cache.get(key);
    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
}

const slowSquare = (n) => n * n;
const fastSquare = memoize(slowSquare);
fastSquare(4); // hisoblaydi => 16
fastSquare(4); // keshdan => 16
```

**⚠️ Ehtiyot bo'l:** `JSON.stringify` key'lar funksiya, serializatsiyalanmaydigan arg yoki key-tartibi farqida ishlamaydi. Bitta obyekt argument uchun `WeakMap` kesh leak'dan qochadi. Memoization faqat **pure** funksiyalar uchun ishlaydi — impure funksiyani keshlash eskirgan natija qaytaradi.

---

## Immutability va pure functions

**💡 Tushuncha:** **Pure function** — deterministik (bir xil input → bir xil output) va side effect'siz. **Immutability** — mavjud ma'lumotni hech qachon mutatsiya qilmaslik; o'rniga yangi nusxalar ishlab chiqarish. Birgalikda ular kodni oldindan aytsa bo'ladigan, test qilinadigan va tushunarli qiladi (va React/Redux asosini tashkil etadi).

### ❓ Funksiyani nima pure qiladi?

**✅ Javob:** Ikki shart: (1) qaytarish qiymati faqat argumentlariga bog'liq (tashqi mutable holatga tayanmaydi), va (2) hech qanday kuzatiladigan side effect yo'q (argument/global mutatsiyasi yo'q, I/O yo'q, DOM o'zgarishi yo'q). Pure funksiyalar keshlanadigan, parallellashtiriladigan va oson test qilinadigan.

```js
// ❌ impure: input'ni mutatsiya qiladi + tashqi holatdan foydalanadi
let total = 0;
function addImpure(arr, x) { arr.push(x); total += x; }

// ✅ pure: yangi ma'lumot qaytaradi, side effect yo'q
function addPure(arr, x) { return [...arr, x]; }
```

**⚠️ Ehtiyot bo'l:** Immutable yangilanishlar spread (`{...obj, key: val}`) yoki yangi massiv qaytaruvchi method'lar (`map`, `filter`, `concat`, `slice`) ishlatadi — **mutatsiya** qiluvchi method'lar emas (`push`, `splice`, `sort`, `reverse`). Esda tuting: `sort` va `reverse` joyida mutatsiya qiladi; avval nusxa oling (`[...arr].sort()`).

---

## Masalalar

> Yechimlar: [`solutions/frontend/03-javascript.md`](../solutions/frontend/03-javascript.md)

1. **`once`** — funksiyani ko'pi bilan bir marta chaqiradigan `once(fn)` yozing; keyingi chaqiruvlar birinchi natijani qaytarsin.
2. **`flatten`** — istalgan chuqurlikdagi massivni tekislaydigan `flatten(arr, depth = Infinity)` ni `reduce` orqali yozing.
3. **`curry`** — `add(1)(2)(3)`, `add(1, 2)(3)`, `add(1)(2)(3)` larning hammasi `6` qaytaradigan umumiy `curry(fn)` yozing.
4. **`debounce`** — `leading` va `trailing` opsiyalarini qo'llab-quvvatlaydigan `debounce(fn, delay, { leading, trailing })` yozing.
5. **`throttle`** — trailing chaqiruvni yo'qotmaydigan (oxirgi chaqiruv interval tugagach ishga tushadigan) `throttle(fn, limit)` yozing.
6. **`deepClone`** — siklik havolalarni (`WeakMap` orqali) va `Date`, `Map`, `Set` ni to'g'ri boshqaradigan `deepClone(value)` yozing.
7. **`Promise.all`** — `[val1, val2, ...]` tartibini saqlaydigan va birinchi rejection'da fail-fast qiladigan `promiseAll(promises)` yozing.
8. **`Promise.allSettled`** — hech qachon reject qilmaydigan va har biri uchun `{status, value|reason}` qaytaradigan `allSettled(promises)` yozing.
9. **`MyPromise`** — `then`/`catch` chaining'ni va microtask orqali asinxron settle'ni qo'llab-quvvatlaydigan minimal `MyPromise` class yozing.
10. **`memoize`** — bitta obyekt argument uchun `WeakMap` ishlatadigan va ko'p argumentli holatni ham boshqaradigan `memoize(fn)` yozing.

---

← [Frontend bo'limiga qaytish](./README.md)
