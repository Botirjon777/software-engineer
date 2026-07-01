# Bus'lar va Komponentlar Muloqoti

Kompyuter — bu bir necha komponentning (CPU, RAM, GPU, storage, I/O qurilmalari) yig'indisi. Lekin ular qanday **bir-biri bilan gaplashadi**? Ma'lumot CPU'dan RAM'ga, disk'dan GPU'ga qanday yetib boradi? Bu bo'lim aynan shu savolga — "elementlar o'zaro qanday muloqot qiladi" — batafsil javob beradi: bus'lar, chipset, PCIe, interrupt, DMA va boshqalar.

**💡 Tushuncha:** Komponentlar simlar to'plami — **bus** orqali bog'lanadi. Bus — bir necha qurilma umumiy ishlatadigan "yo'l"; unda kim qachon ma'lumot uzatishi qat'iy tartibga solinadi.

## Mundarija

- [Bus nima](#bus-nima)
- [Bus turlari: address, data, control](#bus-turlari-address-data-control)
- [System bus va motherboard](#system-bus-va-motherboard)
- [Chipset: northbridge/southbridge](#chipset-northbridgesouthbridge)
- [PCIe va lane'lar](#pcie-va-lanelar)
- [Memory bus](#memory-bus)
- [I/O qanday ishlaydi](#io-qanday-ishlaydi)
- [Interrupt (IRQ) va polling](#interrupt-irq-va-polling)
- [DMA — Direct Memory Access](#dma--direct-memory-access)
- [Memory-mapped vs port-mapped I/O](#memory-mapped-vs-port-mapped-io)
- [Bus bandwidth va bottleneck](#bus-bandwidth-va-bottleneck)
- [USB, SATA, NVMe](#usb-sata-nvme)
- [To'liq tizim diagrammasi](#toliq-tizim-diagrammasi)
- [Savol-Javob](#savol-javob)
- [Masalalar](#masalalar)

---

## Bus nima

**Bus** — bu komponentlar o'rtasida ma'lumot uzatuvchi umumiy simlar (o'tkazgichlar) to'plami. "Umumiy" so'zi muhim: bir necha qurilma bitta bus'ga ulanadi va navbatma-navbat foydalanadi.

```
   ┌─────┐   ┌─────┐   ┌─────┐
   │ CPU │   │ RAM │   │ GPU │
   └──┬──┘   └──┬──┘   └──┬──┘
      │         │         │
   ═══╧═════════╧═════════╧═══  ← BUS (umumiy yo'l)
```

Bus'ning ikki asosiy xarakteristikasi:
- **Kenglik (width)** — bir vaqtda nechta bit uzatiladi (masalan 64-bit).
- **Chastota (clock)** — sekundiga necha marta uzatish bo'ladi.

## Bus turlari: address, data, control

Klassik system bus uch mantiqiy qismdan iborat. Har biri boshqa narsa uzatadi:

| Bus turi | Nima uzatadi | Yo'nalish |
|----------|--------------|-----------|
| **Address bus** | Qaysi manzil (xotira/qurilma) bilan ishlanadi | CPU → (bir tomonlama) |
| **Data bus** | Haqiqiy ma'lumot (o'qildi/yozildi) | Ikki tomonlama |
| **Control bus** | Buyruq signallari: read/write, tayyor, interrupt | Ikki tomonlama |

```
   CPU ──[ADDRESS BUS]──► "0x1000 manzilga"
   CPU ◄─[ DATA BUS  ]──► "42 qiymati"
   CPU ──[CONTROL BUS]──► "READ" yoki "WRITE" signali
```

**💡 Tushuncha:** Xotiradan o'qish: CPU address bus'ga manzilni qo'yadi, control bus'ga "READ" signalini yuboradi, RAM esa data bus orqali qiymatni qaytaradi. Uch bus birga ishlaydi.

**⚠️ Ehtiyot bo'l:** Address bus kengligi maksimal adreslanadigan xotirani belgilaydi. 32-bit address bus = 2³² = 4 GB. Shuning uchun eski 32-bit tizimlar 4 GB'dan ortiq RAM'ni to'liq ishlatolmaydi.

## System bus va motherboard

**System bus** — CPU'ni asosiy komponentlarga (RAM, chipset) bog'lovchi asosiy bus. **Motherboard** (asosiy plata) — barcha komponentlar ulanadigan fizik plata; undagi bus'lar, slotlar va chipset komponentlarni bog'laydi.

Motherboard rolini shunday tasavvur qiling: u "yo'llar tarmog'i", chipset esa "svetofor/tugun" — ma'lumot qayerga ketishini boshqaradi.

## Chipset: northbridge/southbridge

Ilgari motherboard'da ikkita chip bor edi:

```
                ┌─────┐
                │ CPU │
                └──┬──┘
       (tez)       │
   ┌───────────────┴──────────────┐
   │        NORTHBRIDGE            │──► RAM (memory bus)
   │  (tez qurilmalar: RAM, GPU)  │──► GPU (grafika)
   └───────────────┬──────────────┘
                    │
   ┌────────────────┴─────────────┐
   │        SOUTHBRIDGE            │──► SATA (disk)
   │  (sekin I/O: disk, USB, PCI) │──► USB, tarmoq, audio
   └──────────────────────────────┘
```

- **Northbridge** — CPU'ga eng yaqin, tez komponentlarni (RAM, GPU) boshqaradi.
- **Southbridge** — sekinroq I/O qurilmalarini (disk, USB, tarmoq) boshqaradi.

**Zamonaviy integratsiya:** Bugun northbridge funksiyalari (memory controller, PCIe) **CPU ichiga** ko'chirilgan. Motherboard'da faqat bitta chipset (eski southbridge o'rnida) qoldi. Bu latency'ni kamaytiradi — CPU RAM va GPU bilan to'g'ridan-to'g'ri gaplashadi.

## PCIe va lane'lar

**PCIe** (Peripheral Component Interconnect Express) — GPU, NVMe SSD, tarmoq kartalari kabi tez kengaytma qurilmalarini ulash uchun asosiy zamonaviy bus.

PCIe **lane** (yo'lak) tushunchasidan foydalanadi. Har lane — mustaqil ikki tomonlama ketma-ket ulanish. Qurilmalar bir necha lane ishlatishi mumkin:

```
   x1  = 1 lane   (sekin, masalan tarmoq kartasi)
   x4  = 4 lane   (NVMe SSD)
   x16 = 16 lane  (GPU — eng ko'p bandwidth)
```

Ko'proq lane = ko'proq bandwidth. GPU odatda x16 slotga o'rnatiladi. PCIe har avlodda tezligini ikki baravar oshiradi (PCIe 3.0 → 4.0 → 5.0).

## Memory bus

**Memory bus** — CPU (memory controller) va RAM o'rtasidagi maxsus bus. Bu eng ko'p ishlatiladigan yo'llardan biri, chunki CPU doim RAM'dan ma'lumot/instruksiya o'qiydi.

```
   ┌─────┐  memory bus (tez, keng)  ┌─────┐
   │ CPU │ ◄══════════════════════► │ RAM │
   └─────┘   masalan 64-bit x kanal └─────┘
```

Zamonaviy CPU'larda **memory controller** CPU ichida, shuning uchun latency past. Ko'p **kanal** (dual/quad-channel) bandwidth'ni oshiradi.

## I/O qanday ishlaydi

I/O (Input/Output) — CPU tashqi qurilmalar (klaviatura, disk, tarmoq) bilan ma'lumot almashishi. CPU qurilmaga ma'lumot yuborishi yoki undan olishi kerak. Buning uchun bir necha usul bor: **polling**, **interrupt** va **DMA**. Ularni ketma-ket ko'rib chiqamiz.

## Interrupt (IRQ) va polling

**Muammo:** klaviatura tugmasi bosilganini CPU qanday biladi?

**Polling** — CPU doimiy ravishda qurilmadan so'raydi: "tayyormisan? tayyormisan?". Bu CPU vaqtini behuda sarflaydi.

**Interrupt (IRQ, Interrupt Request)** — qurilma o'zi tayyor bo'lganda CPU'ni "chaqiradi": control bus orqali interrupt signalini yuboradi. CPU joriy ishni to'xtatib, **interrupt handler**ni bajaradi, so'ng ishni davom ettiradi.

```
   POLLING:                    INTERRUPT:
   CPU: tayyormisan?           CPU: o'z ishini qiladi...
   Qurilma: yo'q               Qurilma: [tayyor!] ──IRQ──► CPU
   CPU: tayyormisan?           CPU: to'xtaydi → handler → davom
   Qurilma: yo'q               (CPU vaqti behuda ketmaydi)
   ... (vaqt behuda)
```

| | Polling | Interrupt |
|--|---------|-----------|
| CPU yuki | Yuqori (doim so'raydi) | Past (chaqirilganda ishlaydi) |
| Latency | O'zgaruvchan | Past |
| Qachon yaxshi | Juda tez-tez hodisa | Kamdan-kam / kutilmagan hodisa |

**💡 Tushuncha:** Interrupt sabab CPU million marta "tayyormisan?" deb so'ramasdan, boshqa foydali ish bilan band bo'ladi. Bu zamonaviy tizimlarning samaradorligi asosi.

## DMA — Direct Memory Access

Katta ma'lumot (masalan disk'dan 1 GB fayl) ko'chirilganda, agar har baytni CPU ko'chirsa — CPU butunlay band bo'lib qoladi.

**DMA** (Direct Memory Access) — maxsus kontroller qurilma va RAM o'rtasida ma'lumotni **CPU'siz** ko'chiradi. CPU faqat "shu diapazonni ko'chir" deb buyruq beradi, keyin boshqa ish bilan davom etadi. Ko'chirish tugagach, DMA kontroller CPU'ni **interrupt** bilan xabardor qiladi.

```
   DMA'siz:  Disk ──► CPU ──► RAM   (CPU har baytni ushlaydi, band)

   DMA bilan: Disk ──► [DMA controller] ──► RAM
                              │
              CPU: boshqa ish bilan band, oxirida IRQ oladi
```

**⚠️ Ehtiyot bo'l:** DMA bo'lmasa, disk yoki tarmoq bilan ishlash CPU'ni butunlay egallab, tizimni sekinlashtiradi. DMA — yuqori tezlikli I/O uchun majburiy: shuning uchun fayl ko'chirilayotganda ham kompyuter javob berib turadi.

## Memory-mapped vs port-mapped I/O

CPU qurilma registrlariga qanday murojaat qiladi? Ikki yondashuv:

| | Memory-mapped I/O | Port-mapped I/O |
|--|-------------------|-----------------|
| G'oya | Qurilma registrlari xotira manzili sifatida ko'rinadi | Qurilma alohida "port" manzil fazosida |
| Kirish | Oddiy `load/store` (xotira kabi) | Maxsus `IN`/`OUT` instruksiyalar |
| Address fazosi | Xotira bilan bir | Alohida |
| Zamonaviy | Keng tarqalgan (ARM, RISC-V) | Asosan eski x86 |

**💡 Tushuncha:** Memory-mapped I/O'da dasturchi qurilmani xuddi xotiradek o'qiydi/yozadi — alohida instruksiya kerak emas. Bu soddaroq va zamonaviy arxitekturalarda ustunlik qiladi.

## Bus bandwidth va bottleneck

**Bandwidth** — bus sekundiga qancha ma'lumot uzata olishi (masalan GB/s). Formula soddalashtirilgan holda:

```
   bandwidth ≈ bus_kengligi (bit) × chastota (Hz) / 8   → bayt/sek
```

**Bottleneck** (torbo'g'iz) — tizimning eng sekin qismi umumiy tezlikni cheklaydi. Agar GPU juda tez hisoblasa, lekin PCIe bus ma'lumotni yetkazib berolmasa — bottleneck PCIe'da.

```
   [Tez CPU] ──► [SEKIN BUS] ──► [Tez RAM]
                     ▲
              bottleneck shu yerda:
              umumiy tezlik shunga bog'liq
```

## USB, SATA, NVMe

Amaliy interfeys/bus'lar:

| Interfeys | Nima uchun | Xususiyat |
|-----------|-----------|-----------|
| **USB** | Tashqi qurilmalar (klaviatura, flesh, telefon) | Universal, hot-plug, turli tezliklar |
| **SATA** | An'anaviy HDD/SSD ulanishi | Sekinroq (~600 MB/s cheklov) |
| **NVMe** | Zamonaviy SSD, PCIe orqali | Juda tez, past latency (parallel navbatlar) |

**⚠️ Ehtiyot bo'l:** SATA SSD va NVMe SSD ikkalasi ham SSD, lekin NVMe to'g'ridan-to'g'ri PCIe orqali ulanib, SATA'dan bir necha barobar tez. SATA — eski protokol, disk davri uchun mo'ljallangan.

## To'liq tizim diagrammasi

Hamma komponent qanday bog'lanishi:

```
                         ┌───────────────────┐
                         │        CPU        │
                         │  ┌─────────────┐  │
                         │  │ Memory Ctrl │  │◄════ memory bus ════► [ RAM ]
                         │  └─────────────┘  │
                         │  ┌─────────────┐  │
                         │  │ PCIe Ctrl   │  │◄══ PCIe x16 ══► [ GPU + VRAM ]
                         │  └─────────────┘  │
                         └─────────┬─────────┘
                                   │ (DMI / bus)
                         ┌─────────┴─────────┐
                         │      CHIPSET      │
                         │   (I/O tugun)     │
                         └──┬────┬────┬───┬──┘
                            │    │    │   │
              PCIe x4 ──────┘    │    │   └────── USB ──► klaviatura,
              [ NVMe SSD ]       │    │                   flesh, telefon
                                 │    └──── SATA ──► [ HDD / SSD ]
                                 │
                                 └──── tarmoq (Ethernet), audio

   Signal turlari bus ichida: [ADDRESS] [DATA] [CONTROL]
   Ko'chirish: DMA (CPU'siz),  Xabar: INTERRUPT (IRQ)
```

Muloqotning umumiy ssenariysi (disk'dan fayl o'qish):
1. CPU chipset orqali SSD'ga "shu blokni o'qi" buyrug'ini yuboradi (control/address).
2. SSD ma'lumotni tayyorlaydi va **DMA** orqali to'g'ridan-to'g'ri RAM'ga ko'chiradi (CPU band emas).
3. Tugagach SSD **interrupt** (IRQ) yuboradi.
4. CPU handler'ni bajaradi va ma'lumot RAM'da tayyor bo'ladi.

---

## Savol-Javob

### ❓ Bus nima?

**✅ Javob:** Komponentlar o'rtasida ma'lumot uzatuvchi umumiy simlar to'plami. Bir necha qurilma bitta bus'ga ulanib, navbatma-navbat foydalanadi. Asosiy xarakteristikalari — kenglik (bit) va chastota.

### ❓ Address, data va control bus nima uzatadi?

**✅ Javob:** Address bus — qaysi manzil bilan ishlanishini; data bus — haqiqiy ma'lumotni; control bus — buyruq signallarini (read/write, interrupt, tayyor). Uchtasi birga ishlaydi.

### ❓ Address bus kengligi nimaga ta'sir qiladi?

**✅ Javob:** Maksimal adreslanadigan xotira hajmiga. 32-bit address bus = 2³² = 4 GB. Shuning uchun 32-bit tizimlar 4 GB'dan ortiq RAM'ni to'liq ishlatolmaydi.

### ❓ System bus nima?

**✅ Javob:** CPU'ni asosiy komponentlarga (RAM, chipset) bog'lovchi asosiy bus. Motherboard esa barcha komponent ulanadigan fizik plata.

### ❓ Northbridge va southbridge nima?

**✅ Javob:** Eski chipset ikki qismi. Northbridge — CPU'ga yaqin, tez qurilmalarni (RAM, GPU) boshqaradi. Southbridge — sekin I/O (disk, USB, tarmoq). Zamonaviy tizimlarda northbridge funksiyalari CPU ichiga ko'chgan.

### ❓ Zamonaviy chipset qanday o'zgardi?

**✅ Javob:** Memory controller va PCIe controller CPU ichiga ko'chdi. CPU RAM va GPU bilan to'g'ridan-to'g'ri gaplashadi (past latency), motherboard'da faqat I/O uchun bitta chipset qoladi.

### ❓ PCIe lane nima?

**✅ Javob:** Har lane — mustaqil ikki tomonlama ketma-ket ulanish. Qurilma bir necha lane ishlatadi: x1 (sekin), x4 (NVMe), x16 (GPU). Ko'proq lane = ko'proq bandwidth.

### ❓ Memory bus nima?

**✅ Javob:** CPU'ning memory controller'i va RAM o'rtasidagi maxsus bus. Eng band yo'llardan biri; ko'p kanal (dual/quad-channel) bandwidth'ni oshiradi.

### ❓ Polling va interrupt farqi nima?

**✅ Javob:** Polling — CPU doim qurilmadan "tayyormisan?" deb so'raydi (vaqt behuda). Interrupt — qurilma o'zi tayyor bo'lganda CPU'ni IRQ bilan chaqiradi; CPU shu paytgacha boshqa ish qiladi. Interrupt samaraliroq.

### ❓ Interrupt handler nima?

**✅ Javob:** IRQ kelganda CPU joriy ishni to'xtatib bajaradigan maxsus kod. U hodisaga javob beradi (masalan klaviaturadan belgini o'qiydi), so'ng CPU oldingi ishni davom ettiradi.

### ❓ DMA nima va nega muhim?

**✅ Javob:** Direct Memory Access — maxsus kontroller qurilma va RAM o'rtasida ma'lumotni CPU'siz ko'chiradi. CPU faqat buyruq beradi, keyin boshqa ish qiladi; tugagach IRQ oladi. Muhim, chunki katta I/O'da CPU'ni band qilmaydi.

### ❓ Memory-mapped va port-mapped I/O farqi?

**✅ Javob:** Memory-mapped — qurilma registrlari xotira manzili sifatida ko'rinadi, oddiy load/store bilan kiriladi. Port-mapped — alohida port fazosi, maxsus IN/OUT instruksiyalari. Zamonaviy arxitekturalarda memory-mapped keng tarqalgan.

### ❓ Bus bandwidth nima?

**✅ Javob:** Bus sekundiga qancha ma'lumot uzata olishi (GB/s). Taxminan: kenglik × chastota / 8. Bu tizim tezligini cheklovchi omillardan biri.

### ❓ Bottleneck nima?

**✅ Javob:** Tizimning eng sekin qismi umumiy tezlikni cheklaydi. Masalan, tez GPU'ga sekin PCIe ma'lumot yetkazib berolmasa — bottleneck PCIe'da.

### ❓ SATA va NVMe farqi nima?

**✅ Javob:** Ikkalasi ham SSD ulash uchun. SATA — eski protokol (~600 MB/s cheklov). NVMe — PCIe orqali to'g'ridan-to'g'ri ulanadi, past latency va bir necha barobar tez.

### ❓ Disk'dan fayl o'qish jarayonida bus'lar qanday ishlaydi?

**✅ Javob:** CPU chipset orqali SSD'ga o'qish buyrug'ini yuboradi (control/address bus). SSD DMA orqali ma'lumotni to'g'ridan-to'g'ri RAM'ga ko'chiradi. Tugagach IRQ yuboradi, CPU handler'ni bajaradi — ma'lumot RAM'da tayyor.

---

## Masalalar

> Yechimlar: [solutions/hardware/06-buses-communication.md](../solutions/hardware/06-buses-communication.md)

1. Address, data va control bus'ning har biri xotiradan bitta baytni o'qish jarayonida qanday rol o'ynashini qadamma-qadam yozing.

2. 40-bit address bus qancha xotirani adreslay oladi? Formula bilan hisoblang va natijani GB/TB'da bering.

3. Polling va interrupt'ni taqqoslang. Qaysi holatda polling aslida yaxshiroq bo'lishi mumkin? Bitta amaliy misol keltiring.

4. DMA bo'lmagan tizimda 1 GB faylni disk'dan RAM'ga ko'chirish CPU'ga qanday ta'sir qiladi? DMA buni qanday hal qiladi — diagramma bilan tushuntiring.

5. Nega zamonaviy CPU'larda memory controller va PCIe controller CPU ichiga ko'chirildi? Bu qanday afzallik beradi?

6. GPU x16 slotga, NVMe SSD x4 slotga ulanadi. Nega GPU'ga ko'proq lane kerak? Bandwidth nuqtai nazaridan tushuntiring.

7. O'zingizning ASCII diagrammangizda quyidagilarni bog'lang: CPU, RAM, GPU, NVMe SSD, USB klaviatura, chipset. Har ulanishda qaysi bus/interfeys ishlatilishini belgilang.

8. Memory-mapped va port-mapped I/O o'rtasidagi farqni jadval ko'rinishida yozing (kamida 3 nuqta) va qaysi biri zamonaviy arxitekturalarda ustun ekanini ayting.

9. Bottleneck tushunchasini tushuntiring: tizimda tez CPU, tez RAM, lekin sekin storage bor. Katta fayl bilan ishlaganda umumiy tezlik nimaga bog'liq bo'ladi?

10. SATA SSD va NVMe SSD o'rtasidagi arxitektura farqini tushuntiring. Nega NVMe tezroq — qaysi bus orqali ulanadi va nima uchun?

---

← [Hardware bo'limiga qaytish](./README.md)
