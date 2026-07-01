# CPU (Protsessor)

CPU — kompyuterning "miyasi": barcha instruksiyalarni bajaradigan va sekundiga milliardlab amal qiladigan komponentning ichki tuzilishini o'rganamiz.

## Mundarija

- [CPU vazifasi](#vazifa)
- [Asosiy qismlar: ALU, Control Unit, Registers](#qismlar)
- [Register nima va turlari](#register)
- [Clock va clock speed](#clock)
- [ISA: x86 vs ARM, CISC vs RISC](#isa)
- [Pipelining](#pipelining)
- [Superscalar va out-of-order execution](#superscalar)
- [Branch prediction](#branch-prediction)
- [Core nima](#core)
- [Cache (L1/L2/L3)](#cache)
- [CPU issiqlik va TDP](#tdp)
- [CPU qanday milliardlab amal bajaradi](#milliardlab)
- [Savol-javob](#savol-javob)
- [Masalalar](#masalalar)

<a name="vazifa"></a>
## CPU vazifasi

CPU (Central Processing Unit) — instruksiyalarni fetch, decode, execute qiladigan komponent. U RAM'dan instruksiya oladi, uni tushunadi, ALU'da amalni bajaradi va natijani qaytaradi. Butun kompyuterning barcha mantiq va hisob-kitobi CPU'dan o'tadi.

**💡 Tushuncha:** CPU o'zi juda "ahmoq", lekin juda tez. U faqat oddiy amallarni (qo'shish, taqqoslash, ko'chirish, sakrash) biladi — lekin ularni sekundiga milliardlab marta bajaradi. Murakkab dasturlar shu oddiy amallardan tuzilgan.

<a name="qismlar"></a>
## Asosiy qismlar: ALU, Control Unit, Registers

CPU ichida uchta asosiy blok bor.

```text
   ┌─────────────────────────────────────────────┐
   │                    CPU                       │
   │                                              │
   │   ┌──────────────┐      ┌─────────────────┐  │
   │   │ Control Unit │◄────►│    Registers    │  │
   │   │   (CU)       │      │  PC │ IR │ ACC   │  │
   │   │ boshqaradi   │      │  R1 │ R2 │ ...   │  │
   │   └──────┬───────┘      └────────┬────────┘  │
   │          │                       │           │
   │          ▼                       ▼           │
   │   ┌──────────────────────────────────────┐  │
   │   │              ALU                     │  │
   │   │  (Arithmetic Logic Unit)             │  │
   │   │  +  -  ×  ÷  AND  OR  NOT  compare    │  │
   │   └──────────────────────────────────────┘  │
   │                                              │
   └───────────────────┬──────────────────────────┘
                       │  bus
                       ▼
                    ┌──────┐
                    │ RAM  │
                    └──────┘
```

| Blok | Vazifasi |
|------|----------|
| **ALU** (Arithmetic Logic Unit) | Arifmetik (+, −, ×, ÷) va mantiqiy (AND, OR, NOT, taqqoslash) amallarni bajaradi |
| **Control Unit** (CU) | Instruksiyani decode qiladi va boshqa bloklarni boshqaradi (nima qilishni aytadi) |
| **Registers** | CPU ichidagi juda tez, juda kichik xotira (hozir ishlatilayotgan qiymatlar) |

<a name="register"></a>
## Register nima va turlari

**Register** — CPU ichidagi eng tez xotira. Juda kichik (odatda 32 yoki 64 bit), lekin nol kechikish bilan ishlaydi. ALU faqat register'lardagi qiymatlar ustida amal bajaradi.

| Register | Vazifasi |
|----------|----------|
| **PC** (Program Counter / Instruction Pointer) | Keyingi bajariladigan instruksiya manzilini saqlaydi |
| **IR** (Instruction Register) | Hozir bajarilayotgan instruksiyani saqlaydi |
| **Accumulator** (ACC) | Arifmetik amallar natijasini vaqtincha saqlaydi |
| **General-purpose** (R1, R2, RAX, RBX, ...) | Har qanday oraliq qiymat uchun — dasturchi/compiler erkin ishlatadi |
| **Status/Flags** | Amaldan keyingi holatlar (nol chiqdimi? overflow bo'ldimi? manfiymi?) |

```text
   ADD amali register'lar bilan:

   R1 = 5          ┌────┐
   R2 = 3          │ R1 │ 5
                   ├────┤
   ADD R3,R1,R2 ─► │ R2 │ 3   ──► ALU ──► R3 = 8
                   ├────┤
                   │ R3 │ 8
                   └────┘
```

**💡 Tushuncha:** Register'lar juda tez, chunki ular CPU ichida, ALU'ga juda yaqin. Lekin ular juda kam (masalan x86-64'da ~16 ta general-purpose). Shuning uchun ma'lumotni register ↔ RAM o'rtasida doim ko'chirib turishga to'g'ri keladi.

<a name="clock"></a>
## Clock va clock speed

CPU'ni **clock** (soat) signali boshqaradi — bu sekundiga milliardlab marta 0→1→0 o'zgaruvchi signal. Har bir "tik" (cycle) CPU'ga bir qadam ishlash imkonini beradi.

**Clock speed** — sekundiga nechta cycle bo'lishini bildiradi, GHz'da o'lchanadi:
- 1 GHz = sekundiga 1 milliard cycle
- 3 GHz = sekundiga 3 milliard cycle

```text
   Clock signali:
    ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐
    │ │ │ │ │ │ │ │ │ │       har ko'tarilish = 1 cycle
  ──┘ └─┘ └─┘ └─┘ └─┘ └──►
    1   2   3   4   5     cycle raqami
```

**⚠️ Ehtiyot bo'l:** Yuqori GHz har doim ham tezroq degani emas. Bir instruksiya bir necha cycle olishi mumkin (yoki bir cycle'da bir nechta instruksiya bajarilishi mumkin — pipelining/superscalar tufayli). Shuning uchun 3 GHz protsessor 4 GHz'dan tezroq bo'lishi ham mumkin, agar u har cycle'da ko'proq ish qilsa (IPC — Instructions Per Cycle yuqori bo'lsa).

<a name="isa"></a>
## ISA: x86 vs ARM, CISC vs RISC

**ISA** (Instruction Set Architecture) — CPU tushunadigan instruksiyalar to'plami va ularning formatini belgilaydigan "shartnoma". Software va hardware o'rtasidagi chegara.

Ikki asosiy falsafa:

| | CISC | RISC |
|--|------|------|
| To'liq nomi | Complex Instruction Set Computer | Reduced Instruction Set Computer |
| Instruksiyalar | Ko'p, murakkab (bittasi ko'p ish qiladi) | Kam, sodda (har biri oz ish) |
| Instruksiya uzunligi | O'zgaruvchan | Bir xil (fixed) |
| Misol | x86 / x86-64 (Intel, AMD) | ARM, RISC-V |
| Qayerda | PC, server (Intel/AMD) | Telefon, Apple Silicon, embedded |
| Energiya | Ko'proq | Kamroq (samaraliroq) |

**💡 Tushuncha:** x86 (CISC) tarixan PC va serverlarda hukmron. ARM (RISC) telefon va planshetlarda energiyani tejagani uchun g'olib. Apple M-serie chiplari ARM'ga asoslangan. Amalda zamonaviy x86 CPU'lar ichkarida murakkab instruksiyalarni sodda "micro-op"larga bo'lib bajaradi — ya'ni ichkarida RISC'ga o'xshaydi.

<a name="pipelining"></a>
## Pipelining

Oddiy CPU bir instruksiyani to'liq tugatmasdan keyingisini boshlamaydi. **Pipelining** esa — konveyer kabi — bosqichlarni parallel bajarish imkonini beradi.

```text
   Pipelining YO'Q (ketma-ket):
   Instr 1: [F][D][E][W]
   Instr 2:             [F][D][E][W]
   Instr 3:                         [F][D][E][W]
   → 3 instruksiya = 12 cycle

   Pipelining BOR (konveyer):
   Instr 1: [F][D][E][W]
   Instr 2:    [F][D][E][W]
   Instr 3:       [F][D][E][W]
   Instr 4:          [F][D][E][W]
   → har cycle'da yangi instruksiya "tugaydi"

   (F=Fetch, D=Decode, E=Execute, W=Writeback)
```

Fabrikadagi konveyer kabi: bir mashina bo'yalayotganda, ikkinchisi yig'ilayapti, uchinchisi tekshirilayapti — hammasi bir vaqtda, lekin har biri boshqa bosqichda.

**Pipeline hazard** — pipelining'ni buzadigan holatlar:
- **Data hazard:** keyingi instruksiya oldingisining natijasiga bog'liq (hali tayyor emas).
- **Control hazard:** sakrash (branch) bo'ladi — keyingi instruksiya qaysi ekani noma'lum.
- **Structural hazard:** ikki instruksiya bir xil resursni talab qiladi.

**⚠️ Ehtiyot bo'l:** Hazard yuz berganda pipeline "stall" (to'xtash) yoki "flush" (tozalash) qilishga majbur bo'ladi — bu cycle'larni behuda sarflaydi. Shuning uchun branch prediction muhim.

<a name="superscalar"></a>
## Superscalar va out-of-order execution

**Superscalar** — CPU'da bir nechta execution unit bo'lib, u bir cycle'da bir nechta instruksiyani bajara oladi (IPC > 1).

**Out-of-order execution** — CPU instruksiyalarni yozilgan tartibida emas, tayyor bo'lganida bajaradi. Agar bir instruksiya ma'lumot kutayotgan bo'lsa, CPU keyingisini oldinroq bajarib, vaqtni tejaydi. Natija esa oxirida to'g'ri tartibda taqdim etiladi.

```text
   In-order (tartib bilan):
   A (RAM kutadi, sekin) ──► B kutadi ──► C kutadi
       └── bo'sh vaqt behuda ketadi

   Out-of-order (aqlli):
   A (RAM kutayapti...) 
   B, C ni oldinroq bajaramiz (ular tayyor)
   A tayyor bo'lgach, natija to'g'ri tartibga keltiriladi
```

**💡 Tushuncha:** Bu texnikalar tufayli zamonaviy CPU bir cycle'da o'rtacha 1 dan ko'p instruksiya bajaradi — garchi ular ketma-ket yozilgan bo'lsa ham.

<a name="branch-prediction"></a>
## Branch prediction

Kod `if` va sikllar bilan to'la — ya'ni "sakrash" (branch) juda ko'p. Pipeline uchun muammo: sakrash natijasi ma'lum bo'lguncha, keyingi instruksiya qaysi ekani noma'lum.

**Branch prediction** — CPU sakrash qaysi tomonga borishini oldindan taxmin qiladi va shu yo'nalishdagi instruksiyalarni pipeline'ga oldindan yuklaydi.

```text
   if (x > 0) {  ← CPU: "odatda TRUE bo'ladi" deb taxmin qiladi
       ...       ← shu shoxdan instruksiyalarni oldindan yuklaydi
   }

   Taxmin TO'G'RI  → vaqt tejaldi (pipeline uzilmadi)
   Taxmin NOTO'G'RI → pipeline flush, cycle'lar yo'qoldi (misprediction)
```

**💡 Tushuncha:** Zamonaviy CPU'lar branch prediction'da ~95%+ aniqlikka erishadi, o'tmish tarixiga qarab o'rganadi. To'g'ri taxmin — tekin tezlik; noto'g'ri taxmin — jarima (bir necha cycle behuda).

<a name="core"></a>
## Core nima

**Core** (yadro) — mustaqil ishlay oladigan to'liq CPU birligi (o'z ALU, CU, register'lari bilan). Zamonaviy CPU'da bir nechta core bor (multi-core), ular bir vaqtda alohida vazifalarni parallel bajara oladi.

```text
   4-core CPU:
   ┌──────────────────────────────────────────┐
   │  ┌────────┐ ┌────────┐ ┌────────┐ ┌──────┐│
   │  │ Core 0 │ │ Core 1 │ │ Core 2 │ │Core 3││
   │  │ L1 L2  │ │ L1 L2  │ │ L1 L2  │ │L1 L2 ││
   │  └────────┘ └────────┘ └────────┘ └──────┘│
   │  ┌────────────────────────────────────────┐│
   │  │        L3 Cache (umumiy)               ││
   │  └────────────────────────────────────────┘│
   └──────────────────────────────────────────┘
```

**💡 Tushuncha:** 4 core degani — 4 ta vazifani chindan parallel bajarish mumkin. Lekin dastur "multi-threaded" yozilmagan bo'lsa, faqat bitta core'dan foydalanadi. Ko'p core'dan foyda olish uchun dastur parallel yozilishi kerak.

<a name="cache"></a>
## Cache (L1/L2/L3)

RAM CPU uchun sekin. **Cache** — CPU ichidagi kichik, juda tez xotira: yaqinda ishlatilgan ma'lumotni saqlaydi, RAM'ga borishni kamaytiradi.

```text
   CPU ichida cache darajalari:

   ┌──────┐  eng tez     ┌────┐   ┌────┐   ┌──────┐   ┌─────┐
   │ Reg. │ ───────────► │ L1 │ ─►│ L2 │ ─►│  L3  │ ─►│ RAM │
   └──────┘              └────┘   └────┘   └──────┘   └─────┘
   ~0.3ns    ~1ns        ~4ns      ~12ns    ~40ns      ~100ns
   KB        ~64KB       ~256KB-1MB  ~ MB     GB
```

| Cache | Tezlik | Hajm | Joyi |
|-------|--------|------|------|
| L1 | Eng tez | Eng kichik (~KB) | Har core ichida (ko'pincha L1i + L1d) |
| L2 | O'rta | O'rta (~KB–MB) | Har core ichida |
| L3 | Sekinroq (RAM'dan tez) | Katta (~MB) | Barcha core'lar uchun umumiy |

Cache va memory hierarchy haqida batafsil: keyingi mavzu — [Xotira Ierarxiyasi](./03-memory-hierarchy.md).

**⚠️ Ehtiyot bo'l:** "Cache hit" — kerakli ma'lumot cache'da bor (tez). "Cache miss" — yo'q, RAM'dan olib kelish kerak (sekin). Yaxshi yozilgan dastur ma'lumotni ketma-ket ishlatib, cache hit'ni oshiradi.

<a name="tdp"></a>
## CPU issiqlik va TDP

CPU ishlaganda transistorlar tez o'chib-yonadi va bu issiqlik chiqaradi. **TDP** (Thermal Design Power) — CPU o'rtacha maksimal yukda qancha issiqlik (vatt'da) chiqarishini bildiradi va sovutish tizimi shunga mos tanlanadi.

**💡 Tushuncha:** Ko'proq clock speed va ko'proq core — ko'proq issiqlik. Shuning uchun CPU'lar sovutgichga (kuler, radiator, ba'zan suvli sovutish) muhtoj. Agar CPU qizib ketsa, u o'zini himoya qilish uchun tezligini pasaytiradi (thermal throttling).

<a name="milliardlab"></a>
## CPU qanday milliardlab amal bajaradi

Bir CPU sekundiga milliardlab amal bajarishga qanday erishadi? Bir necha texnika birlashadi:

```text
   Sekundiga amal soni ≈
       clock speed  (masalan 3×10⁹ cycle/s)
     × core soni    (masalan 8 core)
     × IPC          (bir cycle'da o'rtacha instruksiya, masalan ~3)
     × pipelining   (bosqichlar parallel)

   Natija: sekundiga o'nlab milliardlab amal
```

1. **Yuqori clock speed** — sekundiga milliardlab cycle.
2. **Pipelining** — har cycle'da yangi instruksiya "tugaydi".
3. **Superscalar** — bir cycle'da bir nechta instruksiya.
4. **Multi-core** — bir nechta core parallel.
5. **Cache** — RAM kutishni kamaytiradi, cycle'lar behuda ketmaydi.

**💡 Tushuncha:** Bu texnikalarning hech biri yolg'iz sehr emas — lekin birgalikda ular CPU'ni sekundiga o'nlab milliardlab amalga qodir qiladi.

<a name="savol-javob"></a>
## Savol-javob

### ❓ ALU va Control Unit farqi nima?

**✅ Javob:** ALU haqiqiy amalni bajaradi (qo'shish, taqqoslash, mantiqiy amallar). Control Unit esa "dirijyor" — instruksiyani decode qiladi va ALU, register, xotiraga nima qilishni buyuradi. ALU hisoblaydi, CU boshqaradi.

### ❓ Register nima uchun kerak, RAM yetmaydimi?

**✅ Javob:** ALU faqat register'lardagi qiymatlar ustida ishlay oladi va register'lar RAM'dan ~100 baravar tez. RAM juda sekin bo'lgani uchun, har amal uchun undan o'qish CPU'ni sekinlashtirardi. Register'lar CPU ichida, ALU'ga eng yaqin.

### ❓ Program Counter (PC) nima saqlaydi?

**✅ Javob:** PC (Instruction Pointer) keyingi bajariladigan instruksiyaning xotiradagi manzilini saqlaydi. Har fetch'dan keyin u avtomatik keyingi instruksiyaga o'tadi; sakrash (branch) instruksiyasi uni boshqa manzilga o'zgartiradi.

### ❓ Clock speed nima va GHz nimani bildiradi?

**✅ Javob:** Clock speed — CPU sekundiga nechta cycle bajarishi. 1 GHz = 1 milliard cycle/sekund. 3 GHz protsessor sekundiga 3 milliard marta "tiklaydi", har tik CPU'ga bir qadam beradi.

### ❓ Yuqori GHz doim tezroq CPU degani-mi?

**✅ Javob:** Yo'q. Tezlik = clock speed × IPC × core soni. Past GHz'li CPU har cycle'da ko'proq ish qilsa (yuqori IPC yoki ko'proq core), yuqori GHz'li CPU'dan tezroq bo'lishi mumkin. Faqat GHz'ga qarab taqqoslash noto'g'ri.

### ❓ CISC va RISC farqi nima?

**✅ Javob:** CISC (x86) — kam sonli, lekin murakkab instruksiyalar, bittasi ko'p ish qiladi. RISC (ARM, RISC-V) — ko'p sonli sodda instruksiyalar, har biri oz ish qiladi, bir xil uzunlikda. RISC odatda energiyani tejaydi, shuning uchun telefonlarda ustun.

### ❓ x86 va ARM qayerda ishlatiladi?

**✅ Javob:** x86 (Intel/AMD) — PC, laptop, serverlarda ustun. ARM — telefon, planshet, embedded qurilmalar va Apple Silicon (M-serie)'da, energiya samaradorligi tufayli. Serverlarda ham ARM tobora ko'payyapti.

### ❓ Pipelining nima va qanday tezlashtiradi?

**✅ Javob:** Pipelining — instruksiya bosqichlarini (fetch/decode/execute/writeback) konveyer kabi parallel bajarish. Bir instruksiya decode qilinayotganda, ikkinchisi fetch qilinadi. Natijada har cycle'da yangi instruksiya "tugaydi", umumiy tezlik oshadi.

### ❓ Pipeline hazard nima?

**✅ Javob:** Pipeline'ni buzadigan holatlar: data hazard (keyingi instruksiya oldingisining natijasiga bog'liq), control hazard (branch/sakrash tufayli keyingi instruksiya noma'lum), structural hazard (resurs to'qnashuvi). Hazard cycle'larni behuda sarflashi mumkin.

### ❓ Out-of-order execution nima beradi?

**✅ Javob:** CPU instruksiyalarni yozilgan tartibida emas, tayyor bo'lganida bajaradi. Agar bir instruksiya RAM kutayotgan bo'lsa, CPU keyingi tayyor instruksiyalarni oldinroq bajarib vaqtni tejaydi. Natija oxirida to'g'ri tartibda taqdim etiladi.

### ❓ Branch prediction nima uchun kerak?

**✅ Javob:** `if`/sikllar sakrash yaratadi; pipeline uchun keyingi instruksiya qaysi ekani noma'lum bo'lib qoladi. Branch prediction natijani oldindan taxmin qilib, pipeline'ni to'ldiradi. To'g'ri taxmin tezlik beradi; noto'g'ri taxmin pipeline flush (jarima) keltiradi.

### ❓ Core nima va multi-core nima beradi?

**✅ Javob:** Core — mustaqil to'liq CPU birligi. Multi-core CPU bir nechta vazifani chindan parallel bajara oladi. Lekin dastur multi-threaded yozilmasa, faqat bitta core ishlaydi — ko'p core'dan foyda olish uchun parallel dasturlash kerak.

### ❓ L1, L2, L3 cache farqi nima?

**✅ Javob:** L1 — eng tez, eng kichik, har core ichida (ko'pincha instruction va data alohida). L2 — o'rta tezlik va hajm, har core ichida. L3 — sekinroq (lekin RAM'dan tez), katta, barcha core'lar uchun umumiy. Yaqinlik va tezlik teskari proporsional hajm bilan.

### ❓ TDP nima anglatadi?

**✅ Javob:** TDP (Thermal Design Power) — CPU maksimal yukda qancha issiqlik (vatt) chiqarishi. Sovutish tizimi TDP'ga qarab tanlanadi. CPU qizib ketsa, o'zini himoya qilib tezligini pasaytiradi (thermal throttling).

### ❓ CPU qanday qilib sekundiga milliardlab amal bajaradi?

**✅ Javob:** Bir necha texnika birlashadi: yuqori clock speed (milliardlab cycle/s), pipelining (bosqichlar parallel), superscalar (bir cycle'da bir nechta instruksiya), multi-core (parallel core'lar), va cache (RAM kutishni kamaytiradi). Birgalikda ular o'nlab milliardlab amal/sekundga erishadi.

<a name="masalalar"></a>
## Masalalar

> Yechimlar: [solutions/hardware/02-cpu.md](../solutions/hardware/02-cpu.md)

1. 3.2 GHz CPU sekundiga nechta clock cycle bajaradi? Agar bir instruksiya o'rtacha 4 cycle olsa (pipelining'siz), sekundiga nechta instruksiya bajariladi?

2. Bir CPU 8 core, 4 GHz, IPC = 3. Nazariy maksimal instruksiya/sekund ni hisoblang. Nega amalda bu songa erishilmaydi?

3. ALU, Control Unit va registerlarning har biri `z = a + b` amalida qanday rol o'ynashini bosqichma-bosqich yozing.

4. CISC va RISC ni beshta mezon (instruksiya soni, uzunlik, murakkablik, energiya, misol) bo'yicha jadvalда taqqoslang.

5. Pipeline'da 5 bosqich bor. 5 instruksiyani pipelining bilan va pipelining'siz bajarishga nechta cycle ketishini hisoblang va farqini tushuntiring.

6. Branch prediction 90% aniqlikda ishlaydi. Har misprediction 15 cycle jarima keladi. 1000 ta branch'da o'rtacha nechta cycle jarima uchun sarflanadi?

7. Nima uchun L1 cache L3'dan kichik, lekin tezroq? Tezlik-hajm-narx muvozanatini tushuntiring.

8. Bir dastur faqat bitta core'dan foydalanyapti, lekin CPU'da 8 core bor. Muammo nimada va uni qanday hal qilish mumkin? Konseptual javob bering.

9. CPU'ni 4 GHz'dan 5 GHz'ga oshirdik. Tezlik oshdi, lekin TDP ham oshdi. Bu qanday muammolarga olib keladi va thermal throttling nima?

← [Hardware bo'limiga qaytish](./README.md)
