# Concurrency va Parallelism — Yechimlar

Bu fayl "Concurrency va Parallelism" mavzusidagi masalalar uchun to'liq yechimlarni o'z ichiga oladi.

## 1. `buy()` kodidagi race condition va uni tuzatish

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

**Nima uchun race condition?** Bu yerda "check-then-act" (tekshir, keyin amal qil) naqshi bor. `if (stock > 0)` tekshiruvi va `stock = stock - 1` yozuvi atomic emas — ular ikkita alohida qadam. Ikki thread bir vaqtda kelsa:

```text
stock = 1
Thread A: if (stock > 0) -> rost (1 ko'rdi)
Thread B: if (stock > 0) -> rost (hali ham 1 ko'rdi!)
Thread A: stock = 0, "muvaffaqiyat"
Thread B: stock = -1, "muvaffaqiyat"   <- 1 ta tovar 2 marta sotildi!
```

Natijada bitta tovar ikki kishiga sotiladi va `stock` manfiy bo'lib qoladi (oversell).

**Lock orqali tuzatish** — tekshiruv va kamaytirishni bitta critical section'ga o'rab, atomic qilamiz:

```text
lock();
  if (stock > 0) {
    stock = stock - 1;
    natija = "muvaffaqiyat";
  } else {
    natija = "tugadi";
  }
unlock();
```

Endi A lock'ni ushlab turganda B kutadi, shuning uchun B `stock = 0` ni ko'radi va "tugadi" qaytaradi.

