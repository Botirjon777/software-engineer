# Operatsion Tizimlar

Operatsion tizimlar (OS) — bu kompyuter texnikasi (hardware) bilan ilovalar (applications) o'rtasidagi ko'prik. Backend dasturchi sifatida siz har kuni process, thread, fayl, tarmoq va xotira bilan ishlaysiz — bularning hammasini OS boshqaradi. Intervyularda esa "process bilan thread farqi nima?", "context switch nima?", "Node.js qanday qilib bitta thread bilan minglab so'rovni eplaydi?" kabi savollar tez-tez uchraydi.

**💡 Tushuncha:** OS — bu shunchaki Windows yoki Linux logotipi emas. Bu sizning kodingiz ortidagi "ko'rinmas menejer": qaysi process CPU oladi, xotira qayerdan ajratiladi, diskka yozish qachon bo'ladi — barchasini u hal qiladi. Bu menejerni tushunmasangiz, performance muammolarini hech qachon to'liq tushunmaysiz.

## Mundarija

- [OS nima va vazifalari](#os-nima-va-vazifalari)
- [Kernel va user space](#kernel-va-user-space)
- [System call](#system-call)
- [Process nima](#process-nima)
- [Thread va process vs thread](#thread-va-process-vs-thread)
- [Context switch](#context-switch)
- [CPU scheduling](#cpu-scheduling)
- [Concurrency vs parallelism](#concurrency-vs-parallelism)
- [Virtual memory va paging](#virtual-memory-va-paging)
- [I/O: blocking vs non-blocking](#io-blocking-vs-non-blocking)
- [Node.js single process modeli](#nodejs-single-process-modeli)
- [Savol-javoblar](#savol-javoblar)
- [Masalalar](#masalalar)

---

## OS nima va vazifalari

Operatsion tizim — bu hardware resurslarini boshqaradigan va ilovalar uchun qulay interfeys (abstraksiya) taqdim etadigan dasturiy ta'minot. Sizning kodingiz to'g'ridan-to'g'ri protsessor yoki disk bilan gaplashmaydi — u OS orqali so'raydi.

OS ning asosiy vazifalari:

- **Process management** — processlarni yaratish, to'xtatish, ularni CPU'ga taqsimlash.
- **Memory management** — har bir processga qancha xotira berishni, uni himoyalashni boshqarish.
- **File system** — disk dagi baytlarni fayl va papka abstraksiyasiga aylantirish.
- **Device / I/O management** — disk, tarmoq kartasi, klaviatura kabi qurilmalar bilan ishlash.
- **Security va isolation** — bir process boshqasining xotirasiga kira olmasligini ta'minlash.

```text
+---------------------------------------------+
|   Applications (Node.js, Postgres, browser) |
+---------------------------------------------+
|              Operating System               |
|   (process, memory, file, I/O boshqaruvi)   |
+---------------------------------------------+
|         Hardware (CPU, RAM, disk, NIC)      |
+---------------------------------------------+
```

## Kernel va user space

OS ikki "zona"ga bo'linadi:

- **Kernel space** — OS ning yadrosi (kernel) ishlaydigan privilegiyalangan zona. Bu yerda kodning hardware'ga to'liq ruxsati bor.
- **User space** — sizning ilovalaringiz ishlaydigan cheklangan zona. Bu yerdagi kod hardware'ga to'g'ridan-to'g'ri tegisha olmaydi.

**💡 Tushuncha:** Bu ajratish xavfsizlik uchun. Agar har qanday ilova diskni yoki xotirani to'g'ridan-to'g'ri boshqara olsa, bitta xato butun tizimni qulatardi. Shuning uchun ilovalar "iltimos" qilib so'raydi — system call orqali.

```text
USER SPACE   |  Sizning ilovangiz (Node.js)
             |        |  system call
-------------|--------v---------------------  (chegara)
KERNEL SPACE |  Kernel — hardware'ni boshqaradi
```

## System call

**System call** — bu user space dagi kodning kernel'dan biror amal bajarishni so'rashi uchun interfeys. Misol: faylni o'qish (`read`), tarmoqdan ma'lumot olish (`recv`), process yaratish (`fork`).

Node.js'da `fs.readFile()` chaqirganingizda, ostida `read` system call ishga tushadi. Bu paytda CPU "user mode"dan "kernel mode"ga o'tadi (mode switch), kernel ishni bajaradi va natijani qaytaradi.

```js
const fs = require("fs");
// Bu funksiya ostida kernel'ga 'open' va 'read' system call'lari yuboriladi
fs.readFile("/etc/hosts", "utf8", (err, data) => {
  console.log(data);
});
```

**⚠️ Ehtiyot bo'l:** System call'lar arzon emas — har biri user/kernel mode o'tishini talab qiladi. Shuning uchun yuqori performance'li kod system call sonini kamaytirishga harakat qiladi (masalan, batching).

## Process nima

**Process** — bu ishga tushirilgan dasturning bir nusxasi (instance). Diskdagi dastur — bu shunchaki fayl; uni ishga tushirsangiz, OS uni xotiraga yuklaydi va process yaratadi.

Har bir process'ning o'z **address space**'i bor — bu unga ajratilgan virtual xotira hududi. Bir process boshqasining address space'iga kira olmaydi (isolation).

Process xotirasining tuzilishi:

```text
+------------------+  yuqori manzillar
|      Stack       |  (lokal o'zgaruvchilar, funksiya chaqiriqlari) | pastga o'sadi
|        |         |
|        v         |
|        ^         |
|        |         |
|      Heap        |  (dinamik xotira: new, malloc)  | yuqoriga o'sadi
+------------------+
|  Data (global)   |
+------------------+
|  Text (kod)      |
+------------------+  past manzillar
```

OS har bir process haqida ma'lumotni **PCB** (Process Control Block) da saqlaydi: process ID (PID), holati, register qiymatlari, ochilgan fayllar ro'yxati, xotira manzillari va h.k.

### Process holatlari

```text
   new
    |  (admit)
    v
  ready  <------------------+
    |  (scheduler dispatch) |
    v                       | (interrupt / timeslice tugadi)
 running -------------------+
    |  \
    |   \ (I/O yoki event kutish)
    |    v
    |  waiting
    |    |  (I/O tugadi)
    |    +------> ready
    v
terminated
```

- **new** — process endi yaratilmoqda.
- **ready** — ishlashga tayyor, lekin CPU'ni kutyapti.
- **running** — hozir CPU'da bajarilmoqda.
- **waiting** (blocked) — biror hodisani (I/O, lock) kutyapti.
- **terminated** — tugagan.

## Thread va process vs thread

**Thread** — bu process ichidagi bajarilish oqimi (thread of execution). Bitta process bir nechta thread'ga ega bo'lishi mumkin. Threadlar process'ning xotirasini (heap, global data) **birga ishlatadi**, lekin har birining o'z stack'i va register'lari bor.

```text
PROCESS
+---------------------------------------+
|  Heap, Global data (BIRGA ishlatiladi)|
|                                       |
|  Thread 1     Thread 2     Thread 3   |
|  [stack]      [stack]      [stack]    |
|  [registers]  [registers]  [registers]|
+---------------------------------------+
```

### Process vs Thread taqqoslash

| Xususiyat | Process | Thread |
|-----------|---------|--------|
| Xotira | Alohida address space | Process xotirasini birga ishlatadi |
| Yaratish narxi | Qimmat (sekin) | Arzon (tez) |
| Aloqa (communication) | IPC kerak (pipe, socket) | To'g'ridan-to'g'ri shared memory orqali |
| Isolation | Yuqori (biri qulasa boshqasiga ta'sir qilmaydi) | Past (bir thread qulasa butun process qulaydi) |
| Context switch | Sekin (xotira xaritasi almashadi) | Tez |
| Xavfsizlik | Xato izolatsiyalangan | Race condition xavfi yuqori |

**💡 Tushuncha:** Thread'lar tez va arzon, lekin xotirani birga ishlatgani uchun xavfli (race condition). Process'lar xavfsiz va izolatsiyalangan, lekin og'irroq. Tanlov — performance va xavfsizlik o'rtasidagi murosaga bog'liq.

## Context switch

**Context switch** — bu CPU bir thread/process'dan boshqasiga o'tganida sodir bo'ladigan jarayon. OS joriy thread'ning holatini (registerlar, program counter) PCB'ga saqlaydi, keyingi thread'ning holatini yuklaydi.

```text
Thread A ishlamoqda
    |
    |  interrupt / scheduler qaror qildi
    v
[A ning holati saqlanadi -> A ning PCB]
[B ning holati yuklanadi <- B ning PCB]
    |
    v
Thread B ishlamoqda
```

**⚠️ Ehtiyot bo'l:** Context switch bepul emas — u CPU vaqtini "yutadi" (overhead). Juda ko'p thread bo'lsa, CPU ishning o'zidan ko'ra switch qilishga ko'p vaqt sarflaydi (thrashing). Shuning uchun "minglab thread yaratish" har doim ham yaxshi g'oya emas.

## CPU scheduling

Bir CPU'da bir vaqtning o'zida faqat bitta thread ishlay oladi. **Scheduler** — qaysi ready thread keyingi navbatda CPU oladi degan qarorni qabul qiladigan kernel qismi.

### Preemptive vs non-preemptive

- **Non-preemptive** — process CPU'ni ixtiyoriy ravishda yoki I/O'ga ketganda bo'shatadi. Scheduler uni majburan to'xtata olmaydi.
- **Preemptive** — scheduler timeslice (vaqt bo'lagi) tugaganda process'ni majburan to'xtatib, boshqasiga beradi. Zamonaviy OS'lar shunday ishlaydi.

### Asosiy algoritmlar (qisqacha)

- **FCFS** (First-Come, First-Served) — kim avval keldi, o'sha avval ishlaydi. Oddiy, lekin uzun process qisqalarni bloklaydi (convoy effect).
- **SJF** (Shortest Job First) — eng qisqa ish avval. O'rtacha kutish vaqtini kamaytiradi, lekin ish uzunligini oldindan bilish kerak.
- **Round Robin** — har process navbat bilan teng timeslice oladi. Adolatli, interaktiv tizimlar uchun yaxshi.
- **Priority** — har process'ning prioriteti bor; yuqori prioritetli avval ishlaydi. Xavf: past prioritetli process'lar hech qachon navbat olmasligi mumkin (starvation).

```text
Round Robin (timeslice = 4):
P1[----] P2[----] P3[----] P1[----] P2[--] ...
har biri navbat bilan 4 birlik oladi
```

## Concurrency vs parallelism

Bu ikki tushuncha tez-tez aralashtiriladi.

- **Concurrency** — bir nechta vazifani *boshqarish*, ularni galma-gal bajarib, "bir vaqtda ishlayotgandek" ko'rinishni berish. Bitta CPU'da ham bo'ladi.
- **Parallelism** — bir nechta vazifani *haqiqatda bir vaqtda* bajarish. Buning uchun bir nechta CPU/core kerak.

```text
CONCURRENCY (1 core):    A-B-A-B-A-B  (galma-gal, bittadan)
PARALLELISM (2 core):    A-A-A-A      (haqiqatda bir vaqtda)
                         B-B-B-B
```

**💡 Tushuncha:** Concurrency — bu struktura (vazifalarni qanday tashkil qilish), parallelism — bu bajarilish (qancha core ishlatiladi). Node.js concurrent, lekin standart holatda parallel emas (asosiy ishni bitta thread bajaradi).

## Virtual memory va paging

**Virtual memory** — har bir process'ga go'yo butun xotira o'ziga tegishlidek tuyuladigan abstraksiya. Aslida OS virtual manzillarni fizik manzillarga **paging** orqali xaritalaydi.

Xotira **page** (sahifa, odatda 4KB) larga bo'linadi. Page table virtual page'ni fizik frame'ga bog'laydi. Agar kerakli page RAM'da bo'lmasa (diskda bo'lsa), **page fault** sodir bo'ladi va OS uni diskdan yuklaydi.

```text
Virtual address  ->  [Page Table]  ->  Physical address (RAM)
                                   ->  (yo'q bo'lsa: page fault -> diskdan yukla)
```

**⚠️ Ehtiyot bo'l:** Agar dastur RAM'dan ko'p xotira ishlatib, doimo diskka page'larni yozib-o'qib tursa (swapping/thrashing), performance keskin pasayadi. "Nega serverim sekin?" savolining javobi ba'zan shu.

## I/O: blocking vs non-blocking

- **Blocking I/O** — I/O so'rovi tugaguncha thread "kutadi" (waiting holatiga o'tadi). Shu thread boshqa ish qila olmaydi.
- **Non-blocking I/O** — so'rov darhol qaytadi; natija tayyor bo'lganda xabar beriladi (callback/event). Thread shu vaqt ichida boshqa ishni bajaradi.

```text
BLOCKING:      read() ----[kutish, thread band]---- natija
NON-BLOCKING:  read() -> darhol qaytadi -> (boshqa ish) -> event: tayyor!
```

**💡 Tushuncha:** Node.js'ning "siri" aynan shu — non-blocking I/O. Bitta thread minglab ulanishni eplaydi, chunki I/O kutayotganda u boshqa so'rovlar ustida ishlaydi, behuda kutib turmaydi.

## Node.js single process modeli

Node.js standart holatda **bitta process, bitta asosiy (main) thread** bilan ishlaydi. Sizning JavaScript kodingiz shu yagona thread'da, **event loop** ostida bajariladi.

Lekin Node.js to'liq single-threaded emas:

- **Event loop** — JS kodini va callback'larni navbat bilan bajaradi (concurrency).
- **libuv thread pool** — fayl tizimi, DNS kabi ba'zi blocking operatsiyalar ostida kichik thread pool ishlaydi (standart 4 thread).
- **worker_threads** — CPU-intensive ish uchun haqiqiy parallel thread'lar.
- **cluster / child_process** — bir nechta Node.js process'i (multi-core'dan foydalanish uchun).

```text
Node.js process
+--------------------------------------------------+
|  Main thread: Event Loop (JS kod + callbacklar)  |
|        |                                          |
|        v                                          |
|  libuv thread pool (fs, dns, crypto uchun)        |
|  [t1] [t2] [t3] [t4]                              |
+--------------------------------------------------+
```

**⚠️ Ehtiyot bo'l:** Agar event loop'da og'ir, sinxron CPU ishini bajarsangiz (masalan, katta JSON'ni sync parse qilish yoki cheksiz tsikl), butun server "muzlaydi" — birorta ham so'rovga javob bermaydi. CPU-bound ishni `worker_threads`ga oling.

---

## Savol-javoblar

### ❓ Operatsion tizim asosan nima qiladi?

**✅ Javob:** OS hardware resurslarini (CPU, xotira, disk, I/O) boshqaradi va ilovalar uchun qulay abstraksiya taqdim etadi. Asosiy vazifalari: process management, memory management, file system, I/O/device management va xavfsizlik/isolation. U ilovalar bilan hardware o'rtasidagi vositachi qatlam.

### ❓ Kernel space va user space farqi nimada?

**✅ Javob:** Kernel space — kernel ishlaydigan privilegiyalangan zona, hardware'ga to'liq ruxsati bor. User space — ilovalar ishlaydigan cheklangan zona, hardware'ga to'g'ridan-to'g'ri kira olmaydi. Bu ajratish xavfsizlik va barqarorlik uchun: ilovadagi xato butun tizimni qulatmasligi kerak. Ilovalar kernel'dan amal bajarishni system call orqali so'raydi.

### ❓ System call nima va u oddiy funksiya chaqiruvidan nimasi bilan farq qiladi?

**✅ Javob:** System call — user space kodning kernel'dan privilegiyalangan amal so'rashi (masalan, fayl o'qish, tarmoq, process yaratish). Oddiy funksiya chaqiruvidan farqi: u CPU'ni user mode'dan kernel mode'ga o'tkazadi (mode switch), bu esa overhead beradi. Shuning uchun system call'lar oddiy funksiyalardan qimmatroq.

### ❓ Process nima va uning address space'ida nimalar bor?

**✅ Javob:** Process — ishga tushirilgan dasturning instance'i. Uning address space'i: Text (kod), Data (global o'zgaruvchilar), Heap (dinamik xotira, yuqoriga o'sadi) va Stack (lokal o'zgaruvchilar, funksiya chaqiriqlari, pastga o'sadi). Har process izolatsiyalangan — boshqa process'ning xotirasiga kira olmaydi.

### ❓ PCB nima?

**✅ Javob:** PCB (Process Control Block) — OS har bir process haqida saqlaydigan ma'lumotlar tuzilmasi: PID, joriy holat (ready/running/...), register qiymatlari, program counter, ochilgan fayllar, xotira manzillari. Context switch paytida process holati aynan PCB'ga saqlanadi va undan tiklanadi.

### ❓ Process holatlarini sanab bering.

**✅ Javob:** new (yaratilmoqda), ready (ishga tayyor, CPU kutyapti), running (CPU'da bajarilmoqda), waiting/blocked (I/O yoki hodisa kutyapti), terminated (tugagan). running'dan ready'ga timeslice tugaganda, running'dan waiting'ga I/O kutilganda o'tadi.

### ❓ Process va thread o'rtasidagi asosiy farq nima?

**✅ Javob:** Process'ning o'z izolatsiyalangan address space'i bor; thread'lar esa bitta process xotirasini birga ishlatadi (lekin har birining alohida stack/register'i bor). Thread yaratish va context switch arzonroq/tezroq, lekin shared memory tufayli race condition xavfi bor. Process izolatsiyalangan va xavfsiz, lekin og'irroq va aloqa uchun IPC kerak.

### ❓ Context switch nima va nega u "qimmat"?

**✅ Javob:** Context switch — CPU bir thread/process'dan boshqasiga o'tishi. OS joriy holatni (registerlar, PC) saqlaydi, yangisini yuklaydi. Qimmat, chunki bu paytda foydali ish bajarilmaydi (sof overhead), process switch'da esa xotira xaritasi va cache ham almashadi. Juda ko'p thread bo'lsa, CPU ishdan ko'ra switch'ga vaqt sarflaydi.

### ❓ Preemptive va non-preemptive scheduling farqi nima?

**✅ Javob:** Non-preemptive'da process CPU'ni o'zi (ixtiyoriy yoki I/O'ga ketganda) bo'shatadi. Preemptive'da scheduler timeslice tugaganda process'ni majburan to'xtatadi. Zamonaviy OS'lar preemptive — bu bir process boshqalarni "ushlab" turishining oldini oladi va responsivelikni ta'minlaydi.

### ❓ Round Robin va FCFS farqini tushuntiring.

**✅ Javob:** FCFS — kelish tartibida ishlatadi, non-preemptive; uzun process qisqalarini bloklaydi (convoy effect). Round Robin — har process navbat bilan teng timeslice oladi, preemptive; interaktiv tizimlar uchun adolatli va responsive. RR timeslice juda kichik bo'lsa context switch overhead'i ko'payadi, juda katta bo'lsa FCFS'ga o'xshab qoladi.

### ❓ Concurrency va parallelism farqini bir gapda ayting.

**✅ Javob:** Concurrency — bir nechta vazifani boshqarish/galma-gal bajarib bir vaqtda ishlayotgandek ko'rsatish (1 core'da ham bo'ladi); parallelism — ularni haqiqatda bir vaqtda bajarish (ko'p core kerak). Concurrency — struktura, parallelism — bajarilish. Node.js concurrent, lekin standartda parallel emas.

### ❓ Virtual memory va paging nima uchun kerak?

**✅ Javob:** Virtual memory har process'ga butun xotira o'zinikidek ko'rinadigan abstraksiya beradi va process'larni bir-biridan izolatsiya qiladi. Paging xotirani page'larga bo'lib, virtual manzilni fizik manzilga xaritalaydi. Bu RAM'dan ko'p xotira ishlatish (diskka swap), izolatsiya va xotirani samarali taqsimlash imkonini beradi. Kerakli page RAM'da bo'lmasa page fault sodir bo'ladi.

### ❓ Blocking va non-blocking I/O farqi nima?

**✅ Javob:** Blocking I/O'da thread operatsiya tugaguncha kutadi (waiting holati), shu vaqt boshqa ish qila olmaydi. Non-blocking I/O'da so'rov darhol qaytadi, natija tayyor bo'lganda callback/event orqali xabar beriladi, thread shu vaqt ichida boshqa ish qiladi. Node.js'ning yuqori concurrency'si non-blocking I/O'ga asoslangan.

### ❓ Node.js bitta thread bilan qanday qilib minglab so'rovni eplaydi?

**✅ Javob:** Node.js non-blocking I/O va event loop ishlatadi. I/O kutayotganda (DB, tarmoq) main thread bekorga turmaydi — boshqa so'rovlar ustida ishlaydi; natija tayyor bo'lganda event loop tegishli callback'ni ishga tushiradi. Veb yuklamalar asosan I/O-bound bo'lgani uchun bu model juda samarali. CPU-bound ish esa bu modelni bloklaydi.

### ❓ Nega Node.js'da og'ir CPU ishini main thread'da bajarmaslik kerak?

**✅ Javob:** Chunki butun event loop bitta thread'da ishlaydi. Og'ir sinxron CPU ishi (katta hisob, sync parse) shu thread'ni egallab oladi va event loop boshqa hech qanday callback/so'rovni ishlata olmaydi — server "muzlaydi". Yechim: ishni `worker_threads`ga olish yoki algoritmni bo'laklarga bo'lish.

### ❓ Bir process bir nechta thread'ga ega bo'lsa, biri qulasa nima bo'ladi?

**✅ Javob:** Thread'lar bitta process xotirasini birga ishlatgani uchun, bir thread og'ir xato (segfault, ushlanmagan exception) bilan qulasa, odatda butun process qulaydi. Bu thread'larning kamchiligi — past isolation. Process'larda esa biri qulasa boshqalari ishlashda davom etadi.

---

## Masalalar

> Yechimlar: [solutions/cs-fundamentals/01-operating-systems.md](../solutions/cs-fundamentals/01-operating-systems.md)

1. O'z so'zlaringiz bilan process va thread o'rtasidagi 5 ta farqni jadval ko'rinishida yozing va har biriga qaysi holatda qaysi biri afzalligini bir misol bilan asoslang.

2. FCFS, SJF va Round Robin algoritmlari uchun quyidagi processlar bilan (burst vaqtlari): P1=8, P2=4, P3=2 — har bir algoritm bo'yicha o'rtacha kutish (waiting) vaqtini hisoblang (RR uchun timeslice=3 deb oling).

3. Quyidagi Node.js kod nima uchun serverni bloklaydi va uni qanday tuzatish kerakligini tushuntiring:

```js
app.get("/report", (req, res) => {
  let sum = 0;
  for (let i = 0; i < 5e9; i++) sum += i; // og'ir hisob
  res.json({ sum });
});
```

4. Context switch nima ekanligini va "juda ko'p thread yaratish performance'ni oshiradi" degan da'vo nega har doim ham to'g'ri emasligini tushuntiring.

5. Blocking va non-blocking I/O farqini tasvirlovchi ASCII diagramma chizing va veb-serverlar uchun nega non-blocking afzal ekanligini yozing.

6. Page fault nima? Tizimingiz RAM'dan ko'p xotira ishlatganda performance nega keskin pasayishini (thrashing) tushuntiring.

7. Node.js'da CPU-bound vazifani parallel bajarish uchun qaysi mexanizmlar bor (kamida 2 ta) va ular bir-biridan nimasi bilan farq qiladi? Qisqacha taqqoslang.

8. "Concurrency parallelism'ni talab qiladi" — bu da'vo to'g'rimi? Misol bilan tasdiqlang yoki rad eting.

---

← [CS Asoslari bo'limiga qaytish](./README.md)
