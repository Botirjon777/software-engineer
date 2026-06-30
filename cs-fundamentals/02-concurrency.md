# Concurrency va Parallelism

Concurrency — bu zamonaviy dasturlashning eng chigal, lekin eng muhim mavzularidan biri. Backend dasturchi sifatida siz ko'p foydalanuvchiga bir vaqtda xizmat ko'rsatadigan tizimlar yozasiz va shu yerda race condition, deadlock, lock kabi tushunchalar paydo bo'ladi. Intervyularda "race condition nima?", "deadlock qachon yuz beradi?", "mutex va semaphore farqi?" kabi savollar klassik hisoblanadi.

**💡 Tushuncha:** Concurrency'dagi xatolar eng yomon xatolar — ular tasodifiy yuz beradi, ko'pincha faqat production'da, og'ir yuklama ostida. Bitta sinov da o'tib ketishi mumkin, lekin mingta parallel so'rovda "buziladi". Shuning uchun bu tushunchalarni mustahkam o'zlashtirish kerak.

## Mundarija

- [Concurrency vs parallelism (analogiya)](#concurrency-vs-parallelism-analogiya)
- [Race condition](#race-condition)
- [Critical section va mutual exclusion](#critical-section-va-mutual-exclusion)
- [Synchronization primitivlari](#synchronization-primitivlari)
- [Atomic operatsiya](#atomic-operatsiya)
- [Deadlock](#deadlock)
- [Livelock va starvation](#livelock-va-starvation)
- [Producer-consumer muammosi](#producer-consumer-muammosi)
- [Concurrency modellari](#concurrency-modellari)
- [JavaScript'da concurrency](#javascriptda-concurrency)
- [Thread-safe nima](#thread-safe-nima)
- [Savol-javoblar](#savol-javoblar)
- [Masalalar](#masalalar)

---

## Concurrency vs parallelism (analogiya)

- **Concurrency** — bir nechta vazifani boshqarish, ularni galma-gal bajarib bir vaqtda ishlayotgandek ko'rsatish.
- **Parallelism** — bir nechta vazifani haqiqatda bir vaqtda bajarish (ko'p ishlovchi kerak).

**Analogiya:** Bitta oshpaz (1 core) bir vaqtda osh va salat tayyorlayapti — osh qaynayotganda salat to'g'raydi, keyin oshga qaytadi. Bu **concurrency**: bitta odam, lekin ikkala ishni boshqaryapti. Endi ikkita oshpaz bo'lsa — biri faqat osh, biri faqat salat qiladi, ikkalasi haqiqatda bir vaqtda ishlaydi. Bu **parallelism**.

```text
CONCURRENCY (1 oshpaz):   osh--salat--osh--salat--osh   (galma-gal)
PARALLELISM (2 oshpaz):   osh--osh--osh                 (bir vaqtda)
                          salat--salat--salat
```

**💡 Tushuncha:** Concurrency — bu masalalarni qanday tashkil qilish (struktura), parallelism — bu qancha "qo'l" ishlatish (bajarilish). Concurrent kod parallel bo'lishi shart emas.

## Race condition

**Race condition** — bir nechta thread bir xil ma'lumotga bir vaqtda kirib, natija ularning bajarilish tartibiga bog'liq bo'lib qolishi. "Poyga" — kim oldin yetib boradi, shunga qarab natija o'zgaradi.

Klassik misol — hisoblagichni oshirish. `counter++` aslida 3 ta amal: o'qish, +1, yozish.

```text
Boshlang'ich: counter = 0

Thread A: counter o'qiydi (0)
Thread B: counter o'qiydi (0)     <- ikkalasi 0 ko'rdi!
Thread A: +1 qiladi -> yozadi (1)
Thread B: +1 qiladi -> yozadi (1)  <- 2 bo'lishi kerak edi!

Natija: counter = 1 (XATO, 2 bo'lishi kerak)
```

```js
// Tasavvur qiling: ikki thread bir vaqtda chaqiryapti
let counter = 0;
function increment() {
  const value = counter; // o'qish
  // ... bu yerda boshqa thread aralashishi mumkin
  counter = value + 1;   // yozish
}
```

**⚠️ Ehtiyot bo'l:** Race condition'lar "ba'zan" yuz beradi — bajarilish tartibi tasodifiy. Kod 1000 marta to'g'ri ishlab, 1001-marta buzilishi mumkin. Shuning uchun ularni topish va debug qilish juda qiyin.

## Critical section va mutual exclusion

**Critical section** — kodning shu qismi bo'lib, unda umumiy (shared) resursga murojaat qilinadi va bir vaqtda faqat bitta thread kirishi kerak.

**Mutual exclusion** (o'zaro istisno) — critical section'ga bir vaqtda faqat bitta thread kirishini ta'minlash prinsipi. Yuqoridagi `counter++` ni critical section sifatida himoyalasak, race condition yo'qoladi.

```text
        +-------------------+
Thread A->| CRITICAL SECTION |  <- bir vaqtda faqat 1 ta thread
Thread B->|  (counter++)     |     boshqalari kutadi
Thread C->+-------------------+
```

## Synchronization primitivlari

Critical section'ni himoyalash uchun quyidagi vositalar ishlatiladi.

### Mutex (lock)

**Mutex** (mutual exclusion) — "kalit"ga o'xshaydi. Thread critical section'ga kirishdan oldin lock oladi (`lock`), chiqishda qaytaradi (`unlock`). Lock band bo'lsa, boshqa thread kutadi.

```text
lock();        // kalitni ol
  counter++;   // critical section
unlock();      // kalitni qaytar
```

**Muhim qoida:** kim lock olgan bo'lsa, o'sha unlock qilishi kerak (egalik — ownership).

### Semaphore

**Semaphore** — bu hisoblagichli signal mexanizmi. U bir vaqtda nechta thread resursdan foydalanishi mumkinligini boshqaradi.

- **Binary semaphore** — qiymati 0 yoki 1. Mutex'ga o'xshaydi (lekin egalik tushunchasi yo'q).
- **Counting semaphore** — qiymati N gacha. N ta resurs/slot mavjud bo'lganda ishlatiladi (masalan, connection pool'da 10 ta ulanish).

```text
Counting semaphore (N=3): 3 ta slot bor
  Thread1 oladi -> 2 qoldi
  Thread2 oladi -> 1 qoldi
  Thread3 oladi -> 0 qoldi
  Thread4 -> KUTADI (slot yo'q)
  Thread1 qaytaradi -> 1 qoldi -> Thread4 kiradi
```

**💡 Tushuncha:** Mutex — "bitta kalit, bitta odam". Semaphore — "N ta kalit, N ta odam". Mutex'da egalik bor (kim oldi, o'sha qaytaradi), semaphore'da yo'q (har kim signal bera oladi).

## Atomic operatsiya

**Atomic operation** — bo'linmas (uzilmaydigan) operatsiya: u yo to'liq bajariladi, yo umuman bajarilmaydi, o'rtada boshqa thread aralasha olmaydi. Atomic operatsiyalar lock'siz ham race condition'dan himoya qiladi.

Misol: ko'p protsessorlarda "atomic increment", "compare-and-swap" (CAS) kabi maxsus CPU instruksiyalari bor. Ular `counter++` ni bitta bo'linmas qadamda bajaradi.

```text
Oddiy (3 qadam):    o'qi -> +1 -> yoz   (aralashish mumkin)
Atomic (1 qadam):   [o'qi+1+yoz]        (bo'linmas, xavfsiz)
```

## Deadlock

**Deadlock** (o'lik holat) — ikki yoki undan ortiq thread bir-birini abadiy kutib qolishi. Hech biri davom eta olmaydi.

Klassik misol: A thread Lock1 ni oldi, Lock2 ni kutyapti; B thread Lock2 ni oldi, Lock1 ni kutyapti. Ikkalasi ham boshqasi qaytarishini kutadi — abadiy.

```text
Thread A: Lock1'ni oldi -> Lock2'ni kutadi
                              ^         |
                              |         v
Thread B: Lock2'ni oldi -> Lock1'ni kutadi

(aylana kutish -> abadiy bloklanish)
```

### Coffman shartlari (4 ta)

Deadlock yuz berishi uchun quyidagi 4 sharti bir vaqtda bo'lishi kerak:

1. **Mutual exclusion** — resursni bir vaqtda faqat bitta thread egallaydi.
2. **Hold and wait** — thread bitta resursni ushlab, boshqasini kutadi.
3. **No preemption** — resursni thread'dan majburan tortib olib bo'lmaydi.
4. **Circular wait** — thread'lar aylana shaklida bir-birini kutadi.

**Oldini olish/qochish:** Hech bo'lmasa bitta shartni buzish kerak. Eng amaliy yo'l — **lock'larni har doim bir xil tartibda olish** (circular wait'ni buzadi). Yana: timeout qo'yish, barcha lock'larni bir vaqtda olish (hold and wait'ni buzish).

**⚠️ Ehtiyot bo'l:** Agar har bir thread lock'larni o'zicha har xil tartibda olsa, deadlock vaqt masalasi. Loyihada lock olish tartibini standartlashtiring.

## Livelock va starvation

- **Livelock** — thread'lar bloklanmaydi, lekin doimo bir-biriga "yo'l berib", hech qachon ishni bajarmaydi. (Ikki kishi koridorda bir-biriga yo'l berib, doimo bir tomonga o'tib qolgani kabi.)
- **Starvation** (ochlik) — bir thread doimo resursdan mahrum qoladi, chunki boshqa (yuqori prioritetli) thread'lar uni doimo o'tib ketadi. Priority scheduling'da past prioritetli thread starve bo'lishi mumkin.

**💡 Tushuncha:** Deadlock'da hech kim harakatlanmaydi; livelock'da hamma harakatlanadi, lekin foyda yo'q; starvation'da ba'zilar ishlaydi, lekin bittasi doimo kutib qoladi. Yechim sifatida fairness (adolatlilik) qoidalari qo'llaniladi.

## Producer-consumer muammosi

**Producer-consumer** — klassik concurrency masalasi. Bir yoki bir nechta producer (ishlab chiqaruvchi) ma'lumot yaratib buffer'ga qo'yadi; consumer (iste'molchi) undan oladi. Muammo: buffer to'lganda producer kutishi, bo'sh bo'lganda consumer kutishi kerak — va bularning hammasini xavfsiz (race'siz) qilish.

```text
Producer --> [ buffer: bo'sh slotlar ] --> Consumer
              ^                        ^
              | to'lsa producer kutadi | bo'sh bo'lsa consumer kutadi
```

Yechim odatda counting semaphore'lar bilan quriladi: biri bo'sh slotlarni, biri to'lgan slotlarni hisoblaydi, plus buffer'ni himoyalaydigan mutex.

## Concurrency modellari

Turli tillar concurrency'ni turlicha hal qiladi:

- **Threads (shared memory)** — Java, C++, C#. Thread'lar xotirani birga ishlatadi. Kuchli, lekin lock/race xavfi yuqori, debug qilish qiyin.
- **Event loop** — Node.js, brauzer. Bitta thread, callback/event navbati. Shared memory race'i yo'q, lekin CPU-bound ish bloklaydi.
- **Actor model** — Erlang, Akka. Mustaqil "actor"lar faqat xabar (message) orqali aloqa qiladi, shared memory yo'q. Race'siz, masshtablanadigan.
- **CSP** — Go (goroutine + channel). Thread'lar channel orqali ma'lumot uzatadi ("ma'lumotni almashish orqali aloqa qil, xotirani almashish orqali emas").

```text
Shared memory:  Thread A <--> [umumiy xotira] <--> Thread B   (lock kerak)
Message passing: Actor A --[xabar]--> Actor B                 (lock kerak emas)
```

**💡 Tushuncha:** Asosiy ajralish — shared memory (lock kerak) va message passing (lock kerak emas). Message passing modellari (actor, CSP) ko'pincha xavfsizroq, chunki race condition manbai — umumiy o'zgaruvchi — yo'q.

## JavaScript'da concurrency

JavaScript **single-threaded** — sizning kodingiz bitta thread'da, **event loop** ostida ishlaydi. Bu shuni anglatadi: ikki JS funksiya hech qachon haqiqatda bir vaqtda ishlamaydi, shuning uchun oddiy o'zgaruvchilarda race condition deyarli bo'lmaydi (lekin async oraliqlarda mantiqiy race bo'lishi mumkin).

Parallel ish uchun:

- **Web Workers** (brauzer) / **worker_threads** (Node.js) — alohida thread'lar. Ular standart holatda xotirani almashmaydi, faqat `postMessage` orqali xabar yuboradi (actor-modeliga o'xshaydi).
- **SharedArrayBuffer + Atomics** — worker'lar o'rtasida haqiqiy shared memory. Bu yerda race condition mumkin, shuning uchun `Atomics` API atomic operatsiyalar bilan himoya beradi.

```js
// Async kod ham "mantiqiy race" yaratishi mumkin
let balance = 100;
async function withdraw(amount) {
  const current = balance;        // o'qish
  await saveToDb(current - amount); // await: boshqa kod aralashadi!
  balance = current - amount;     // eski qiymatga asoslanib yozish
}
// Ikki withdraw bir vaqtda chaqirilsa, biri ikkinchisini "yo'qotadi"
```

**⚠️ Ehtiyot bo'l:** "JS single-threaded, demak race yo'q" — to'liq to'g'ri emas. `await` nuqtalarida bajarilish to'xtab, boshqa kod ishlaydi. Umumiy holatni `await`dan keyin yangilash mantiqiy race condition keltirib chiqaradi.

## Thread-safe nima

**Thread-safe** — kod yoki ma'lumot tuzilmasi bir nechta thread tomonidan bir vaqtda ishlatilganda ham to'g'ri ishlashi. Thread-safe kodda critical section'lar to'g'ri himoyalangan, shuning uchun race condition yuz bermaydi.

Kodni thread-safe qilish usullari: lock/mutex ishlatish, atomic operatsiyalar, immutable (o'zgarmas) ma'lumotlardan foydalanish, yoki umuman shared state'dan qochish (message passing).

**💡 Tushuncha:** Eng oson thread-safe kod — shared mutable state'i bo'lmagan kod. "Lock qo'shishdan oldin, umuman umumiy o'zgaruvchi kerakmi?" deb so'rang. Ko'pincha dizaynni o'zgartirish lock'lardan ko'ra yaxshiroq yechim.

---

## Savol-javoblar

### ❓ Concurrency va parallelism farqini analogiya bilan tushuntiring.

**✅ Javob:** Concurrency — bitta oshpaz osh va salatni galma-gal tayyorlashi (bitta resurs, lekin ikkala ish boshqariladi). Parallelism — ikki oshpaz har biri o'z ishini haqiqatda bir vaqtda qilishi (ikki resurs). Concurrency — struktura (vazifalarni boshqarish), parallelism — bajarilish (ko'p core'da haqiqatda bir vaqtda ishlash). Concurrent kod parallel bo'lishi shart emas.

### ❓ Race condition nima va qachon yuz beradi?

**✅ Javob:** Race condition — bir nechta thread umumiy ma'lumotga bir vaqtda kirib, natija bajarilish tartibiga bog'liq bo'lib qolishi. `counter++` kabi "o'qi-o'zgartir-yoz" amali atomic bo'lmagani uchun ikki thread bir xil eski qiymatni o'qib, biri ikkinchisining yozuvini "yo'qotadi". U shared mutable state himoyalanmagan paytda yuz beradi.

### ❓ Critical section nima?

**✅ Javob:** Critical section — kodning umumiy resursga murojaat qiladigan qismi, unga bir vaqtda faqat bitta thread kirishi kerak. Uni mutual exclusion (mutex/lock) bilan himoyalab, race condition'ning oldi olinadi.

### ❓ Mutex va semaphore farqi nimada?

**✅ Javob:** Mutex — bir vaqtda faqat bitta thread'ga ruxsat beradigan "kalit"; egalik (ownership) tushunchasi bor — kim lock olgan, o'sha unlock qiladi. Semaphore — hisoblagichli mexanizm, bir vaqtda N ta thread'ga ruxsat berishi mumkin (counting) yoki 1 ta (binary); egalik yo'q, har qaysi thread signal bera oladi. Mutex — bitta resurs, semaphore — N ta resurs/slot uchun.

### ❓ Binary va counting semaphore farqi?

**✅ Javob:** Binary semaphore qiymati 0 yoki 1 — bitta resursni boshqaradi (mutex'ga o'xshaydi). Counting semaphore qiymati N gacha bo'lib, N ta bir xil resurs/slot mavjud bo'lganda ishlatiladi — masalan, ulanishlar pool'ida 10 ta ulanishni cheklash.

### ❓ Atomic operatsiya nima va u nega muhim?

**✅ Javob:** Atomic operatsiya — bo'linmas, uzilmaydigan operatsiya: yo to'liq bajariladi, yo umuman, o'rtada boshqa thread aralasha olmaydi. CPU darajasidagi atomic increment yoki compare-and-swap kabi instruksiyalar `counter++` ni bitta xavfsiz qadamda bajarib, lock'siz ham race'ning oldini oladi.

### ❓ Deadlock nima va uning 4 sharti qanday?

**✅ Javob:** Deadlock — ikki yoki undan ortiq thread bir-birini abadiy kutib qolishi. Coffman 4 sharti: mutual exclusion (resurs bitta thread'da), hold and wait (ushlab turib boshqasini kutish), no preemption (majburan tortib olib bo'lmaydi), circular wait (aylana kutish). To'rtalasi bir vaqtda bo'lsa deadlock bo'ladi.

### ❓ Deadlock'ning oldini qanday olasiz?

**✅ Javob:** Coffman shartlaridan kamida bittasini buzish kerak. Eng amaliy usul — barcha thread'lar lock'larni har doim bir xil tartibda olishi (circular wait'ni buzadi). Boshqa usullar: lock olishda timeout qo'yish, barcha kerakli lock'larni bir vaqtda olish (hold and wait'ni buzish), yoki shared state'dan qochish.

### ❓ Livelock va starvation o'rtasidagi farq?

**✅ Javob:** Livelock — thread'lar bloklanmaydi, doimo harakatlanadi, lekin bir-biriga "yo'l berib" hech qachon ishni tugatmaydi. Starvation — bir thread boshqalar (yuqori prioritetli) tomonidan doimo o'tib ketilib, resursdan abadiy mahrum qoladi. Deadlock'da hech kim harakatlanmaydi; livelock'da harakat bor, foyda yo'q; starvation'da bittasi doimo kutadi.

### ❓ Producer-consumer muammosi nima?

**✅ Javob:** Producer'lar ma'lumot yaratib umumiy buffer'ga qo'yadi, consumer'lar undan oladi. Muammo: buffer to'lsa producer kutishi, bo'sh bo'lsa consumer kutishi va bularning hammasi race'siz bajarilishi kerak. Odatda ikkita counting semaphore (bo'sh va to'lgan slotlar uchun) plus mutex bilan hal qilinadi.

### ❓ Shared memory va message passing modellari qanday farq qiladi?

**✅ Javob:** Shared memory'da (Java thread'lari) thread'lar umumiy xotirani ishlatadi — kuchli, lekin lock/race xavfi bor. Message passing'da (actor — Erlang, CSP — Go) thread'lar faqat xabar orqali aloqa qiladi, umumiy o'zgaruvchi yo'q — race manbai yo'qolgani uchun xavfsizroq va masshtablanadigan. Go'ning shiori: "xotirani almashish orqali aloqa qilma, aloqa orqali xotirani almash".

### ❓ JavaScript single-threaded bo'lsa, race condition bo'ladimi?

**✅ Javob:** Oddiy sinxron o'zgaruvchilarda yo'q — ikki JS funksiya bir vaqtda ishlamaydi. Lekin `await` nuqtalarida bajarilish to'xtab, boshqa kod ishlaydi, shuning uchun mantiqiy race condition bo'lishi mumkin (masalan, balansni `await`dan oldin o'qib, keyin eski qiymatga asoslanib yozish). Worker'lar SharedArrayBuffer ishlatsa esa haqiqiy race ham mumkin.

### ❓ Node.js'da haqiqiy parallel ish qanday qilinadi?

**✅ Javob:** `worker_threads` yoki (brauzerda) Web Workers orqali — bular alohida thread'lar. Standart holatda ular xotirani almashmaydi, faqat `postMessage` orqali xabar yuboradi (actor modeliga o'xshaydi). Haqiqiy shared memory kerak bo'lsa `SharedArrayBuffer` + `Atomics` ishlatiladi, bu yerda atomic operatsiyalar race'dan himoya beradi.

### ❓ "Thread-safe" degani nima?

**✅ Javob:** Kod yoki ma'lumot tuzilmasi bir nechta thread bir vaqtda ishlatganda ham to'g'ri ishlashi. Bunda critical section'lar lock/atomic bilan himoyalangan yoki shared mutable state umuman yo'q. Eng oson thread-safe kod — immutable ma'lumot yoki shared state'siz dizayn.

### ❓ Lock qo'shish performance'ga qanday ta'sir qiladi?

**✅ Javob:** Lock critical section'ni ketma-ketlashtiradi (serialize) — thread'lar navbat kutadi, parallelism kamayadi. Juda keng (coarse) lock barcha ishni bitta nuqtaga to'plab bottleneck yaratadi; juda nozik (fine-grained) lock esa deadlock va murakkablik xavfini oshiradi. Maqsad — lock'ni iloji boricha qisqa ushlab turish va imkon bo'lsa lock-free (atomic, immutable) yondashuvni tanlash.

### ❓ Optimistic va pessimistic locking farqi nima?

**✅ Javob:** Pessimistic locking — resursni o'zgartirish oldidan lock olib, boshqalarni kutishga majbur qiladi (konflikt ko'p bo'lganda yaxshi). Optimistic locking — lock olmaydi, o'zgartirish paytida konflikt bormi tekshiradi (masalan, version raqami bilan); konflikt bo'lsa qayta uriniladi (konflikt kam bo'lganda samaraliroq). DB'larda optimistic locking version ustuni orqali keng qo'llaniladi.

---

## Masalalar

> Yechimlar: [solutions/cs-fundamentals/02-concurrency.md](../solutions/cs-fundamentals/02-concurrency.md)

1. Quyidagi kod nima uchun race condition'ga moyil ekanligini tushuntiring va uni qanday tuzatish mumkinligini (atomic yoki lock orqali) tasvirlang:

```js
let stock = 1;
function buy() {
  if (stock > 0) {
    stock = stock - 1;
    return "muvaffaqiyat";
  }
  return "tugadi";
}
```

2. Mutex va counting semaphore o'rtasidagi farqlarni jadval ko'rinishida yozing va har biriga bittadan amaliy misol ke
ltiring.

3. Coffman'ning 4 deadlock shartini sanab bering va har bir shartni buzish orqali deadlock'ning oldini olishning bittadan usulini yozing.

4. Ikki thread va ikki lock bilan deadlock yuzaga keladigan ssenariyni ASCII diagramma bilan chizing, so'ng lock tartibini standartlashtirish uni qanday hal qilishini ko'rsating.

5. Quyidagi async JavaScript kodida qanday mantiqiy race condition borligini tushuntiring va tuzatishni taklif qiling:

```js
let balance = 100;
async function withdraw(amount) {
  const current = balance;
  await db.save(current - amount);
  balance = current - amount;
}
```

6. Livelock va starvation o'rtasidagi farqni o'z so'zlaringiz bilan, har biriga real hayotiy analogiya keltirib tushuntiring.

7. Producer-consumer muammosini tasvirlang va uni 2 ta counting semaphore plus 1 ta mutex bilan qanday hal qilish mumkinligini bosqichma-bosqich yozing (kod emas, mantiq).

8. To'rt concurrency modelini (threads/shared memory, event loop, actor, CSP) taqqoslang: har biri uchun qaysi til misol bo'ladi va asosiy afzallik/kamchiligini yozing.

9. "JavaScript single-threaded, demak unda hech qachon concurrency muammolari bo'lmaydi" — bu da'voni baholang. To'g'ri yoki noto'g'ri ekanligini misol bilan asoslang.

---

← [CS Asoslari bo'limiga qaytish](./README.md)