**Atomic orqali tuzatish** — compare-and-swap (CAS) yoki atomic decrement ishlatamiz. Masalan, "agar `stock == 1` bo'lsa, uni `0` ga o'zgartir" ni bitta bo'linmas amalda bajaramiz. CAS muvaffaqiyatsiz bo'lsa (boshqa thread allaqachon o'zgartirgan), "tugadi" qaytaramiz.

**Izoh:** Real database'da bu muammo `UPDATE stock SET qty = qty - 1 WHERE id = ? AND qty > 0` kabi shartli yangilanish bilan hal qilinadi — DB'ning o'zi yozuvni atomic qiladi va `WHERE qty > 0` oversell'ning oldini oladi.

## 2. Mutex va counting semaphore farqi (jadval)

| Xususiyat | Mutex | Counting semaphore |
|-----------|-------|--------------------|
| Bir vaqtda ruxsat | 1 ta thread | N ta thread |
| Egalik (ownership) | Bor — kim lock olsa, o'sha unlock qiladi | Yo'q — har qanday thread signal bera oladi |
| Qiymati | "band" / "bo'sh" (0/1) | 0 dan N gacha hisoblagich |
| Asosiy maqsad | Critical section'ni himoyalash | Cheklangan sondagi resursni boshqarish |
| Amaliy misol | Bitta hisoblagich (`counter++`) ni bir vaqtda faqat bitta thread o'zgartirsin | Connection pool'da 10 ta ulanishni cheklash: 10 ta slot bo'sh, 11-thread kutadi |

**Mutex misoli:** Bank balansini yangilash — bir vaqtda faqat bitta thread `balance` ni o'zgartirishi kerak. Mutex shu kafolatni beradi.

**Counting semaphore misoli:** Bir API'ga bir vaqtda eng ko'pi 5 ta tashqi so'rov yuborish (rate limit). Semaphore'ni 5 bilan boshlaymiz; har so'rov bittasini oladi, tugagach qaytaradi; 6-so'rov slot bo'shaguncha kutadi.

## 3. Coffman'ning 4 sharti va har birini buzish

Deadlock yuz berishi uchun 4 shart bir vaqtda bo'lishi kerak. Kamida bittasini buzsak, deadlock bo'lmaydi:

1. **Mutual exclusion** — resursni bir vaqtda faqat bitta thread egallaydi.
   *Buzish:* Resursni shared (faqat o'qish) yoki lock-free qilish — masalan immutable ma'lumot ishlatish, shunda lock umuman kerak emas.

2. **Hold and wait** — thread bitta resursni ushlab, boshqasini kutadi.
   *Buzish:* Barcha kerakli lock'larni bir vaqtda (atomar) olish; agar hammasini ola olmasa, hech narsani olmasdan qaytib qayta urinish.

3. **No preemption** — resursni majburan tortib olib bo'lmaydi.
   *Buzish:* Lock'ga timeout qo'yish — belgilangan vaqtda ola olmasa, thread o'z lock'larini bo'shatib qayta uradi.

4. **Circular wait** — thread'lar aylana shaklida bir-birini kutadi.
   *Buzish (eng amaliy):* Barcha lock'larni doimo bir xil global tartibda olish (masalan har doim Lock1 dan keyin Lock2). Aylana hosil bo'lmaydi.

**Izoh:** Amalda eng ko'p ishlatiladigani — 4-shartni buzish (lock tartibini standartlashtirish), chunki u oddiy va dasturning qolgan qismini o'zgartirmaydi.

## 4. Deadlock ssenariysi diagramma va yechim

**Deadlock holati** — A va B lock'larni teskari tartibda oladi:

```text
Thread A: Lock1 OLDI ──────► Lock2 ni KUTADI
                                  ▲       │
                                  │       ▼
Thread B: Lock2 OLDI ──────► Lock1 ni KUTADI

A Lock2 ni kutadi (B ushlab turibdi)
B Lock1 ni kutadi (A ushlab turibdi)
=> ikkalasi abadiy kutadi (DEADLOCK)
```

**Lock tartibini standartlashtirish bilan yechim** — ikkala thread ham doimo avval Lock1, keyin Lock2 oladi:

```text
Thread A: Lock1 OLDI -> Lock2 OLDI -> ish -> ikkalasini bo'shatadi
Thread B: Lock1 ni KUTADI (A ushlab turibdi)
          ... A tugagach Lock1 ni oladi -> Lock2 ni oladi -> ish

Endi aylana yo'q: Lock1 — yagona "darvoza", uni kim olsa,
Lock2 ni ham bemalol oladi. Hech kim teskari yo'nalishda kutmaydi.
```

**Izoh:** Asosiy g'oya — circular wait'ni yo'q qilish. Barcha lock'larga global tartib (Lock1 < Lock2) berib, har doim shu tartibda olsak, aylana hosil bo'lishi mantiqan imkonsiz.

## 5. Async withdraw kodidagi mantiqiy race condition

```js
let balance = 100;
async function withdraw(amount) {
  const current = balance;        // o'qish
  await db.save(current - amount); // await: boshqa kod aralashadi!
  balance = current - amount;     // eski qiymatga asoslanib yozish
}
```

**Muammo:** JavaScript single-threaded bo'lsa ham, `await` nuqtasida funksiya to'xtab, event loop boshqa kodni ishga tushiradi. Ikki `withdraw` deyarli bir vaqtda chaqirilsa:

```text
balance = 100
Call1: current = 100
Call1: await db.save(70)   <- to'xtadi, Call2 ishlaydi
Call2: current = 100       <- hali ham 100 o'qidi!
Call2: await db.save(90)
Call1: balance = 70
Call2: balance = 90        <- Call1 ning yechib olishi "yo'qoldi"

Kutilgan: 100 - 30 - 10 = 60
Haqiqiy: 90 (XATO)
```

Bu "lost update" (yo'qolgan yangilanish) — biri ikkinchisining o'zgarishini eski qiymat ustiga yozib yuboradi.

**Tuzatishlar:**

1. **Serializatsiya (mutex/queue):** `withdraw` chaqiruvlarini navbatga qo'yish — bittasi to'liq tugamaguncha ikkinchisi boshlanmaydi. Async mutex (masalan oddiy promise-zanjir) buni ta'minlaydi:

```js
let lock = Promise.resolve();
function withdraw(amount) {
  lock = lock.then(async () => {
    const current = balance;
    await db.save(current - amount);
    balance = current - amount;
  });
  return lock;
}
```

2. **Atomar yangilash (DB darajasida):** Qiymatni JS'da o'qib-yozish o'rniga, database'da atomic shartli yangilanish qilish: `UPDATE accounts SET balance = balance - :amount WHERE balance >= :amount`. Shunda eski qiymatni JS xotirasida ushlab turmaymiz.

**Izoh:** Asosiy xato — holatni (`balance`) `await`dan oldin o'qib, `await`dan keyin eski nusxaga asoslanib yozish. To'g'ri yondashuv: holatni o'zgartirishni bitta uzilmaydigan birlik (mutex yoki atomic DB amali) ichida bajarish.

## 6. Livelock va starvation farqi (analogiya bilan)

**Livelock** — thread'lar bloklanmagan, doimo harakatlanadi, lekin bir-biriga "yo'l berish"ni shu qadar muvofiqlashtiradiki, hech qachon haqiqiy ishni tugatmaydi.

*Analogiya:* Ikki kishi tor koridorda yuzma-yuz keladi. Biri o'ngga o'tadi, ikkinchisi ham o'ngga; keyin ikkalasi chapga; yana o'ngga... Ikkalasi ham harakatda, ikkalasi ham xushmuomala, lekin hech qaysi o'ta olmaydi.

**Starvation (ochlik)** — bir thread doimo resursdan mahrum qoladi, chunki boshqa (ko'pincha yuqori prioritetli) thread'lar uni navbatda doimo o'tib ketadi.

*Analogiya:* Navbatsiz xizmat ko'rsatadigan oshxona — har safar yangi "muhim mehmon" kelaverib, oddiy navbatda turgan odam doimo orqaga suriladi va hech qachon ovqat olmaydi.

**Izoh:** Deadlock'da hech kim qimirlamaydi; livelock'da hamma qimirlaydi, lekin foyda yo'q; starvation'da boshqalar ishlaydi, lekin bittasi doimo kutib qoladi. Starvation'ni adolatli navbat (fairness, FIFO) bilan, livelock'ni esa "yo'l berishga" tasodifiy kechikish (randomized backoff) qo'shib hal qilinadi.

## 7. Producer-consumer: 2 semaphore + 1 mutex bilan yechim (mantiq)

**Muammo:** Producer'lar buffer'ga element qo'shadi, consumer'lar oladi. Buffer to'lsa producer kutishi, bo'sh bo'lsa consumer kutishi, va buffer'ning o'zi race'siz o'zgarishi kerak.

Uchta vosita ishlatamiz:

- `emptySlots` — counting semaphore, boshlang'ich qiymati = N (buffer sig'imi). Nechta bo'sh joy borligini hisoblaydi.
- `filledSlots` — counting semaphore, boshlang'ich qiymati = 0. Nechta to'lgan element borligini hisoblaydi.
- `mutex` — buffer'ga (qo'shish/olish) bir vaqtda faqat bitta thread tegishini ta'minlaydi.

**Producer mantig'i (bosqichma-bosqich):**

1. `emptySlots` dan birini ol — agar 0 bo'lsa (buffer to'la), kut.
2. `mutex` ol — buffer'ni himoyala.
3. Elementni buffer'ga qo'sh.
4. `mutex` ni qaytar.
5. `filledSlots` ga signal ber (+1) — consumer'larga "yangi element bor" deb bildir.

**Consumer mantig'i (bosqichma-bosqich):**

1. `filledSlots` dan birini ol — agar 0 bo'lsa (buffer bo'sh), kut.
2. `mutex` ol — buffer'ni himoyala.
3. Elementni buffer'dan ol.
4. `mutex` ni qaytar.
5. `emptySlots` ga signal ber (+1) — producer'larga "joy bo'shadi" deb bildir.

