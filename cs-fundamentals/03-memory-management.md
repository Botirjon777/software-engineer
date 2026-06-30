# Memory Management — Dastur Xotirasini Boshqarish

Har bir dastur ishlaganda xotira (RAM) ishlatadi: o'zgaruvchilar, obyektlar, funksiya chaqiruvlari — bularning hammasi qayerdadir saqlanadi. Yaxshi dasturchi xotira qanday tashkil etilganini, qaysi ma'lumot qayerda yashashini va u qanday tozalanishini tushunishi kerak. Bu hujjat dastur xotirasi tuzilishidan boshlab, stack va heap farqi, garbage collection algoritmlari va memory leak sabablarigacha bo'lgan asoslarni o'zbek tilida tushuntiradi.

**💡 Tushuncha:** Memory management — bu dastur xotirani qanday so'rab oladi (`allocate`), ishlatadi va qaytaradi (`free`) jarayonini boshqarishdir. JavaScript'da bu avtomatik (garbage collector), C'da esa qo'lda (`malloc`/`free`).

## Mundarija

- [Dastur xotirasi tuzilishi](#dastur-xotirasi-tuzilishi)
- [Stack vs Heap](#stack-vs-heap)
- [Stack frame va function call](#stack-frame-va-function-call)
- [Pointer vs Reference](#pointer-vs-reference)
- [Value vs Reference (JS)](#value-vs-reference-js)
- [Manual vs Automatic memory](#manual-vs-automatic-memory)
- [Garbage Collection](#garbage-collection)
- [Memory Leak](#memory-leak)
- [Memory profiling](#memory-profiling)
- [Stack overflow vs Out-of-memory](#stack-overflow-vs-out-of-memory)
- [Savol-javoblar](#savol-javoblar)
- [Masalalar](#masalalar)

## Dastur xotirasi tuzilishi

Operatsion tizim har bir jarayonga (`process`) virtual xotira manzil maydonini ajratadi. Bu maydon bir nechta segmentlarga bo'linadi. Har bir segment o'z vazifasiga ega.

```text
Yuqori manzillar (high addresses)
 ┌──────────────────────────────┐
 │           STACK              │  ← funksiya chaqiruvlari, local o'zgaruvchilar
 │   (pastga o'sadi ↓)          │     LIFO, avtomatik tozalanadi
 ├──────────────────────────────┤
 │              │               │
 │              ▼               │
 │     (bo'sh joy)              │
 │              ▲               │
 │              │               │
 ├──────────────────────────────┤
 │           HEAP               │  ← dinamik ajratilgan xotira (malloc, new)
 │   (yuqoriga o'sadi ↑)        │     qo'lda yoki GC bilan tozalanadi
 ├──────────────────────────────┤
 │       BSS segment            │  ← initsializatsiya qilinmagan global/static
 │  (int x;  → 0 bilan to'la)   │
 ├──────────────────────────────┤
 │       DATA segment           │  ← initsializatsiya qilingan global/static
 │  (int x = 42;)               │
 ├──────────────────────────────┤
 │       CODE / TEXT segment    │  ← mashina kodi (read-only)
 └──────────────────────────────┘
Past manzillar (low addresses)
```

**Segmentlar:**

- **Code / Text** — kompilyatsiya qilingan mashina kodi (instruksiyalar). Odatda `read-only` (o'zgartirib bo'lmaydi), bir nechta jarayon o'rtasida ulashilishi mumkin.
- **Data** — qiymat berib initsializatsiya qilingan global va `static` o'zgaruvchilar (`int counter = 10;`).
- **BSS** (Block Started by Symbol) — qiymat berilmagan global/`static` o'zgaruvchilar. Dastur boshlanishida nol bilan to'ldiriladi (`static int x;`).
- **Heap** — dasturchi ish vaqtida (`runtime`) dinamik so'rab oladigan xotira. Pastdan yuqoriga o'sadi.
- **Stack** — funksiya chaqiruvlari va lokal o'zgaruvchilar uchun. Yuqoridan pastga o'sadi.

**💡 Tushuncha:** Stack va heap bir-biriga qarama-qarshi yo'nalishda o'sadi. Ular o'rtasidagi bo'sh joy ikkalasi uchun ham zaxira. Agar ular to'qnashsa — `stack overflow` yoki xotira tugashi yuz beradi.

## Stack vs Heap

Bu ikkalasi runtime'da ma'lumot saqlaydigan asosiy joylar, lekin tubdan farq qiladi.

```text
            STACK                          HEAP
 ┌─────────────────────────┐   ┌─────────────────────────┐
 │ Ajratish: avtomatik     │   │ Ajratish: qo'lda/GC     │
 │ Tezlik: juda tez        │   │ Tezlik: sekinroq        │
 │ Hajm: kichik (~1-8 MB)  │   │ Hajm: katta (GB lar)    │
 │ Tartib: LIFO            │   │ Tartib: tartibsiz       │
 │ Hayoti: scope tugaguncha│   │ Hayoti: free/GC gacha   │
 │ Fragmentatsiya: yo'q    │   │ Fragmentatsiya: bor     │
 └─────────────────────────┘   └─────────────────────────┘
```

| Xususiyat | Stack | Heap |
|-----------|-------|------|
| Qanday ajratiladi | Avtomatik (compiler stack pointer'ni siljitadi) | Aniq so'rov bilan (`malloc`, `new`) |
| Tezlik | Juda tez (faqat pointer siljitish) | Sekinroq (bo'sh blok izlash kerak) |
| Hajm | Cheklangan (odatda 1–8 MB) | Katta (mavjud RAM gacha) |
| Tozalash | Avtomatik (scope tugaganda) | Qo'lda (`free`) yoki GC orqali |
| Qaysi ma'lumot | Lokal o'zgaruvchilar, funksiya argumentlari, return manzillar | Obyektlar, massivlar, hajmi runtime'da aniqlanadigan ma'lumot |

**Qaysi ma'lumot qayerda (JS misolida):**

```js
function example() {
  let n = 42;              // primitive → stack'da qiymat
  let s = "salom";         // primitive (string) → engine optimallashtiradi
  let user = {             // obyekt → heap'da, stack'da faqat reference
    name: "Ali",
    age: 30
  };
  let arr = [1, 2, 3];     // massiv → heap'da, stack'da reference
}
// funksiya tugashi bilan stack'dagi qism tozalanadi
// heap'dagi obyekt esa GC kelguncha turadi
```

**⚠️ Ehtiyot bo'l:** Stack tez, lekin kichik. Katta massivni stack'da yaratmoqchi bo'lsangiz (C'da) yoki juda chuqur rekursiya qilsangiz — stack overflow olasiz. Katta ma'lumot heap'da bo'lishi kerak.

## Stack frame va function call

Har bir funksiya chaqirilganda stack'ga yangi **frame** (stack frame / activation record) qo'shiladi. Bu frame quyidagilarni saqlaydi:

- Funksiya argumentlari
- Lokal o'zgaruvchilar
- Return manzili (funksiya tugagach qayerga qaytish)
- Oldingi frame'ga pointer

```text
function a() { b(); }
function b() { c(); }
function c() { /* shu yerda */ }

a() chaqirilganda stack:

 ┌──────────────┐  ← stack pointer (top)
 │  frame: c()  │     lokal o'zgaruvchilar, return → b
 ├──────────────┤
 │  frame: b()  │     return → a
 ├──────────────┤
 │  frame: a()  │     return → main
 ├──────────────┤
 │  frame: main │
 └──────────────┘

c() tugaydi → uning frame'i "pop" qilinadi (o'chadi)
keyin b(), keyin a() — LIFO tartibida
```

**💡 Tushuncha:** Bu LIFO (Last In, First Out) tuzilishi — oxirgi chaqirilgan funksiya birinchi tugaydi. Aynan shu sabab call stack deyiladi. JS'da xato bo'lganda ko'radigan "stack trace" — aynan shu frame'lar ro'yxati.

## Pointer vs Reference

**Pointer** (ko'rsatkich) — bu boshqa qiymatning xotiradagi **manzilini** saqlaydigan o'zgaruvchi. C/C++'da pointer'ni to'g'ridan-to'g'ri ishlatish mumkin: manzilni olish (`&`), manzil orqali qiymatga kirish (`*`), pointer arifmetikasi.

```text
 o'zgaruvchi x = 42        manzil: 0x1000
 pointer p   = 0x1000      p "x ga ko'rsatadi"

 ┌──────┐         ┌──────┐
 │  p   │────────▶│  x   │
 │0x1000│         │  42  │
 └──────┘         └──────┘
```

**Reference** (havola) — pointer'ning xavfsizroq, sodda ko'rinishi. Foydalanuvchi manzilni ko'rmaydi, lekin obyekt baribir "ko'rsatiladi". C++'da reference alohida tushuncha (`int& r = x;`).

**JS'da reference:** JavaScript'da aniq pointer yo'q. Lekin obyektlar **reference** orqali ishlatiladi. O'zgaruvchi obyektni emas, obyektga ko'rsatuvchi havolani saqlaydi.

```js
let a = { value: 1 };
let b = a;          // b — a bilan BIR XIL obyektga reference
b.value = 99;
console.log(a.value); // 99 — chunki a va b bir obyektni ko'rsatadi
```

**💡 Tushuncha:** Pointer = manzilni o'zingiz boshqarasiz (C). Reference = engine manzilni siz uchun boshqaradi (JS, Java, Python). JS'da "pointer" so'zini ishlatmaymiz, "reference" deymiz.

## Value vs Reference (JS)

Bu JavaScript'dagi eng muhim va eng ko'p xato keltiruvchi mavzulardan biri.

**Primitive'lar** (number, string, boolean, null, undefined, symbol, bigint) — **value** (qiymat) bo'yicha uzatiladi. Ular stack'da saqlanadi (engine optimallashtirishiga bog'liq, lekin kontseptual model shunday).

**Obyektlar** (object, array, function) — **reference** (havola) bo'yicha uzatiladi. Ularning o'zi heap'da, stack'da esa faqat havola turadi.

```text
 let x = 10;            STACK              HEAP
 let y = x;          ┌─────────┐
                     │ x │ 10  │
 let o = {a:1};      │ y │ 10  │      ┌────────────┐
 let p = o;          │ o │ ────┼─────▶│ { a: 1 }   │
                     │ p │ ────┼─────▶│ (bir xil)  │
                     └─────────┘      └────────────┘
```

```js
// PRIMITIVE — value bo'yicha (nusxa olinadi)
let x = 10;
let y = x;     // y ga 10 ning NUSXASI
y = 20;
console.log(x); // 10 — x o'zgarmadi

// OBYEKT — reference bo'yicha (havola nusxalanadi)
let o = { a: 1 };
let p = o;     // p va o bir obyektni ko'rsatadi
p.a = 5;
console.log(o.a); // 5 — ikkalasi bir obyekt!
```

**Funksiyaga uzatishda ham xuddi shunday:**

```js
function changePrimitive(n) { n = 100; }
function changeObject(obj) { obj.x = 100; }

let num = 1;
changePrimitive(num);
console.log(num);     // 1 — o'zgarmadi (value nusxasi)

let data = { x: 1 };
changeObject(data);
console.log(data.x);  // 100 — o'zgardi (bir reference)
```

**⚠️ Ehtiyot bo'l:** `obj = newValue` (qayta tayinlash) reference'ni o'zgartiradi, original obyektga ta'sir qilmaydi. Lekin `obj.prop = newValue` (mutatsiya) original obyektni o'zgartiradi. Bu farqni chalkashtirmang.

## Manual vs Automatic memory

Heap'dagi xotira qachondir qaytarilishi (`free`) kerak, aks holda u to'planib qoladi. Buni boshqarishning ikki yo'li bor.

**Manual (qo'lda) — C/C++:**

```text
malloc(size)  → heap'dan size bayt so'rab olish, pointer qaytaradi
free(ptr)     → o'sha xotirani qaytarish

malloc'ni HAR DOIM free bilan yakunlash kerak.
```

Manual boshqaruv tez va aniq nazorat beradi, lekin xatolar xavfli:

- **Memory leak** — `free` qilishni unutish → xotira to'planadi.
- **Dangling pointer** — `free` qilingan xotiraga murojaat → xato/crash.
- **Double free** — bir xotirani ikki marta `free` qilish → crash.

**Automatic (avtomatik) — JS, Java, Python, Go:**

Engine xotirani avtomatik tozalaydi — bu **garbage collection** (GC). Dasturchi `free` chaqirmaydi; GC qaysi obyekt endi kerak emasligini aniqlab, o'zi tozalaydi.

**💡 Tushuncha:** Automatic boshqaruv xavfsizroq (leak va dangling pointer xatolari kamayadi), lekin tozalash qachon bo'lishini siz nazorat qilmaysiz va GC ozgina performance narxiga ega ("GC pause"). JS, Java, Python, Go — hammasi avtomatik.

## Garbage Collection

**Nega kerak?** Avtomatik tillarda obyektlar heap'da yaratiladi, lekin qachon kerak emasligini bilish qiyin. GC "yetib bo'lmaydigan" (`unreachable`) obyektlarni topib, xotirani qaytaradi.

**Asosiy g'oya — reachability (yetib borish):** Obyekt "tirik" hisoblanadi, agar unga root'lardan (global obyektlar, joriy stack o'zgaruvchilari) reference zanjiri orqali yetib borish mumkin bo'lsa.

### Algoritm 1: Reference counting

Har bir obyekt nechta reference ega ekanini sanaydi. Hisob 0 ga tushsa — obyekt o'chiriladi.

```text
let a = {};   → obyekt refcount = 1
let b = a;    → refcount = 2
a = null;     → refcount = 1
b = null;     → refcount = 0 → o'chiriladi
```

**Muammo — cycle (tsikl):** Ikki obyekt bir-birini ko'rsatsa, refcount hech qachon 0 ga tushmaydi, garchi ularga tashqaridan yetib bo'lmasa ham:

```text
 let a = {};  let b = {};
 a.ref = b;   // b refcount = 2
 b.ref = a;   // a refcount = 2
 a = null; b = null;

 ┌─────┐      ┌─────┐
 │  a  │─────▶│  b  │      Tashqaridan yetib bo'lmaydi,
 │     │◀─────│     │      lekin refcount > 0 → LEAK!
 └─────┘      └─────┘
```

**⚠️ Ehtiyot bo'l:** Sof reference counting cycle'larni tozalay olmaydi. Shu sabab zamonaviy JS engine'lar (V8) buni ishlatmaydi — ular mark-and-sweep ishlatadi, u cycle muammosidan xoli.

### Algoritm 2: Mark-and-sweep

Ikki bosqichli algoritm. Zamonaviy JS engine'lar asosi.

```text
1. MARK:  root'lardan boshlab, yetib boriladigan
          BARCHA obyektlarni "tirik" deb belgila.
2. SWEEP: belgilanmagan (yetib bo'lmaydigan)
          obyektlarni o'chir, xotirani qaytar.

 root ──▶ A ──▶ B        C ──▶ D
                         (root'dan yetib bo'lmaydi)

 MARK: A, B belgilanadi (tirik)
 SWEEP: C, D o'chiriladi
```

Cycle muammosini hal qiladi: `C ──▶ D ──▶ C` bo'lsa ham, root'dan yetib bo'lmasa — ikkalasi ham o'chiriladi.

### Algoritm 3: Generational GC (V8 misolida)

Kuzatuv (empirik haqiqat): **ko'p obyektlar tez o'ladi** ("generational hypothesis"). V8 heap'ni avlodlarga bo'ladi:

```text
 ┌─────────────────────────────────────────────┐
 │ Young Generation (yangi)                     │
 │  - kichik, tez-tez tozalanadi (Scavenge)     │
 │  - yangi obyektlar shu yerda tug'iladi       │
 │  - omon qolganlar Old'ga "ko'chiriladi"      │
 ├─────────────────────────────────────────────┤
 │ Old Generation (eski)                        │
 │  - katta, kamdan-kam tozalanadi              │
 │  - uzoq yashaydigan obyektlar (Mark-Sweep)   │
 └─────────────────────────────────────────────┘
```

- **Young generation** — Scavenge algoritmi (tez, "copying"). Yangi obyektlar shu yerda. Ko'pi tez o'ladi, qisqa tozalash.
- **Old generation** — omon qolib, "katta yoshga yetgan" obyektlar. Mark-and-sweep + mark-compact bilan kamroq tozalanadi.

**💡 Tushuncha:** V8 ko'p ishni kichik, tez-tez tozalanadigan young avlodda bajaradi. Bu butun heap'ni har safar tekshirishdan tezroq. Old avlod kamdan-kam tozalanadi, chunki u yerdagi obyektlar uzoq yashaydi.

## Memory Leak

**Memory leak** — endi kerak bo'lmagan, lekin GC tozalay olmaydigan (chunki unga hali reference bor) xotira. Vaqt o'tib xotira to'planadi, dastur sekinlashadi yoki crash bo'ladi.

**JS'da asosiy sabablar:**

**1. Tasodifiy global o'zgaruvchilar:**

```js
function leak() {
  globalVar = "men hech qachon o'chmayman"; // let/const yo'q → global!
}
```

**2. Unutilgan timer / interval:**

```js
setInterval(() => {
  // hugeData reference tutib turadi, GC tozalay olmaydi
  doSomething(hugeData);
}, 1000);
// clearInterval chaqirilmasa — abadiy yashaydi
```

**3. Unutilgan event listener:**

```js
element.addEventListener("click", handler);
// element olib tashlansa ham, listener reference tutadi
// removeEventListener qilinmasa — leak
```

**4. Closure orqali ushlab qolish:**

```js
function outer() {
  const huge = new Array(1000000).fill("x");
  return function inner() {
    // huge ishlatilmasa ham, closure uni ushlab turadi
    return "salom";
  };
}
const fn = outer(); // huge xotirada qoladi
```

**5. Detached DOM nodes:**

```js
let detached = document.getElementById("item");
document.body.removeChild(detached.parentNode);
// DOM'dan olib tashlandi, lekin JS o'zgaruvchi reference tutadi
// → DOM tugun xotirada qoladi
```

**⚠️ Ehtiyot bo'l:** Eng ko'p uchraydigan leak'lar — tozalanmagan `setInterval`/`addEventListener`. Komponent yo'q qilinganda (React `useEffect` cleanup, `componentWillUnmount`) doimo timer va listener'larni o'chiring.

## Memory profiling

Memory leak'ni topish uchun xotirani profil qilish kerak.

- **Chrome DevTools — Memory tab:**
  - **Heap snapshot** — ma'lum bir lahzadagi barcha obyektlar surati. Ikki snapshot'ni solishtirib, qaysi obyektlar to'planayotganini ko'rish mumkin.
  - **Allocation timeline** — vaqt bo'yicha xotira ajratilishi grafigi.
- **Performance tab** — xotira o'sish trendini ko'rsatadi. Agar grafik faqat o'sib boraversa (tushmasa) — leak belgisi.
- **Node.js:**
  - `process.memoryUsage()` — `heapUsed`, `heapTotal`, `rss` qiymatlari.
  - `--inspect` flag + Chrome DevTools.
  - `node --heapsnapshot-signal` yoki `heapdump` paketi.

**💡 Tushuncha:** Leak diagnostikasining klassik usuli: bir amalni bir necha marta bajaring (masalan, sahifani ochib-yoping), keyin GC'ni majburan ishga tushiring va xotirani o'lchang. Agar amal tugagach ham xotira asl darajasiga qaytmasa — leak bor.

## Stack overflow vs Out-of-memory

Bu ikki xato turini chalkashtirmaslik kerak — ular boshqa segmentga tegishli.

```text
 STACK OVERFLOW                    OUT OF MEMORY (OOM)
 ┌──────────────────┐             ┌──────────────────┐
 │ Stack to'lib     │             │ Heap to'lib       │
 │ ketdi            │             │ ketdi             │
 │                  │             │                   │
 │ Sabab: juda      │             │ Sabab: juda ko'p  │
 │ chuqur/cheksiz   │             │ obyekt, leak,     │
 │ rekursiya        │             │ ulkan ma'lumot    │
 └──────────────────┘             └──────────────────┘
```

**Stack overflow:** Call stack sig'imidan oshib ketadi. Asosiy sabab — cheksiz yoki juda chuqur rekursiya.

```js
function recurse() {
  return recurse(); // base case yo'q → cheksiz
}
recurse(); // RangeError: Maximum call stack size exceeded
```

**Out of memory (OOM):** Heap to'lib ketadi. Sabab — juda ko'p obyekt yaratish yoki memory leak.

```js
const arr = [];
while (true) {
  arr.push(new Array(1000000)); // heap to'ladi
}
// FATAL ERROR: ... JavaScript heap out of memory
```

**💡 Tushuncha:** Stack overflow = stack segmenti to'ldi (kichik, rekursiyaga bog'liq). OOM = heap segmenti to'ldi (katta, obyektlar/leak'ga bog'liq). Xato xabari ham farq qiladi: `Maximum call stack size exceeded` vs `JavaScript heap out of memory`.

## Savol-javoblar

### ❓ Stack va heap o'rtasidagi asosiy farq nima?

**✅ Javob:** Stack avtomatik boshqariladi (funksiya scope'i tugaganda tozalanadi), juda tez, lekin kichik (~1–8 MB), LIFO tartibda ishlaydi va lokal o'zgaruvchilarni saqlaydi. Heap dinamik (qo'lda yoki GC orqali tozalanadi), sekinroq, lekin katta, va obyektlar/massivlar kabi runtime'da hajmi aniqlanadigan ma'lumotni saqlaydi. JS'da primitive'lar kontseptual stack'da, obyektlar heap'da.

### ❓ Dastur xotirasidagi data va BSS segmentlari farqi nimada?

**✅ Javob:** Ikkalasi ham global va `static` o'zgaruvchilar uchun. **Data** segmenti qiymat berib initsializatsiya qilingan o'zgaruvchilarni saqlaydi (`int x = 42;`). **BSS** esa qiymat berilmaganlarni (`static int x;`) — ular dastur boshida nol bilan to'ldiriladi. BSS faylda joy egallamaydi (faqat hajmi yoziladi), shuning uchun ijro etiluvchi fayl kichikroq bo'ladi.

### ❓ JS'da `let b = a` (a obyekt) qilganda nima bo'ladi?

**✅ Javob:** Obyektning nusxasi olinmaydi — `b` ham `a` ham heap'dagi **bir xil obyektni** ko'rsatadigan reference bo'ladi. `b.x = 5` qilsangiz, `a.x` ham 5 bo'ladi. Bu shallow copy emas, umuman copy emas — bitta obyektga ikkita havola. Haqiqiy nusxa uchun `{...a}` yoki `structuredClone(a)` kerak.

### ❓ Primitive va obyekt funksiyaga qanday uzatiladi?

**✅ Javob:** Primitive **value** (qiymat nusxasi) bo'yicha — funksiya ichida o'zgartirish tashqaridagi original'ga ta'sir qilmaydi. Obyekt **reference** (havola) bo'yicha — funksiya ichida obyekt xususiyatini mutatsiya qilsa (`obj.x = 5`), tashqaridagi original ham o'zgaradi. Lekin parametrning o'zini qayta tayinlash (`obj = {}`) original'ga ta'sir qilmaydi, chunki faqat lokal havola almashadi.

### ❓ Garbage collection nima va nega kerak?

**✅ Javob:** GC — endi ishlatilmayotgan (root'lardan yetib bo'lmaydigan) obyektlarning xotirasini avtomatik qaytaradigan mexanizm. Kerak, chunki avtomatik tillarda (JS, Java) dasturchi `free` chaqirmaydi; GC bo'lmasa heap to'lib, OOM bo'lardi. U memory leak va dangling pointer kabi qo'lda boshqaruv xatolarini kamaytiradi.

### ❓ Reference counting'ning asosiy kamchiligi nima?

**✅ Javob:** **Cycle (tsikl)** muammosi. Agar ikki obyekt bir-birini ko'rsatsa (`a.ref = b; b.ref = a`), ularning refcount'i hech qachon 0 ga tushmaydi — garchi tashqaridan ularga yetib bo'lmasa ham. Natijada xotira tozalanmay qoladi (leak). Shu sabab V8 sof reference counting emas, mark-and-sweep ishlatadi, u cycle'lardan xoli.

### ❓ Mark-and-sweep qanday ishlaydi?

**✅ Javob:** Ikki bosqich. **Mark:** root'lardan (global'lar, stack o'zgaruvchilari) boshlab reference zanjiri orqali yetib boriladigan barcha obyektlar "tirik" deb belgilanadi. **Sweep:** belgilanmagan (yetib bo'lmaydigan) obyektlar o'chiriladi va xotira qaytariladi. Reachability'ga asoslangani uchun cycle muammosini avtomatik hal qiladi.

### ❓ Generational GC nima va V8 nega uni ishlatadi?

**✅ Javob:** "Generational hypothesis" — ko'p obyektlar tez o'ladi. V8 heap'ni **young** (yangi, tez-tez va tez tozalanadi — Scavenge) va **old** (eski, kamdan-kam tozalanadi — mark-sweep) avlodlarga ajratadi. Yangi obyektlar young'da tug'iladi; omon qolganlar old'ga ko'chiriladi. Bu samaraliroq, chunki ko'p qisqa umrli obyektlar kichik young hududda arzon tozalanadi, butun heap'ni har safar skanerlash shart emas.

### ❓ Memory leak nima va JS'da asosiy sabablari?

**✅ Javob:** Kerak emas, lekin GC tozalay olmaydigan (chunki hali reference bor) xotira. Asosiy sabablar: (1) tasodifiy global o'zgaruvchilar (`let` yo'q), (2) tozalanmagan `setInterval`/timer, (3) `removeEventListener` qilinmagan listener'lar, (4) katta ma'lumotni ushlab turuvchi closure, (5) detached DOM tugunlar (DOM'dan olib tashlangan, lekin JS o'zgaruvchi hali ushlab turibdi).

### ❓ Detached DOM node leak'i qanday yuzaga keladi?

**✅ Javob:** DOM tugunni daraxtdan olib tashlasangiz (`removeChild`), lekin JS o'zgaruvchi hali unga reference tutib tursa — tugun va uning bolalari xotirada qoladi, GC tozalay olmaydi. Misol: `let el = document.getElementById("x")` qilib, keyin `el` ni DOM'dan o'chirsangiz, ammo `el` o'zgaruvchisini `null` qilmasangiz. Yechim: ishlatib bo'lgach reference'larni `null` qilish.

### ❓ Memory leak'ni qanday topasiz?

**✅ Javob:** Chrome DevTools Memory tab'da **heap snapshot** olib, amalni bir necha marta bajarib, ikkinchi snapshot'ni solishtiraman — to'planayotgan obyektlarni qidiraman. Performance tab'da xotira grafigi faqat o'sib borsa (GC'dan keyin ham tushmasa) — leak belgisi. Node'da `process.memoryUsage().heapUsed` ni kuzataman yoki heapdump olaman. Klassik test: amalni takrorlab, GC'dan keyin xotira asl darajaga qaytmasa — leak bor.

### ❓ Stack overflow va out-of-memory farqi nimada?

**✅ Javob:** **Stack overflow** — call stack (kichik, ~1 MB) to'lib ketadi, asosan cheksiz yoki juda chuqur rekursiyadan. Xato: `Maximum call stack size exceeded`. **Out of memory** — heap (katta) to'lib ketadi, juda ko'p obyekt yaratish yoki memory leak'dan. Xato: `JavaScript heap out of memory`. Birinchisi stack segmentiga, ikkinchisi heap segmentiga tegishli.

### ❓ Pointer va reference farqi nima? JS'da qaysi biri bor?

**✅ Javob:** Pointer — boshqa qiymatning xotira manzilini saqlaydi va u bilan to'g'ridan-to'g'ri ishlash mumkin (C'da `&`, `*`, pointer arifmetikasi). Reference — pointer'ning xavfsiz, abstraktlashtirilgan ko'rinishi; manzilni engine boshqaradi. JS'da aniq pointer yo'q — obyektlar **reference** orqali ishlatiladi, lekin dasturchi manzilni ko'rmaydi va boshqarmaydi.

### ❓ Manual va automatic memory management qaysi biri xavfsizroq va nega?

**✅ Javob:** Automatic (GC) xavfsizroq, chunki dasturchi `free` chaqirmaydi — bu memory leak (unutilgan `free`), dangling pointer (`free` qilingandan keyin foydalanish) va double free xatolarini yo'qotadi. Lekin GC performance narxiga ega ("GC pause") va tozalash vaqtini nazorat qilmaysiz. Manual (C) tezroq va aniq nazorat beradi, lekin xato xavfi yuqori.

### ❓ Nima uchun katta massivni stack o'rniga heap'da saqlash kerak?

**✅ Javob:** Stack juda kichik (odatda 1–8 MB) va hajmi cheklangan. Katta massiv stack'ga sig'masa — stack overflow yuz beradi. Heap esa katta (mavjud RAM gacha) va dinamik o'sa oladi. JS'da bu avtomatik — obyektlar va massivlar baribir heap'da. C'da esa dasturchi `malloc` orqali aniq heap'da ajratishi kerak.

## Masalalar

> Yechimlar: [Yechimlar fayli](../solutions/cs-fundamentals/03-memory-management.md)

1. **Reference tushunchasi.** Quyidagi kod nima chop etadi va nega? `let a = {n: 1}; let b = a; b.n = 2; let c = b; c = {n: 9}; console.log(a.n, b.n, c.n);`

2. **Value vs reference funksiyada.** `swap(x, y)` funksiyasi ikki sonni almashtirmoqchi (`let t = x; x = y; y = t;`). Nega bu JS'da ishlamaydi va obyektlar bilan qanday qilib ishlatish mumkinligini tushuntiring.

3. **Cycle va GC.** Reference counting va mark-and-sweep algoritmlari quyidagi holatda qanday farq qiladi? `let a = {}; let b = {}; a.ref = b; b.ref = a; a = null; b = null;` Har bir algoritm bu obyektlarni tozalaydimi?

4. **Memory leak topish.** Quyidagi funksiyada memory leak bor. Toping va tuzating:
   ```js
   const cache = {};
   function process(id, data) {
     cache[id] = data;
     setInterval(() => console.log(cache[id]), 1000);
   }
   ```

5. **Closure leak.** Closure qachon memory leak'ga sabab bo'lishini ko'rsatuvchi misol yozing va uni tuzating (closure katta ma'lumotni keraksiz ushlab qoladigan holat).

6. **Stack overflow.** Cheksiz rekursiyaga olib keladigan funksiya yozing, keyin uni base case qo'shib tuzating. Stack overflow xatosi qaysi segmentga tegishli ekanini tushuntiring.

7. **Detached DOM.** Detached DOM node leak'ini keltirib chiqaradigan kod yozing va uni qanday tuzatishni ko'rsating.

8. **Deep clone.** Berilgan ichma-ich obyektni (`{a: 1, b: {c: 2}}`) shunday nusxalang-ki, nusxadagi `b.c` ni o'zgartirsangiz original o'zgarmasin. Shallow copy (`{...obj}`) nega yetarli emasligini tushuntiring.

9. **Segmentlarni aniqlash.** Berilgan C kodidagi har bir o'zgaruvchi qaysi xotira segmentiga (stack/heap/data/bss) tushishini aniqlang: `int g = 5; static int s; int main() { int local = 1; int *p = malloc(4); }`

10. **GC trigger.** Quyidagi koddan keyin `data` obyekti GC tomonidan tozalanadimi? Agar yo'q bo'lsa, qanday qilib uni tozalanadigan qilish mumkin?
    ```js
    let data = { huge: new Array(1e6) };
    window.handler = () => console.log("hi");
    data = null;
    ```

← [CS Asoslari bo'limiga qaytish](./README.md)
