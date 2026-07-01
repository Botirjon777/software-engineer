# Bus'lar va Komponentlar Muloqoti — Yechimlar

Bu fayl [hardware/06-buses-communication.md](../../hardware/06-buses-communication.md) mashqlarining yechimlari. Avval o'zingiz yechib ko'ring, keyin solishtiring.

## Mundarija

- [1-masala: Baytni o'qish qadamlari](#1-masala)
- [2-masala: 40-bit address bus](#2-masala)
- [3-masala: Polling qachon yaxshi](#3-masala)
- [4-masala: DMA'siz fayl ko'chirish](#4-masala)
- [5-masala: Controller'lar CPU ichida](#5-masala)
- [6-masala: GPU'ga ko'proq lane](#6-masala)
- [7-masala: To'liq diagramma](#7-masala)
- [8-masala: Memory-mapped vs port-mapped](#8-masala)
- [9-masala: Bottleneck](#9-masala)
- [10-masala: SATA vs NVMe](#10-masala)

---

## 1-masala

**Savol:** Xotiradan bitta baytni o'qish — bus'lar roli qadamma-qadam.

**Yechim:**
1. CPU o'qmoqchi bo'lgan manzilni **address bus**ga qo'yadi (masalan `0x1000`).
2. CPU **control bus**ga "READ" signalini yuboradi.
3. RAM manzilni oladi, kerakli baytni topadi.
4. RAM baytni **data bus** orqali CPU'ga qaytaradi.
5. Control bus orqali "tayyor" (ready) signali bilan CPU ma'lumot kelganini biladi.

Uchala bus birga ishlaydi: address — qayerdan, control — nima qilish, data — haqiqiy qiymat.

## 2-masala

**Savol:** 40-bit address bus qancha xotira?

**Yechim:**
```
   2^40 bayt = 1,099,511,627,776 bayt
             = 1024 GB
             = 1 TB
```
40-bit address bus **1 TB** xotirani adreslay oladi. (Har qo'shimcha bit adreslanadigan hajmni ikki baravar oshiradi: 2³² = 4 GB, 2⁴⁰ = 1 TB.)

## 3-masala

**Savol:** Polling qachon interrupt'dan yaxshiroq?

**Yechim:** Polling qurilma **juda tez-tez va deyarli doim tayyor** bo'lganda yaxshiroq. Sabab: interrupt'ning o'z narxi bor (kontekst almashish, handler'ga o'tish). Agar hodisa sekundiga million marta bo'lsa, har safar interrupt qilish bu overhead'ni juda oshiradi.

**Amaliy misol:** Yuqori tezlikli tarmoq kartasi (10/100 Gbps). Paketlar shunchalik tez keladiki, har biriga interrupt qilish CPU'ni bo'g'adi (interrupt storm). Shuning uchun bunday holatda **polling** (masalan Linux NAPI, DPDK) ishlatiladi — CPU bir sikl ichida ko'p paketni birdan o'qiydi.

## 4-masala

**Savol:** DMA'siz 1 GB ko'chirish CPU'ga ta'siri; DMA yechimi.

**Yechim:** DMA'siz (programmed I/O) CPU har bir bo'lakni o'zi ko'chiradi: disk'dan registrga o'qiydi, keyin RAM'ga yozadi — million marta. Bu davomida CPU **butunlay band**, boshqa ish qilolmaydi, tizim sekinlashadi.

DMA bilan:
```
   CPU: "DMA, disk'dan RAM'ning shu diapazoniga 1 GB ko'chir"
        │
        └──► [DMA controller] ── ma'lumotni CPU'siz ko'chiradi
        │
   CPU: boshqa ish bilan band bo'ladi
        │
   Tugagach: DMA ──IRQ──► CPU ("tayyor")
```
CPU faqat boshida buyruq beradi va oxirida IRQ oladi — orada erkin. Shuning uchun fayl ko'chirilayotganda ham kompyuter javob berib turadi.

## 5-masala

**Savol:** Nega memory/PCIe controller CPU ichiga ko'chdi?

**Yechim:** Eski tizimda CPU ↔ northbridge ↔ RAM zanjiri bor edi — har qadam **latency** qo'shardi. Controller'ni CPU ichiga olib kirish:

- **Latency kamayadi:** CPU RAM va GPU bilan to'g'ridan-to'g'ri gaplashadi, orada chip yo'q.
- **Bandwidth oshadi:** to'g'ridan-to'g'ri, kengroq ulanish.
- **Soddalik:** motherboard'da faqat bitta I/O chipset qoladi, dizayn arzonlashadi.

Natijada zamonaviy CPU'lar RAM va PCIe (GPU/NVMe) bilan ancha tez ishlaydi.

## 6-masala

**Savol:** Nega GPU'ga x16, NVMe'ga x4?

**Yechim:** Lane soni bandwidth'ni belgilaydi — har lane qo'shimcha parallel yo'l. GPU juda katta ma'lumot oqimini talab qiladi: tekstura, geometriya, ML uchun ulkan tensorlar RAM↔VRAM ko'chiriladi. Shu sababli maksimal bandwidth (x16) kerak.

NVMe SSD tezlik ham kerak, lekin GPU darajasida emas — x4 lane storage uchun yetarli bandwidth beradi. Lane'lar cheklangan resurs, shuning uchun ular eng ko'p talab qiladigan qurilmaga (GPU) beriladi.

## 7-masala

**Savol:** To'liq ulanish diagrammasi.

**Yechim:**

```
                    ┌───────────────────┐
                    │       CPU         │
                    │ [Memory Ctrl]─────┼══ memory bus ══► [ RAM ]
                    │ [PCIe Ctrl]───────┼══ PCIe x16 ═════► [ GPU ]
                    └─────────┬─────────┘
                              │ (DMI bus)
                    ┌─────────┴─────────┐
                    │     CHIPSET       │
                    └──┬─────────────┬──┘
                       │             │
              PCIe x4  │             │  USB
           [ NVMe SSD ]┘             └[ Klaviatura ]
```

Ulanishlar:
- CPU ↔ RAM: **memory bus**
- CPU ↔ GPU: **PCIe x16**
- CPU ↔ chipset: **DMI** (ichki bus)
- Chipset ↔ NVMe SSD: **PCIe x4**
- Chipset ↔ klaviatura: **USB**

## 8-masala

**Savol:** Memory-mapped vs port-mapped jadval + qaysi ustun.

**Yechim:**

| Nuqta | Memory-mapped I/O | Port-mapped I/O |
|-------|-------------------|-----------------|
| Address fazosi | Xotira bilan bir | Alohida port fazosi |
| Kirish usuli | Oddiy `load`/`store` | Maxsus `IN`/`OUT` instruksiya |
| Dasturlash | Sodda (xotira kabi) | Alohida instruksiya kerak |
| Arxitektura | ARM, RISC-V, zamonaviy | Eski x86 |

**Ustun:** Memory-mapped I/O zamonaviy arxitekturalarda ustun. Sabab: alohida instruksiya kerak emas, qurilma xuddi xotiradek ko'riladi — kod sodda va bir xil address fazosidan foydalanadi.

## 9-masala

**Savol:** Tez CPU, tez RAM, sekin storage — katta faylda tezlik nimaga bog'liq?

**Yechim:** Umumiy tezlik **eng sekin qismga** — storage'ga bog'lanadi (bottleneck). Katta fayl bilan ishlaganda ma'lumot avval disk'dan o'qilishi kerak; CPU va RAM tez bo'lsa ham, ular disk ma'lumot yetkazguncha kutadi.

Xulosa: CPU va RAM ko'p vaqt bo'sh turadi (I/O-bound holat). Yaxshilash — tezroq storage (NVMe), keshlash (kerakli ma'lumotni RAM'da saqlash), yoki ketma-ket kirishni optimallashtirish.

## 10-masala

**Savol:** SATA SSD vs NVMe SSD arxitektura farqi; nega NVMe tez?

**Yechim:**
- **SATA SSD:** eski SATA protokoli va bus orqali ulanadi. SATA aslida aylanuvchi HDD davri uchun mo'ljallangan — bitta buyruq navbati, ~600 MB/s cheklovi, yuqoriroq latency.
- **NVMe SSD:** to'g'ridan-to'g'ri **PCIe** orqali ulanadi (masalan x4). NVMe protokoli flesh xotira uchun yaratilgan: minglab parallel buyruq navbati, past latency, bir necha barobar yuqori bandwidth.

**Nega NVMe tez:** (1) PCIe bus SATA'dan ancha keng; (2) protokol ko'p parallel navbatlarni qo'llab, SSD'ning ichki parallelligidan to'liq foydalanadi; (3) orada eski SATA controller yo'q, latency past.

---

← [Hardware bo'limiga qaytish](../../hardware/README.md)