**Izoh:** Ikki semaphore "kutish/uyg'otish" (to'la va bo'sh holatlar) ni boshqaradi, mutex esa buffer ma'lumot tuzilmasining ichki yaxlitligini saqlaydi. Diqqat: avval semaphore, keyin mutex tartibi muhim — agar avval mutex, keyin semaphore olinsa, deadlock yuzaga kelishi mumkin.

## 8. To'rt concurrency modelini taqqoslash

| Model | Til misoli | Afzallik | Kamchilik |
|-------|-----------|----------|-----------|
| Threads (shared memory) | Java, C++, C# | Kuchli, xotira tez bo'lishadi, CPU-bound ishga mos | Lock/race xavfi yuqori, deadlock, debug qiyin |
| Event loop | Node.js, brauzer JS | Bitta thread, shared-memory race yo'q, I/O-bound ishga zo'r | CPU-bound ish event loop'ni bloklaydi, callback murakkabligi |
| Actor | Erlang, Akka (Scala) | Shared state yo'q (faqat xabar), race'siz, xato izolyatsiyasi yaxshi | Xabar uzatish overhead'i, debug oqimi murakkab |
| CSP | Go (goroutine + channel) | Channel orqali xavfsiz uzatish, yengil goroutine'lar | Channel'larni noto'g'ri ishlatish deadlock berishi mumkin |

**Izoh:** Asosiy ajralish — shared memory (threads, event loop ichidagi shared o'zgaruvchilar) lock talab qiladi; message passing (actor, CSP) esa umumiy o'zgaruvchini yo'q qilib, race manbasini bartaraf etadi. Tanlov ish turiga bog'liq: I/O-bound uchun event loop, izolyatsiya kerak bo'lsa actor, yengil parallelism uchun CSP, maksimal nazorat kerak bo'lsa threads.

## 9. "JavaScript single-threaded, demak concurrency muammosi yo'q" da'vosi

**Bu da'vo noto'g'ri (yarim haqiqat).** To'g'ri tomoni: ikki JS funksiya bitta sinxron oqimda haqiqatda bir vaqtda ishlamaydi, shuning uchun `counter++` kabi oddiy sinxron amalda thread-darajadagi race condition bo'lmaydi.

**Lekin concurrency muammolari baribir bor:**

- **`await` oraliqlaridagi mantiqiy race:** 5-masaladagi `withdraw` misolida ko'rganimizdek, `await` da bajarilish to'xtab, boshqa async kod aralashadi va "lost update" yuzaga keladi.
- **Worker'lardagi haqiqiy race:** `Web Workers`/`worker_threads` + `SharedArrayBuffer` ishlatilsa, bir nechta thread haqiqiy shared memory'ga tegadi — bu yerda klassik race condition mumkin (shuning uchun `Atomics` API mavjud).

Misol (mantiqiy race):

```js
let seats = 1;
async function book() {
  if (seats > 0) {
    await chargeCard();   // bu yerda ikkinchi book() kirib keladi
    seats = seats - 1;    // ikkala chaqiruv ham o'rindiqni "bron qiladi"
  }
}
```

**Izoh:** To'g'ri ifoda — "JavaScript single-threaded bo'lgani uchun sinxron kodda thread race yo'q, lekin async oraliqlarda mantiqiy race condition bor, worker'lar bilan esa haqiqiy race ham mumkin". Single-thread concurrency'ni soddalashtiradi, lekin yo'q qilmaydi.

---

← [CS Asoslari bo'limiga qaytish](../../cs-fundamentals/README.md)
