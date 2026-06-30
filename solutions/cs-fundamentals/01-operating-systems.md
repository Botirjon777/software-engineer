# Operatsion Tizimlar — Masalalar Yechimi

Bu yerda [01-operating-systems.md](../../cs-fundamentals/01-operating-systems.md) faylidagi masalalarning yechimlari keltirilgan.

## Mundarija

- [1-masala: Process vs thread jadval](#1-masala)
- [2-masala: Scheduling hisob-kitobi](#2-masala)
- [3-masala: Bloklovchi Node.js kod](#3-masala)
- [4-masala: Context switch va ko'p thread](#4-masala)
- [5-masala: Blocking vs non-blocking diagramma](#5-masala)
- [6-masala: Page fault va thrashing](#6-masala)
- [7-masala: Node.js CPU-bound mexanizmlar](#7-masala)
- [8-masala: Concurrency parallelism'ni talab qiladimi](#8-masala)

---

## 1-masala

**Process vs thread — 5 ta farq:**

| Farq | Process | Thread | Qachon afzal |
|------|---------|--------|--------------|
| Xotira | Alohida address space | Shared memory | Izolatsiya kerak bo'lsa process; tez aloqa kerak bo'lsa thread |
| Yaratish narxi | Qimmat | Arzon | Ko'p, qisqa muddatli ish uchun thread afzal |
| Aloqa | IPC (pipe, socket) | To'g'ridan-to'g'ri xotira | Tez ma'lumot almashinuvi kerak bo'lsa thread |
| Isolation | Yuqori | Past | Xavfsizlik kritik bo'lsa process (masalan, brauzer tab'lari) |
| Crash ta'siri | Faqat o'zini qulatadi | Butun process'ni qulatadi | Barqarorlik kerak bo'lsa process |

Misol: brauzer har bir tab uchun alohida process ishlatadi — bir sayt qulasa, butun brauzer qulamasligi uchun (isolation afzalligi). Aksincha, bitta ilovada UI'ni bloklamasdan fayl yuklash uchun thread ishlatiladi (arzon yaratish va shared memory afzalligi).

## 2-masala

Processlar: P1=8, P2=4, P3=2 (bir vaqtda kelgan deb olamiz, tartib P1, P2, P3).

**FCFS** (tartib: P1, P2, P3):
- P1 boshlanish=0, kutish=0
- P2 boshlanish=8, kutish=8
- P3 boshlanish=12, kutish=12
- O'rtacha kutish = (0+8+12)/3 = **20/3 ≈ 6.67**

**SJF** (qisqadan boshlab: P3=2, P2=4, P1=8):
- P3 kutish=0
- P2 kutish=2
- P1 kutish=6
- O'rtacha kutish = (0+2+6)/3 = **8/3 ≈ 2.67**

**Round Robin** (timeslice=3, navbat P1, P2, P3):

```text
t=0:  P1 ishlaydi 3  (qoldi P1=5)        | 0-3
t=3:  P2 ishlaydi 3  (qoldi P2=1)        | 3-6
t=6:  P3 ishlaydi 2  (P3 tugadi, t=8)    | 6-8
t=8:  P1 ishlaydi 3  (qoldi P1=2)        | 8-11
t=11: P2 ishlaydi 1  (P2 tugadi, t=12)   | 11-12
t=12: P1 ishlaydi 2  (P1 tugadi, t=14)   | 12-14
```

Kutish vaqti = tugash vaqti − kelish − burst:
- P1: tugadi=14, burst=8 → kutish = 14−0−8 = 6
- P2: tugadi=12, burst=4 → kutish = 12−0−4 = 8
- P3: tugadi=8, burst=2 → kutish = 8−0−2 = 6
- O'rtacha kutish = (6+8+6)/3 = **20/3 ≈ 6.67**

Xulosa: SJF eng kichik o'rtacha kutish beradi (bu kutilgan, chunki SJF optimal), lekin u ish uzunligini oldindan bilishni talab qiladi.

## 3-masala

**Muammo:** `for (let i = 0; i < 5e9; i++)` — bu og'ir sinxron CPU ishi. Node.js'ning event loop'i bitta thread'da ishlaydi. Bu tsikl tugaguncha (bir necha soniya) event loop bloklanadi va server boshqa hech qanday so'rovga javob bera olmaydi.

**Tuzatish — worker_threads ishlatish:**

```js
const { Worker } = require("worker_threads");

app.get("/report", (req, res) => {
  const worker = new Worker(`
    const { parentPort } = require("worker_threads");
    let sum = 0;
    for (let i = 0; i < 5e9; i++) sum += i;
    parentPort.postMessage(sum);
  `, { eval: true });

  worker.on("message", (sum) => res.json({ sum }));
  worker.on("error", (err) => res.status(500).json({ error: err.message }));
});
```

Endi og'ir hisob alohida thread'da ishlaydi va main thread (event loop) erkin qoladi — boshqa so'rovlarga javob bera oladi. Muqobil yondashuvlar: hisobni bo'laklarga bo'lib `setImmediate` orqali navbatga qo'yish, yoki natijani cache'lash.

## 4-masala

**Context switch** — CPU bir thread/process'dan boshqasiga o'tishi. OS joriy thread holatini (registerlar, program counter) PCB'ga saqlaydi, keyingisining holatini yuklaydi.

**"Ko'p thread = tez" nega noto'g'ri:** Context switch sof overhead — bu paytda foydali ish bajarilmaydi. Agar thread'lar soni core'lardan ancha ko'p bo'lsa:
- CPU vaqtining katta qismi switch qilishga ketadi (ishning o'ziga emas).
- Cache doimo "buziladi" (cache invalidation), chunki har thread o'z ma'lumotini cache'ga yuklaydi.
- Eng yomon holatda CPU ishdan ko'ra switch'ga ko'p vaqt sarflaydi (thrashing).

Demak, optimal thread soni odatda core soniga yaqin (I/O-bound ish uchun biroz ko'proq). Cheksiz thread yaratish performance'ni oshirmaydi, balki pasaytiradi.

## 5-masala

```text
BLOCKING I/O:
  Thread: ---read()---[KUTISH: thread band, hech narsa qilolmaydi]---natija---keyingi ish-->
                       (bu vaqt behuda ketadi)

NON-BLOCKING I/O:
  Thread: ---read()--> darhol qaytadi
                 |
                 +--> [boshqa so'rovlar ustida ishlaydi]
                 |
          (event: natija tayyor!) --> callback ishga tushadi --> natija qayta ishlanadi
```

**Nega veb-serverlar uchun non-blocking afzal:** Veb yuklamalar asosan I/O-bound (DB so'rovlari, tarmoq, fayl). Blocking modelda har so'rov uchun alohida thread kerak bo'ladi va minglab thread context switch overhead'i va xotira sarfini keltiradi. Non-blocking modelda bitta (yoki kam) thread minglab ulanishni eplaydi, chunki I/O kutilayotgan paytda thread boshqa so'rovlar ustida ishlaydi — hech narsa behuda kutilmaydi.

## 6-masala

**Page fault** — process RAM'da mavjud bo'lmagan virtual page'ga murojaat qilganda sodir bo'ladigan hodisa. OS uni diskdan (yoki swap'dan) RAM'ga yuklaydi, kerak bo'lsa boshqa page'ni diskka chiqaradi.

**Thrashing:** Agar dastur RAM sig'imidan ko'p xotira (working set) talab qilsa, OS doimo page'larni diskka chiqarib, qayta yuklab turishga majbur bo'ladi. Disk RAM'dan minglab marta sekin bo'lgani uchun:
- Process'lar deyarli doimo page kutib (waiting) turadi.
- CPU foydali ish bajarmaydi — vaqt paging'ga ketadi.
- Tizim "muzlagandek" sekinlashadi.

Yechim: RAM qo'shish, dastur xotira iste'molini kamaytirish, yoki yuklamani kamaytirish.

## 7-masala

Node.js'da CPU-bound ish uchun mexanizmlar:

- **worker_threads** — bitta Node.js process ichida haqiqiy parallel thread'lar. Xotirani `SharedArrayBuffer` orqali (yoki message passing bilan) almashish mumkin. Yaratish arzonroq, bir xil process ichida. CPU-intensive hisoblar uchun ideal.
- **child_process / cluster** — alohida Node.js process'lar. To'liq izolatsiya, alohida xotira. `cluster` HTTP serverni bir nechta process'ga taqsimlab, barcha CPU core'lardan foydalanish uchun. Aloqa IPC orqali (sekinroq).

**Taqqoslash:** worker_threads yengilroq va xotira almashishi mumkin (lekin race condition xavfi bor); child_process/cluster og'irroq, to'liq izolatsiyalangan va biri qulasa boshqasi ishlaydi. CPU hisob uchun worker_threads, mustaqil servislar/izolatsiya uchun cluster yoki child_process.

## 8-masala

**Da'vo noto'g'ri.** Concurrency parallelism'ni talab qilmaydi.

Concurrency — bu vazifalarni boshqarish strukturasi (galma-gal bajarish), parallelism esa ularni haqiqatda bir vaqtda bajarish (ko'p core kerak).

**Misol:** Node.js bitta CPU core'da (parallelism yo'q) ham concurrent ishlaydi — event loop minglab so'rovni galma-gal eplaydi, I/O kutilayotganda boshqa ishga o'tadi. Bu yerda hech qanday parallelism yo'q, lekin concurrency to'liq mavjud.

Aksincha ham to'g'ri emas: parallelism concurrency'siz ham bo'ladimi degan savol nozik — amalda parallel ishlash uchun concurrent struktura kerak, lekin concurrency mustaqil ravishda, bitta core'da yashashi mumkin. Demak, ikkisi alohida tushunchalar: concurrency — struktura, parallelism — bajarilish.

---

← [CS Asoslari bo'limiga qaytish](../../cs-fundamentals/README.md)
