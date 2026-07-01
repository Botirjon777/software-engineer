# Thread'lar va Multicore

Zamonaviy protsessorlar bir vaqtning o'zida bir nechta ishni bajara oladi. "8 core 16 thread", "hyperthreading", "parallel dastur" — bu atamalar sizga tanish bo'lishi mumkin, lekin ular aslida hardware darajasida nimani anglatadi? Nega ko'proq thread doim tez emas? Nega ba'zan parallel kod ketma-ket koddan sekin ishlaydi?

Bu hujjatda thread'larni **hardware darajasidan** boshlab tushuntiramiz: core nima, hardware thread OS thread'dan qanday farq qiladi, cache coherence va false sharing kabi nozik muammolar, Amdahl qonuni va nega parallellashtirishning chegarasi bor. Oxirida JavaScript'ning single-thread modeli shu kontekstda qayerda turishini ko'ramiz.

## Mundarija

- [Core nima?](#core-nima)
- [Multicore protsessor](#multicore-protsessor)
- [Hardware thread vs OS thread](#hardware-thread-vs-os-thread)
- [Hyperthreading / SMT](#hyperthreading--smt)
- ["8 core 16 thread" nimani anglatadi?](#8-core-16-thread-nimani-anglatadi)
- [Concurrency vs Parallelism](#concurrency-vs-parallelism)
- [Context switch hardware'da](#context-switch-hardwareda)
- [CPU scheduling va OS](#cpu-scheduling-va-os)
- [Cache coherence va MESI](#cache-coherence-va-mesi)
- [False sharing](#false-sharing)
- [Amdahl qonuni](#amdahl-qonuni)
- [Nega ko'p thread doim tez emas](#nega-kop-thread-doim-tez-emas)
- [JavaScript: single-thread va worker_threads](#javascript-single-thread-va-worker_threads)
- [Savol-javob](#savol-javob)
- [Masalalar](#masalalar)

## Core nima?

**Physical core** (fizik yadro) — protsessor ichidagi mustaqil hisoblash birligi. Har bir core o'zining ALU (arifmetik-mantiqiy blok), registerlari, va odatda o'z L1/L2 cache'iga ega. U buyruqlarni (instructions) mustaqil bajara oladi.

```text
      +-------------------- CPU --------------------+
      |                                             |
      |  +-- Core 0 --+        +-- Core 1 --+        |
      |  | ALU        |        | ALU        |        |
      |  | Registers  |        | Registers  |        |
      |  | L1 cache   |        | L1 cache   |        |
      |  | L2 cache   |        | L2 cache   |        |
      |  +------------+        +------------+        |
      |          \                 /                 |
      |           +--- L3 cache (shared) ---+        |
      |                     |                        |
      +---------------------|------------------------+
                            v
                        RAM (DRAM)
```

Bitta core sekundiga milliardlab buyruq bajaradi (GHz — milliard tsikl/sek). Lekin bitta core bir vaqtda asosan bitta ish oqimini ilgari suradi.

## Multicore protsessor

**Multicore** protsessor — bitta chip ichida bir nechta physical core. 2000-yillarning o'rtalarida bitta core'ning tezligini oshirish fizik chegaralarga (issiqlik, quvvat) urildi. Yechim: tezlikni oshirish o'rniga **core'lar sonini** oshirish.

**💡 Tushuncha:** Multicore — bu "bitta ishchini tezroq ishlatish" o'rniga "ko'proq ishchi yollash" strategiyasi. Agar ish bo'linadigan bo'lsa (parallellashtiriladigan), 4 core taxminan 4 barobar tezlik berishi mumkin. Agar ish bo'linmasa — qo'shimcha core'lar bekor turadi.

Har core mustaqil bo'lgani sabab, ular **haqiqiy bir vaqtda** (truly simultaneously) ishlay oladi — bu **parallelism**.

## Hardware thread vs OS thread

Bu eng ko'p chalkashtiriladigan joy. "Thread" so'zi ikki xil narsani anglatadi:

| | Hardware thread | Software / OS thread |
|---|---|---|
| Nima | Core bir vaqtda ushlab turadigan buyruq oqimi uchun fizik resurs (register to'plami) | OS boshqaradigan bajarilish oqimi (execution stream) |
| Nechta | Cheklangan (core soni × SMT) | Amalda juda ko'p (minglab) bo'lishi mumkin |
| Kim boshqaradi | Protsessor (hardware) | Operatsion tizim (scheduler) |
| Misol | 16 hardware thread | Dasturda 1000 ta `Thread` yaratish mumkin |

**💡 Tushuncha:** OS minglab software thread yaratishga ruxsat beradi, lekin ular **bir vaqtda** faqat hardware thread'lar soni qadar bajariladi. Qolganlari navbat kutadi. OS scheduler ularni hardware thread'larga vaqt bo'lib (time-slicing) taqsimlaydi — buni **context switch** orqali qiladi.

```text
1000 software thread  --> OS scheduler --> 16 hardware thread
   (dasturchi yaratadi)   (navbatga soladi)   (fizik bajarilish)
```

## Hyperthreading / SMT

**SMT** (Simultaneous Multithreading) — Intel'da **Hyperthreading** deb ataladi. Bu bitta physical core'ga **ikkita hardware thread**ni bir vaqtda ko'rsatish texnologiyasi.

**Nega bu ishlaydi?** Bitta thread bajarilayotganda core'ning ba'zi qismlari ko'pincha bo'sh turadi — masalan, thread cache miss tufayli RAM'dan ma'lumot kutayotganda ALU bekor. SMT shu bo'sh vaqtda ikkinchi thread'ni ishga soladi.

```text
Bitta core, SMT'siz (bitta thread):
Thread A:  [hisob][KUT RAM.....][hisob][KUT.....]
                   ^ ALU bekor           ^ ALU bekor

Bitta core, SMT bilan (2 thread):
Thread A:  [hisob][KUT RAM.....][hisob]
Thread B:  [.....][hisob       ][KUT..][hisob]
                   ^ A kutganda B ishlaydi -> core to'laroq band
```

**💡 Tushuncha:** SMT ikkita **to'liq** core bermaydi. Ikkala thread bir xil ALU, cache va boshqa resurslarni **baham ko'radi**. Odatda SMT ishga ~20-30% qo'shimcha unumdorlik qo'shadi (workload'ga bog'liq), 100% emas. Ba'zi hisoblash-intensiv (compute-bound) ishlarda esa umuman foyda bermasligi mumkin.

## "8 core 16 thread" nimani anglatadi?

Endi bu iborani ochib beramiz:

- **8 core** — 8 ta physical core (mustaqil hisoblash birligi).
- **16 thread** — 16 ta hardware thread. Ya'ni har core SMT bilan 2 thread ko'rsatadi (8 × 2 = 16).

```text
CPU "8 core / 16 thread":

Core0[T0 T1]  Core1[T2 T3]  Core2[T4 T5]  Core3[T6 T7]
Core4[T8 T9]  Core5[T10 T11] Core6[T12 T13] Core7[T14 T15]

8 fizik core, har biri 2 ta hardware thread = 16 logical processor
```

**⚠️ Ehtiyot bo'l:** "16 thread" degani "16 barobar tez" degani EMAS. Faqat 8 ta haqiqiy core bor. Qolgan 8 "thread" — SMT orqali qo'shimcha, ular resurslarni baham ko'radi. Agar ishingiz sof CPU-bound bo'lsa, 8 core'dan oshgan foydangiz cheklangan bo'ladi.

## Concurrency vs Parallelism

Hardware nuqtai nazaridan bu ikki tushuncha farq qiladi:

**Concurrency (bir vaqtdalik / bir-birlashuvchanlik):** Bir necha ish "bir vaqtda ilgari suriladi", lekin ular fizik jihatdan bir vaqtda bajarilmasligi mumkin. Bitta core ham bir necha ishni navbatma-navbat tez almashtirib (context switch) concurrent ishlata oladi.

**Parallelism (parallellik):** Bir necha ish **haqiqatan ham bir vaqtda** bajariladi — buning uchun bir nechta core (yoki hardware thread) kerak.

```text
Concurrency (1 core, navbatlash):
Core:  [A][B][A][B][A][B]   <- almashib turadi, bir vaqtda emas

Parallelism (2 core, haqiqiy bir vaqtda):
Core0: [A][A][A][A][A][A]
Core1: [B][B][B][B][B][B]   <- ikkalasi bir paytda
```

**💡 Tushuncha:** Concurrency — bu **tuzilma** (bir necha ishni boshqarish usuli); parallelism — bu **bajarilish** (bir vaqtda fizik ishlash). Parallelism uchun hardware kerak; concurrency bitta core'da ham bo'ladi. Ko'p tizimlar ikkalasini birga ishlatadi.

## Context switch hardware'da

OS bir thread'ni to'xtatib, boshqasini ishga solganda **context switch** sodir bo'ladi. Hardware darajasida bu — core'ning holatini almashtirish.

```text
Thread A ishlayapti           Context switch          Thread B ishlaydi
+-----------------+                                   +-----------------+
| A registerlari  | ---SAQLA---> [A holati xotirada]  | B registerlari  |
| A program count | <--TIKLA---- [B holati xotiradan] | B program count  |
| A stack pointer |                                   | B stack pointer |
+-----------------+                                   +-----------------+
```

Nima saqlanadi/tiklanadi: barcha registerlar, program counter (keyingi buyruq manzili), stack pointer, va holat flag'lari.

**⚠️ Ehtiyot bo'l:** Context switch **bepul emas**. Register holatini saqlash/tiklashdan tashqari, yangi thread'ning cache'i "sovuq" (cold) bo'lishi mumkin — eski thread cache'ni to'ldirgan edi, yangisi qaytadan cache miss'lardan boshlaydi. TLB ham yangilanishi kerak. Juda ko'p context switch (over-subscription) — performance yo'qotishning yashirin sababidir.

## CPU scheduling va OS

Context switch'ni qachon va qanday qilishni **OS scheduler** hal qiladi. U:

- Qaysi thread qaysi core'da ishlashini tanlaydi.
- Har thread'ga qancha vaqt (time slice / quantum) berishni belgilaydi.
- Prioritetlarni hisobga oladi.
- Iloji bo'lsa thread'ni **o'sha core'da** ushlab turadi (CPU affinity) — cache issiq qolishi uchun.

**💡 Tushuncha:** Hardware faqat "resurs" beradi (core'lar, hardware thread'lar). Bu resurslarni minglab software thread o'rtasida qanday **taqsimlash** esa OS scheduler'ning ishi. Shuning uchun thread'lar unumdorligini tushunish uchun hardware va OS'ni birga ko'rish kerak.

## Cache coherence va MESI

Muammo: har core'ning o'z L1/L2 cache'i bor. Agar ikki core bir xil xotira manzilini o'z cache'iga olib, biri o'zgartirsa — ikkinchisi eski (stale) qiymatni ko'radi. Bu falokat.

**Cache coherence** — barcha core'lar umumiy xotiraga nisbatan izchil (consistent) ko'rinishga ega bo'lishini ta'minlaydigan mexanizm. Eng mashhur protokol — **MESI**.

**MESI** — cache line har biri quyidagi to'rt holatdan birida bo'ladi:

| Holat | Nomi | Ma'nosi |
|---|---|---|
| **M** | Modified | Bu core o'zgartirgan, faqat shu cache'da yangi nusxa, RAM eskirgan |
| **E** | Exclusive | Faqat shu cache'da, o'zgartirilmagan, RAM bilan bir xil |
| **S** | Shared | Bir necha cache'da bir xil nusxa, hech kim o'zgartirmagan |
| **I** | Invalid | Bu nusxa yaroqsiz, ishlatib bo'lmaydi |

```text
Core0 x ni o'zgartirdi (M holati)
   |
   v  "invalidate x" signali (bus orqali)
Core1: x -> Invalid (I)   <- endi Core1 qayta o'qishga majbur
```

**💡 Tushuncha:** Bir core cache line'ni o'zgartirsa, protsessor boshqa core'lardagi o'sha line nusxalarini **invalidate** (yaroqsiz) qiladi. Bu avtomatik va shaffof, lekin **tekin emas** — core'lar o'rtasida signal (bus traffic) va sinxronizatsiya kerak.

## False sharing

Cache coherence to'g'ri ishlaydi, lekin u nozik performance muammosi keltirishi mumkin: **false sharing**.

Eslang: cache line 64 bayt. Agar ikki **turli** o'zgaruvchi (masalan, ikki thread'ning alohida hisoblagichlari) tasodifan **bir xil cache line**da yotsa — bir thread o'z o'zgaruvchisini o'zgartirsa, butun line invalidate bo'ladi, va ikkinchi thread o'z o'zgaruvchisi o'zgarmagan bo'lsa ham qaytadan yuklashga majbur.

```text
Cache line (64 bayt):  [ counterA | counterB | ... ]
                          ^Thread0    ^Thread1

Thread0 counterA ni oshiradi -> butun line invalidate
Thread1 counterB (o'zgarmagan!) qaytadan yuklashi kerak
   -> core'lar bir-birini "urib turadi" (ping-pong)
   -> parallel kod ketma-ketdan ham sekin bo'lishi mumkin!
```

**⚠️ Ehtiyot bo'l:** False sharing — parallel kodning eng ayyor performance qotillaridan biri. O'zgaruvchilar mantiqan bog'liq bo'lmasa ham, xotirada yaqin bo'lgani uchun "yolg'ondan" (false) baham ko'rilyapti. Yechim: har thread ma'lumotini alohida cache line'ga joylash (padding) yoki thread-lokal o'zgaruvchilar ishlatish.

## Amdahl qonuni

Parallellashtirishning matematik chegarasi bor. **Amdahl qonuni** buni ifodalaydi: dasturning faqat parallellashtiriladigan qismi tezlashadi; ketma-ket (serial) qism o'zgarmaydi va umumiy tezlashuvni cheklaydi.

```text
Speedup(N) = 1 / ( (1 - P) + P/N )

P = parallellashtiriladigan ulush (0..1)
N = core soni
```

Agar dasturning 95% parallellashtirilsa (P = 0.95):

```text
N = 1:    speedup = 1x
N = 2:    speedup = 1.90x
N = 4:    speedup = 3.48x
N = 8:    speedup = 5.93x
N = 16:   speedup = 9.14x
N = 1000: speedup = 19.6x   <- 1000 core, lekin faqat ~20x!
N -> cheksiz: speedup -> 1/(1-P) = 20x  (chegara)
```

**💡 Tushuncha:** Agar ishingizning 5% ketma-ket bo'lsa, cheksiz core bilan ham 20 barobardan ortiq tezlasholmaysiz. "Serial qism" — bu shishaning bo'g'zi (bottleneck). Shuning uchun parallellashtirishdan oldin serial qismni kamaytirishga harakat qilish ko'pincha muhimroq.

## Nega ko'p thread doim tez emas

Ko'p yangi dasturchi "thread qo'shsam tezlashadi" deb o'ylaydi. Aslida ko'p sabablarga ko'ra tezlashmasligi, hatto sekinlashishi mumkin:

- **Amdahl qonuni** — serial qism chegara qo'yadi.
- **Context switch xarajati** — thread'lar core'dan ko'p bo'lsa, doimiy almashinuv (over-subscription) vaqtni yeydi.
- **Cache coherence / false sharing** — core'lar ma'lumotni baham ko'rsa, sinxronizatsiya sekinlashtiradi.
- **Lock contention** — thread'lar bir resursni qulflab, bir-birini kutadi.
- **Memory bandwidth** — barcha core bitta RAM'ga murojaat qilsa, xotira o'tkazuvchanligi to'yinadi (bottleneck).
- **Sinxronizatsiya murakkabligi** — mutex, atomic operatsiyalar qo'shimcha xarajat.

**💡 Tushuncha:** Parallellashtirish "tekin tushlik" emas. U qo'shimcha xarajat (overhead) keltiradi. Foyda faqat ish yetarlicha katta va yaxshi bo'linadigan bo'lsagina xarajatdan oshadi. Kichik yoki ketma-ket bog'liq ishlarda single-thread yechim ko'pincha yaxshiroq.

## JavaScript: single-thread va worker_threads

JavaScript (Node.js va browser'da) klassik ma'noda **single-threaded** — bir vaqtda bitta asosiy thread (main thread) JS kodini bajaradi. U concurrency'ni **event loop** orqali beradi: I/O ni asinxron kutib, kutish paytida boshqa ishni bajaradi.

```text
JS Event Loop (bitta thread, concurrency, parallelism EMAS):

  [JS kod] -> I/O so'rov (fayl/tarmoq) --> OS'ga topshiriladi
      ^                                          |
      |  (I/O tayyor bo'lgani haqida callback)   |
      +------------------------------------------+
  Kutish paytida boshqa JS ishlarini bajaradi
```

Bu I/O-bound ishlar (server, tarmoq) uchun juda samarali. Lekin **CPU-bound** ish (og'ir hisob-kitob) main thread'ni bloklaydi — hamma narsa muzlaydi.

CPU-bound ishlar uchun Node.js **worker_threads** (browser'da **Web Workers**) beradi — bu haqiqiy OS thread'lar, alohida core'da parallel ishlay oladi.

```text
Main thread (event loop)          Worker thread(lar)
+-------------------+             +-------------------+
| I/O, koordinatsiya| --xabar-->  | og'ir hisob-kitob |
| (bloklanmaydi)    | <--natija-- | (parallel core'da)|
+-------------------+             +-------------------+
   ular xotirani baham ko'rmaydi, xabar (message) almashadi
```

**💡 Tushuncha:** JS'ning single-thread modeli concurrency'ni oson va xavfsiz qiladi (race condition kamroq), lekin CPU parallelism'ni yo'qotadi. `worker_threads` bu bo'shliqni to'ldiradi — ular alohida hardware thread/core'da ishlab, haqiqiy parallelism beradi, ammo ma'lumotni asosan xabar orqali (yoki `SharedArrayBuffer` bilan) almashadi.

**⚠️ Ehtiyot bo'l:** Worker qo'shish har doim yaxshi emas. Worker yaratish va ular bilan ma'lumot almashish (serialization/message passing) xarajat keltiradi. Faqat ish yetarlicha CPU-og'ir bo'lsagina worker foyda beradi — kichik ishlarni main thread'da qoldirish tezroq.

## Savol-javob

### ❓ Physical core nima?

**✅ Javob:** Physical core — protsessor ichidagi mustaqil hisoblash birligi. O'z ALU, registerlari va odatda L1/L2 cache'iga ega. Buyruqlarni mustaqil bajaradi. Multicore protsessorda bir nechta shunday core bir chipda joylashadi.

### ❓ Hardware thread va software (OS) thread o'rtasidagi farq nima?

**✅ Javob:** Hardware thread — core bir vaqtda ushlab turadigan bajarilish uchun fizik resurs (register to'plami), soni cheklangan. Software/OS thread — OS boshqaradigan bajarilish oqimi, minglab bo'lishi mumkin. OS ko'p software thread'ni cheklangan hardware thread'larga time-slicing bilan taqsimlaydi.

### ❓ Hyperthreading (SMT) qanday ishlaydi va nega foyda beradi?

**✅ Javob:** SMT bitta physical core'ga ikkita hardware thread ko'rsatadi. Bitta thread RAM kutayotganda (cache miss) core'ning ba'zi qismlari bo'sh turadi; SMT shu bo'sh vaqtda ikkinchi thread'ni ishga soladi, natijada core to'laroq band bo'ladi. Odatda ~20-30% qo'shimcha unumdorlik beradi, 100% emas.

### ❓ "8 core 16 thread" nimani anglatadi?

**✅ Javob:** 8 physical core, har biri SMT bilan 2 hardware thread ko'rsatadi (8 × 2 = 16 logical processor). Bu 16 barobar tezlik degani emas — faqat 8 haqiqiy core bor, qolgan 8 "thread" SMT orqali, resurslarni baham ko'radi.

### ❓ Concurrency va parallelism farqi nima (hardware nuqtai nazaridan)?

**✅ Javob:** Concurrency — bir necha ishni bir vaqtda ilgari surish (bitta core'da navbatlash bilan ham bo'ladi). Parallelism — bir necha ishni haqiqatan bir vaqtda bajarish, buning uchun bir nechta core/hardware thread kerak. Concurrency — tuzilma, parallelism — bajarilish.

### ❓ Context switch nima va u nima uchun xarajatli?

**✅ Javob:** Context switch — bir thread'dan boshqasiga o'tishda core holatini almashtirish: registerlar, program counter, stack pointer saqlanadi va tiklanadi. Xarajatli, chunki bundan tashqari yangi thread'ning cache'i "sovuq" bo'lishi mumkin (cache miss'lar), TLB yangilanishi kerak. Juda ko'p context switch unumdorlikni pasaytiradi.

### ❓ CPU scheduler nima qiladi?

**✅ Javob:** OS scheduler qaysi thread qaysi core'da, qancha vaqt ishlashini hal qiladi, prioritetlarni hisobga oladi, va iloji bo'lsa thread'ni o'sha core'da ushlaydi (affinity) cache issiq qolishi uchun. Hardware resurs beradi, scheduler uni taqsimlaydi.

### ❓ Cache coherence nima va nega kerak?

**✅ Javob:** Har core'ning o'z cache'i bor; bir core ma'lumotni o'zgartirsa, boshqalar eski qiymatni ko'rmasligi kerak. Cache coherence barcha core'lar umumiy xotiraga izchil ko'rinishga ega bo'lishini ta'minlaydi. Bo'lmasa parallel dasturlar noto'g'ri natija berardi.

### ❓ MESI protokolining to'rt holati nima?

**✅ Javob:** Modified (bu core o'zgartirgan, RAM eskirgan), Exclusive (faqat shu cache'da, o'zgartirilmagan), Shared (bir necha cache'da bir xil nusxa), Invalid (nusxa yaroqsiz). Bir core line'ni o'zgartirsa, boshqalardagi nusxa Invalid bo'ladi.

### ❓ False sharing nima va uni qanday oldini olish mumkin?

**✅ Javob:** False sharing — ikki turli o'zgaruvchi tasodifan bir cache line'da yotib, bir thread birini o'zgartirganda ikkinchi thread'ning o'zgaruvchisi o'zgarmagan bo'lsa ham line invalidate bo'lishi. Bu core'lar o'rtasida "ping-pong" keltiradi va sekinlashtiradi. Yechim: o'zgaruvchilarni alohida cache line'ga joylash (padding) yoki thread-lokal saqlash.

### ❓ Amdahl qonuni nimani aytadi?

**✅ Javob:** Dasturning faqat parallellashtiriladigan qismi tezlashadi; serial (ketma-ket) qism umumiy tezlashuvni cheklaydi. Agar 5% serial bo'lsa, cheksiz core bilan ham eng ko'pi 20x tezlasholasiz. Shuning uchun serial qismni kamaytirish ko'pincha core qo'shishdan muhimroq.

### ❓ Nega ko'p thread doim tez emas?

**✅ Javob:** Amdahl qonuni (serial chegara), context switch xarajati, cache coherence/false sharing, lock contention, memory bandwidth to'yinishi va sinxronizatsiya murakkabligi tufayli. Parallellashtirish overhead keltiradi; foyda faqat ish katta va yaxshi bo'linadigan bo'lsagina xarajatdan oshadi.

### ❓ Over-subscription nima?

**✅ Javob:** Over-subscription — faol thread'lar soni hardware thread'lardan ancha ko'p bo'lishi. Bunda OS doimiy context switch qilishga majbur bo'ladi, bu vaqt va cache issiqligini yo'qotadi. Odatda thread soni core/hardware thread soniga yaqin bo'lgani optimal.

### ❓ Nega JavaScript single-threaded, lekin ko'p ishni "bir vaqtda" qila oladi?

**✅ Javob:** JS main thread'i event loop orqali concurrency beradi: I/O ni asinxron OS'ga topshiradi va kutish paytida boshqa JS kodini bajaradi. Bu parallelism emas (bitta thread) — bu concurrency. I/O-bound ishlar uchun juda samarali, CPU-bound ish esa main thread'ni bloklaydi.

### ❓ worker_threads qachon kerak?

**✅ Javob:** CPU-bound (og'ir hisoblash) ishlar main thread'ni bloklaganda. worker_threads (browser'da Web Workers) haqiqiy OS thread'lar bo'lib, alohida core'da parallel ishlaydi. Ular xotirani asosan xabar (yoki SharedArrayBuffer) orqali almashadi. Kichik ishlarda worker overhead'i foydadan oshadi.

### ❓ SMT haqiqiy core bilan bir xilmi?

**✅ Javob:** Yo'q. SMT (hyperthreading) ikkita thread'ga bitta physical core'ning resurslarini baham ko'rsatadi — ALU, cache umumiy. Ikkita to'liq core emas. Foydasi ish naqshiga bog'liq (odatda ~20-30%), ba'zan sof compute-bound ishda deyarli nol.

### ❓ Lock contention nima?

**✅ Javob:** Lock contention — bir necha thread bir umumiy resursni qulflash (lock) uchun raqobatlashib, biri qulflaganda qolganlari kutishga majbur bo'lishi. Bu parallelizmni yo'qqa chiqaradi — thread'lar navbatga tushadi. Kam qamrovli (fine-grained) lock yoki lock-siz (lock-free) tuzilmalar bilan kamaytiriladi.

## Masalalar

> Yechimlar: [solutions/hardware/04-threads-multicore.md](../solutions/hardware/04-threads-multicore.md)

1. Bir protsessor "6 core 12 thread" deb yozilgan. Nechta physical core bor? SMT ishlatilganmi? Nima uchun "12 thread" 12 barobar tezlikni kafolatlamaydi?

2. Amdahl qonunidan foydalanib, P = 0.90 (90% parallel) bo'lganda 4 core va 8 core uchun speedup'ni hisoblang. Cheksiz core bilan maksimal speedup qancha?

3. Concurrency va parallelism uchun bittadan real hayotdan misol keltiring va nega ular shu turkumga kirishini tushuntiring.

4. Ikki thread har biri o'z hisoblagichini (`long a`, `long b`) tez-tez oshiryapti va bu hisoblagichlar qo'shni xotirada yotibdi. Nega bu kod sekin ishlashi mumkin? Buni qanday tuzatasiz?

5. Bir dasturchi I/O-bound Node.js serverni tezlashtirmoqchi bo'lib, 100 ta worker_threads yaratdi. Nega bu yaxshi fikr bo'lmasligi mumkin? Qachon worker foydali bo'lardi?

6. Context switch paytida hardware darajasida aniq nimalar saqlanadi va tiklanadi? Nega "sovuq cache" qo'shimcha yashirin xarajat keltiradi?

7. MESI protokolida bir core cache line'ni Modified holatiga o'tkazsa, boshqa core'lardagi shu line nusxalariga nima bo'ladi va nega?

8. Sizga to'liq CPU-bound (I/O yo'q) ish berildi, u yaxshi parallellashadi. 8 core / 16 thread mashinada nechta ishchi thread yaratasiz va nega 16 emas, ehtimol 8 ga yaqin son yaxshiroq bo'lishi mumkin?

← [Hardware bo'limiga qaytish](./README.md)
