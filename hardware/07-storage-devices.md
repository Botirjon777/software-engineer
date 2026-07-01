# Storage (HDD, SSD, NVMe)

Storage (saqlash qurilmalari) — bu ma'lumotni **doimiy** (persistent) saqlaydigan qurilmalar: HDD, SSD, NVMe. Kompyuter o'chsa ham, ma'lumot yo'qolmaydi. Bu ularni RAM'dan tubdan farqlaydi. Bu bo'limda storage nima, HDD va SSD ichida nima bo'layotgani, SATA va NVMe interfeyslari, IOPS/throughput/latency metrikalari, RAID va — eng muhimi — nega dasturchi (ayniqsa DB bilan ishlovchi) buni bilishi kerakligi haqida gaplashamiz.

**💡 Tushuncha:** RAM — bu "ishchi stol" (tez, lekin o'chsa hamma narsa yo'qoladi). Storage — bu "shkaf" (sekinroq, lekin doimiy). Dastur ishlashi uchun ma'lumot shkafdan stolga (storage → RAM) ko'chiriladi.

## Mundarija

- [Storage nima va nega kerak (persistent vs volatile)](#storage-nima-va-nega-kerak)
- [HDD qanday ishlaydi (mexanik)](#hdd-qanday-ishlaydi)
- [SSD qanday ishlaydi (elektron, NAND flash)](#ssd-qanday-ishlaydi)
- [HDD vs SSD taqqoslash](#hdd-vs-ssd-taqqoslash)
- [SSD ichki tuzilishi (controller, wear leveling, TRIM)](#ssd-ichki-tuzilishi)
- [SATA vs NVMe (interfeys va PCIe)](#sata-vs-nvme)
- [Storage memory hierarchy'da qayerda](#storage-memory-hierarchyda-qayerda)
- [Metrikalar: IOPS, throughput, latency](#metrikalar-iops-throughput-latency)
- [RAID qisqacha (0/1/5/10)](#raid-qisqacha)
- [Storage dasturchi uchun (DB, sequential vs random I/O)](#storage-dasturchi-uchun)
- [Savol-javoblar](#savol-javoblar)
- [Masalalar](#masalalar)

---

## Storage nima va nega kerak

Kompyuterda ma'lumot ikki xil joyda yashaydi:

| Xususiyat | RAM (xotira) | Storage (HDD/SSD) |
|-----------|--------------|-------------------|
| Turi | **Volatile** (uchuvchan) | **Non-volatile / persistent** (doimiy) |
| O'chsa nima bo'ladi | Ma'lumot yo'qoladi | Ma'lumot saqlanadi |
| Tezlik | Juda tez (ns) | Sekinroq (µs–ms) |
| Hajm | Kichik (GB) | Katta (TB) |
| Narx (GB uchun) | Qimmat | Arzon |
| Vazifasi | Ishchi ma'lumot (running) | Doimiy saqlash (fayllar, DB) |

**💡 Tushuncha:** "Persistent" degani — quvvat o'chsa ham ma'lumot qoladi. Shuning uchun operatsion tizim, fayllaringiz, dasturlar va database hammasi storage'da yashaydi, RAM'da emas. Dastur ishga tushganda esa storage'dan RAM'ga yuklanadi.

```
   Quvvat bor          Quvvat O'CHDI
  ┌──────────┐        ┌──────────┐
  │   RAM    │ ─────► │   RAM    │   ← bo'sh (ma'lumot yo'qoldi)
  │ [data]   │        │  [    ]  │
  └──────────┘        └──────────┘
  ┌──────────┐        ┌──────────┐
  │ STORAGE  │ ─────► │ STORAGE  │   ← ma'lumot saqlandi!
  │ [data]   │        │ [data]   │
  └──────────┘        └──────────┘
```

---

## HDD qanday ishlaydi

**HDD (Hard Disk Drive)** — bu **mexanik** qurilma. Ichida aylanuvchi metall disklar (platter) va ular ustida harakatlanuvchi o'qish/yozish kallagi (head) bor. Ma'lumot magnit qatlamga yoziladi.

Asosiy qismlar:

- **Platter** — magnit qoplamali aylanuvchi disk. Ma'lumot shu yerda saqlanadi.
- **Spindle** — platterlarni aylantiruvchi o'q (motor).
- **Read/Write head** — magnit holatni o'qiydigan/yozadigan kallak (aktuator qo'lida).
- **Actuator arm** — kallakni to'g'ri track'ga olib boruvchi qo'l.

```
        ┌─────────────── HDD (yuqoridan) ───────────────┐
        │                                                │
        │      ╭──────────────╮   ← platter (aylanadi)  │
        │     ╱   ╭────────╮    ╲                        │
        │    │   ╱ track    ╲    │                       │
        │    │  │  ╭────╮    │   │◄── Read/Write head    │
        │    │   ╲ sector╱    ╱  │      (actuator arm)   │
        │     ╲   ╰──────╯    ╱                          │
        │      ╰──── spindle ─╯                          │
        │            (motor, RPM)                        │
        └────────────────────────────────────────────────┘
```

HDD tezligiga ta'sir qiluvchi omillar:

- **RPM (Revolutions Per Minute)** — platter daqiqada necha marta aylanadi. Odatda 5400 yoki 7200 RPM. Tezroq aylanish = tezroq o'qish.
- **Seek time** — kallakni kerakli track'ga siljitish vaqti (mexanik harakat). ~5–10 ms.
- **Rotational latency** — kerakli sector kallak ostiga aylanib kelishini kutish. 7200 RPM'da o'rtacha ~4.16 ms.

**⚠️ Ehtiyot bo'l:** HDD'da har bir tasodifiy (random) o'qish uchun kallak jismonan siljishi kerak (seek time + rotational latency). Shu sababli HDD **random I/O**'da juda sekin. Bu SSD'ga o'tishning asosiy sababi.

---

## SSD qanday ishlaydi

**SSD (Solid State Drive)** — bu **elektron** qurilma. Mexanik qism umuman yo'q (aylanadigan disk yo'q, harakatlanadigan kallak yo'q). Ma'lumot **NAND flash** yarim o'tkazgich xotirasida elektr zaryad shaklida saqlanadi.

Ierarxiya (kichikdan kattaga):

- **Cell** — bitta bit(lar)ni saqlaydigan eng kichik birlik (transistor). SLC=1 bit, MLC=2, TLC=3, QLC=4 bit har bir cell'da.
- **Page** — o'qish/yozishning eng kichik birligi (masalan 4–16 KB). O'qish/yozish **page** darajasida bo'ladi.
- **Block** — ko'p page'lardan iborat. O'chirish (erase) faqat **block** darajasida bo'ladi.

```
   SSD (NAND flash)
   ┌──────────────────────────────────┐
   │  BLOCK (o'chirish birligi)        │
   │  ┌────────┬────────┬────────┐     │
   │  │ PAGE   │ PAGE   │ PAGE   │ ... │ ← o'qish/yozish birligi
   │  │┌─┬─┬─┐ │        │        │     │
   │  ││c│c│c│ │        │        │     │ ← cell (bit saqlaydi)
   │  │└─┴─┴─┘ │        │        │     │
   │  └────────┴────────┴────────┘     │
   └──────────────────────────────────┘
```

**💡 Tushuncha:** SSD'da mexanik harakat yo'qligi uchun seek time yo'q. Har qanday joyga kirish deyarli bir xil vaqt oladi — shu sababli SSD **random I/O**'da HDD'dan yuz barobar tezroq.

**⚠️ Ehtiyot bo'l:** NAND flash'ning muhim cheklovi — **page'ga yozishdan oldin, agar unda eski ma'lumot bo'lsa, avval butun block'ni o'chirish kerak**. Yozish page'da, lekin o'chirish faqat block'da. Bu "write amplification" va "garbage collection" muammolarini keltirib chiqaradi (pastda).

---

## HDD vs SSD taqqoslash

| Xususiyat | HDD | SSD |
|-----------|-----|-----|
| Texnologiya | Magnit disk (mexanik) | NAND flash (elektron) |
| Mexanik qism | Bor (platter, head) | Yo'q |
| Random read latency | ~5–10 ms | ~0.05–0.1 ms (~100x tez) |
| Sequential throughput | ~100–200 MB/s | 500 MB/s (SATA) – 7000 MB/s (NVMe) |
| IOPS (random) | ~100–200 | 10 000 – 1 000 000+ |
| Narx (GB uchun) | Arzon | Qimmatroq |
| Chidamlilik (zarba) | Zaif (harakatlanuvchi qism) | Kuchli (harakat yo'q) |
| Shovqin / issiqlik | Bor | Deyarli yo'q |
| Yozish chidamliligi | Cheksizga yaqin | Cheklangan (cell ~1000–100000 marta) |
| Eng yaxshi ish | Katta arxiv, arzon hajm | OS, DB, tez ishlash |

**💡 Tushuncha:** Umumiy qoida — **tezlik kerak bo'lsa SSD, arzon katta hajm kerak bo'lsa HDD**. Ko'p serverlarda ikkalasi ham ishlatiladi: SSD tez ma'lumot uchun, HDD sovuq arxiv uchun.

---

## SSD ichki tuzilishi

SSD shunchaki NAND flash emas — ichida "aqlli" **controller** bor, u murakkab ishlarni bajaradi:

- **Controller** — SSD'ning "miyasi". Yozish/o'qishni boshqaradi, quyidagi algoritmlarni ishlatadi.
- **Wear leveling** — yozishni barcha cell'larga teng taqsimlaydi. Chunki har bir cell cheklangan marta yoziladi; agar bir joyga qayta-qayta yozilsa, u tez o'ladi. Wear leveling yukni yoyib, SSD umrini uzaytiradi.
- **TRIM** — OS SSD'ga "bu bloklar endi kerak emas (fayl o'chirildi)" deb aytadi. Shunda SSD ularni oldindan tozalab, kelajakdagi yozishni tezlashtiradi.
- **Write amplification** — kichik yozish, aslida ancha ko'proq fizik yozishga aylanadi (page yozish uchun block ko'chirib-o'chiriladi). Nisbat qancha past bo'lsa, shuncha yaxshi.
- **Garbage collection** — ishlatilmayotgan page'larni yig'ib, block'larni tozalab, bo'sh joy tayyorlaydi (fon rejimida).

```
   ┌───────────────────────────────────────────┐
   │                 SSD                        │
   │  ┌───────────────┐                         │
   │  │  CONTROLLER   │  ← miya                 │
   │  │  · wear level │                         │
   │  │  · TRIM       │───► NAND flash chiplar  │
   │  │  · garbage    │      [block][block]...  │
   │  │    collection │                         │
   │  └───────────────┘                         │
   │  ┌───────────────┐                         │
   │  │   DRAM cache  │  ← xarita (mapping)     │
   │  └───────────────┘                         │
   └───────────────────────────────────────────┘
```

**⚠️ Ehtiyot bo'l:** SSD to'lib qolsa (masalan 95%+), garbage collection va wear leveling qiyinlashadi — tezlik pasayadi va write amplification oshadi. Shuning uchun SSD'ni doim biroz bo'sh (over-provisioning) qoldirish tavsiya etiladi.

---

## SATA vs NVMe

Bu **interfeys** (qurilma kompyuterga qanday ulanadi va qanday protokolda gaplashadi) haqida. Muhim: SSD SATA orqali ham, NVMe orqali ham bo'lishi mumkin.

- **SATA** — eski interfeys, aslida HDD uchun mo'ljallangan. **AHCI** protokolidan foydalanadi (bitta buyruq navbati, cheklangan parallelizm). Maksimal ~600 MB/s. SSD'ni bu "quvur"ga bog'lasa, SSD to'liq quvvatini ko'rsata olmaydi.
- **NVMe (Non-Volatile Memory express)** — SSD uchun maxsus yaratilgan protokol. To'g'ridan-to'g'ri **PCIe** shinasi orqali ishlaydi. Minglab parallel navbat (queue), har birida minglab buyruq. Shuning uchun juda tez.

**Nega NVMe (PCIe orqali) tez:**

1. **PCIe keng yo'lak** — PCIe ko'p "lane"ga ega, o'tkazuvchanligi SATA'dan bir necha barobar katta.
2. **Ko'p navbat (parallelism)** — AHCI'da 1 navbat × 32 buyruq; NVMe'da 65536 navbat × 65536 buyruq. SSD'ning ichki parallelizmiga mos.
3. **Qisqa yo'l** — CPU/RAM'ga PCIe orqali to'g'ridan-to'g'ri, ortiqcha kontroller qatlami yo'q — latency past.

```
  SATA SSD:   SSD ──► SATA (AHCI) ──► chipset ──► CPU      (~600 MB/s)
                       [1 navbat]

  NVMe SSD:   SSD ──► PCIe (NVMe) ──────────────► CPU      (3000–7000 MB/s)
                       [minglab navbat, parallel]
```

**💡 Tushuncha:** "SSD" bu xotira turi, "SATA/NVMe" bu ulanish usuli. NVMe SSD = tez xotira + tez quvur. SATA SSD = tez xotira + tor quvur.

---

## Storage memory hierarchy'da qayerda

Storage — xotira ierarxiyasining **eng past va eng sekin, lekin eng katta va doimiy** qatlami. (Batafsil: [Xotira Ierarxiyasi](./03-memory-hierarchy.md).)

```
        TEZ, kichik, qimmat, VOLATILE
   ┌──────────────────────────────┐
   │  Registers        ~1 ns      │
   │  L1 / L2 / L3 cache  ~1–40 ns│
   │  RAM (DRAM)       ~100 ns     │
   ├──────────────────────────────┤ ← bu chegara: volatile / persistent
   │  SSD (NVMe)       ~50 µs      │
   │  SSD (SATA)       ~100 µs     │  PERSISTENT (doimiy)
   │  HDD              ~10 ms      │
   │  Network / tarmoq  ~sekundlar │
   └──────────────────────────────┘
        SEKIN, katta, arzon, doimiy
```

**💡 Tushuncha:** Ierarxiyada pastga tushgani sari: sekinroq, arzonroq, kattaroq. RAM va SSD orasidagi tezlik farqi ~1000x. Shuning uchun tez-tez kerak bo'ladigan ma'lumot RAM'da (cache) saqlanadi — bu "caching"ning butun mohiyati.

---

## Metrikalar: IOPS, throughput, latency

Storage tezligini o'lchashda uch xil metrika bor — ularni chalkashtirmaslik muhim:

| Metrika | Ma'nosi | Birlik | Qachon muhim |
|---------|---------|--------|--------------|
| **Latency** | Bitta operatsiya qancha vaqt oladi | ms / µs | Ta'sirchan (responsive) tizimlar |
| **IOPS** | Sekundiga necha operatsiya | operatsiya/s | Ko'p kichik so'rovlar (DB, OLTP) |
| **Throughput** | Sekundiga necha bayt | MB/s, GB/s | Katta fayllar (video, backup) |

**💡 Tushuncha:** Analogiya — do'kon kassasi. **Latency** = bitta mijozga xizmat vaqti. **IOPS** = daqiqada necha mijozga xizmat qilinadi. **Throughput** = daqiqada sotilgan tovar hajmi. Ular bog'liq, lekin bir xil emas.

**⚠️ Ehtiyot bo'l:** "SSD 7 GB/s" degani — bu **sequential throughput** (ketma-ket katta bloklar). Real DB yuki ko'pincha **random 4KB** so'rovlar — u yerda IOPS va latency muhimroq, va raqamlar boshqacha bo'ladi. Marketing raqamiga aldanmang.

---

## RAID qisqacha

**RAID (Redundant Array of Independent Disks)** — bir nechta diskni birlashtirib, tezlik va/yoki ishonchlilik (redundancy) olish usuli.

| RAID | Tamoyil | Redundancy | Tezlik | Kamchilik |
|------|---------|------------|--------|-----------|
| **RAID 0** | Striping (ma'lumotni disklarga bo'lish) | Yo'q | Yuqori | 1 disk o'lsa — hammasi yo'qoladi |
| **RAID 1** | Mirroring (nusxa) | Bor (100% dublikat) | O'qish tez | Hajmning yarmi "isrof" |
| **RAID 5** | Striping + parity | Bor (1 disk o'lishi mumkin) | Yaxshi | Yozish sekinroq (parity), min 3 disk |
| **RAID 10** | Mirror + stripe (1+0) | Bor | Yuqori | Qimmat (hajmning yarmi) |

```
  RAID 0 (tezlik):     A1 A3     A2 A4      ← bo'lingan, himoya yo'q
                      [disk1]  [disk2]

  RAID 1 (himoya):     A1 A2     A1 A2      ← nusxa
                      [disk1]  [disk2]

  RAID 5 (balans):     A1 A2 Ap    B1 Bp B3   Cp C2 C3   ← parity aylanadi
                      [disk1]     [disk2]    [disk3]
```

**💡 Tushuncha:** RAID 0 = tezlik, himoya yo'q. RAID 1 = himoya, tezlik oddiy. RAID 5 = ikkalasidan biroz. RAID 10 = eng yaxshi (tez + himoya), lekin qimmat.

**⚠️ Ehtiyot bo'l:** RAID **backup emas!** RAID disk nosozligidan himoya qiladi, lekin fayl o'chirilsa, virus yuqsa yoki xato yozilsa — u barcha disklarga tarqaladi. Backup alohida kerak.

---

## Storage dasturchi uchun

Dasturchi uchun eng muhim amaliy xulosalar:

**1. Nega DB SSD'da tez.** Database ko'pincha **random** kichik o'qish/yozish qiladi (index bo'ylab sakrash, alohida yozuvlarni topish). HDD'da har bir random kirish = seek time + rotational latency (~10 ms). SSD'da seek yo'q (~0.1 ms). Shuning uchun DB SSD'da 50–100x tezroq ishlaydi.

**2. Sequential vs Random I/O.** Bu eng muhim tushuncha:

- **Sequential** — ma'lumot ketma-ket, yonma-yon o'qiladi/yoziladi (log fayl, katta fayl ko'chirish). Tez.
- **Random** — turli joylardan sakrab o'qiladi (index lookup). Sekinroq, ayniqsa HDD'da.

```
  Sequential:  [A][B][C][D][E]   ← ketma-ket, kallak siljimaydi (HDD)
               ─────────────►

  Random:      [C]...[A]...[E]...[B]   ← sakrash, har safar seek (HDD)
                ↗    ↘   ↗    ↘
```

**💡 Tushuncha:** Shu sababli DB'lar **sequential yozishni** afzal ko'radi. Masalan, ko'p DB'lar (LSM-tree, WAL — Write-Ahead Log) yozishni ketma-ket log shaklida amalga oshiradi, keyin fon rejimida tartiblaydi. Chunki hatto SSD'da ham sequential > random.

**3. Amaliy maslahatlar:**

- Log va WAL — sequential yozish uchun mo'ljallangan; ularni tez diskka qo'ying.
- Katta fayllarni bloklab (batch) o'qing — har bir kichik so'rovda latency to'lanadi.
- I/O — ko'pincha eng sekin bosqich; profiling'da disk kutishini (I/O wait) tekshiring.
- Ma'lumotni RAM'da cache qiling (Redis, buffer pool) — storage'ga bormaslik eng tez "o'qish".

**⚠️ Ehtiyot bo'l:** "SSD tez" degani random I/O ni cheksiz qilsa bo'ladi degani emas. Har bir I/O operatsiyasi controller, navbat va syscall xarajatiga ega. 1 million kichik so'rovni 1 katta so'rovga birlashtirish har doim tezroq.

---

## Savol-javoblar

### ❓ Storage va RAM o'rtasidagi asosiy farq nima?

**✅ Javob:** RAM **volatile** (uchuvchan) — quvvat o'chsa ma'lumot yo'qoladi, lekin juda tez (ns). Storage **persistent / non-volatile** (doimiy) — quvvat o'chsa ham ma'lumot saqlanadi, lekin sekinroq (µs–ms). RAM ishchi ma'lumot uchun, storage doimiy saqlash uchun. Dastur ishga tushganda storage'dan RAM'ga yuklanadi.

### ❓ HDD'da seek time va rotational latency nima?

**✅ Javob:** Ikkalasi ham HDD'ning mexanik kechikishi. **Seek time** — o'qish kallagini (head) kerakli track'ga jismonan siljitish vaqti (~5–10 ms). **Rotational latency** — kerakli sector kallak ostiga aylanib kelishini kutish (7200 RPM'da o'rtacha ~4.16 ms). Bu ikkisi HDD'ni random I/O'da sekin qiladi.

### ❓ Nega SSD HDD'dan tezroq?

**✅ Javob:** SSD'da mexanik qism yo'q — aylanadigan platter va harakatlanadigan kallak yo'q. Ma'lumot NAND flash'da elektron shaklda saqlanadi va istalgan joyga deyarli bir xil (juda qisqa) vaqtda kiriladi. Seek time va rotational latency bo'lmagani uchun, ayniqsa **random I/O**'da SSD 50–100x tezroq.

### ❓ NAND flash'da cell, page va block nima?

**✅ Javob:** **Cell** — bir yoki bir necha bit saqlaydigan eng kichik birlik. **Page** — o'qish/yozishning eng kichik birligi (masalan 4–16 KB). **Block** — ko'p page'dan iborat va **o'chirishning** birligi. Muhim nomutanosiblik: o'qish/yozish page'da, lekin o'chirish faqat butun block'da bo'ladi.

### ❓ Write amplification nima va nega yomon?

**✅ Javob:** Kichik mantiqiy yozish, aslida ancha ko'proq fizik yozishga aylanishi. Sabab: page'ni qayta yozish uchun SSD avval butun block'ni ko'chirib, o'chirishi kerak. Natijada 4KB yozish, masalan, 512KB fizik yozishga aylanishi mumkin. Bu SSD'ni tezroq eskirtiradi va sekinlashtiradi. Wear leveling va garbage collection buni kamaytirishga harakat qiladi.

### ❓ TRIM nima qiladi?

**✅ Javob:** TRIM — OS SSD'ga "bu bloklardagi ma'lumot endi kerak emas (fayl o'chirildi)" deb xabar beruvchi buyruq. SSD bu bloklarni oldindan tozalab qo'yadi. Natijada kelajakdagi yozish tezroq bo'ladi (avval o'chirish kutilmaydi) va write amplification kamayadi. TRIM'siz SSD vaqt o'tishi bilan sekinlashadi.

### ❓ Wear leveling nima uchun kerak?

**✅ Javob:** NAND cell cheklangan marta yozishga chidaydi (~1000–100000). Agar dastur bir joyga qayta-qayta yozsa, o'sha cell'lar tez o'ladi. Wear leveling — controller yozishni barcha bloklarga teng taqsimlaydi, shunda hech bir blok boshqalardan tezroq eskirmaydi. Bu SSD umrini sezilarli uzaytiradi.

### ❓ SATA va NVMe farqi nima?

**✅ Javob:** Ikkalasi ham interfeys (ulanish usuli). **SATA** — eski, HDD uchun mo'ljallangan, AHCI protokoli, ~600 MB/s, bitta buyruq navbati. **NVMe** — SSD uchun maxsus, PCIe shinasi orqali, minglab parallel navbat, 3000–7000 MB/s. NVMe SSD'ning ichki parallelizmini to'liq ishlata oladi, shuning uchun ancha tez.

### ❓ Nega NVMe SATA'dan tez?

**✅ Javob:** Uch sabab: (1) NVMe **PCIe** orqali ishlaydi — u SATA'dan ancha keng o'tkazuvchanlikka ega. (2) NVMe minglab parallel navbatni qo'llab-quvvatlaydi (SATA/AHCI faqat 1 ta), bu SSD'ning parallel NAND chiplariga mos. (3) CPU'ga to'g'ridan-to'g'ri yo'l, ortiqcha kontroller qatlami yo'q — latency past.

### ❓ IOPS, throughput va latency farqi nima?

**✅ Javob:** **Latency** — bitta operatsiya qancha vaqt oladi (ms/µs). **IOPS** — sekundiga necha operatsiya bajariladi (kichik random so'rovlar uchun muhim, masalan DB). **Throughput** — sekundiga necha bayt o'tkaziladi (MB/s; katta fayllar uchun muhim). Bir SSD katta throughput'ga ega bo'lib, lekin kichik random so'rovda past IOPS ko'rsatishi mumkin — kontekst muhim.

### ❓ RAID 0 va RAID 1 farqi nima?

**✅ Javob:** **RAID 0** (striping) ma'lumotni disklarga bo'ladi — tezlik oshadi, lekin **himoya yo'q**: bitta disk o'lsa, hamma ma'lumot yo'qoladi. **RAID 1** (mirroring) ma'lumotni nusxalaydi — bitta disk o'lsa ikkinchisi ishlaydi (himoya bor), lekin foydali hajm yarmiga tushadi va tezlik oddiy. RAID 0 = tezlik, RAID 1 = ishonchlilik.

### ❓ RAID backup'ni almashtiradimi?

**✅ Javob:** Yo'q. RAID faqat **disk nosozligidan** himoya qiladi (RAID 1/5/10). Lekin fayl xato o'chirilsa, virus yuqsa yoki dastur noto'g'ri yozsa — bu o'zgarish barcha disklarga (RAID 1'da nusxaga ham) tarqaladi. Haqiqiy backup — alohida joyda, alohida nusxa. RAID ≠ backup.

### ❓ Nega database SSD'da tezroq ishlaydi?

**✅ Javob:** DB ko'pincha **random** kichik o'qish/yozish qiladi (index bo'ylab sakrash, alohida yozuvlar). HDD'da har bir random kirish mexanik seek + rotational latency talab qiladi (~10 ms). SSD'da mexanik harakat yo'q (~0.1 ms). Natijada DB SSD'da 50–100x tezroq — ayniqsa ko'p kichik so'rovlarda (OLTP).

### ❓ Sequential va random I/O farqi nima va nega muhim?

**✅ Javob:** **Sequential** — ma'lumot ketma-ket yonma-yon o'qiladi/yoziladi (log, katta fayl) — tez. **Random** — turli joylardan sakrab o'qiladi (index lookup) — sekinroq, ayniqsa HDD'da (har safar seek). Muhim, chunki hatto SSD'da ham sequential random'dan tezroq. Shuning uchun DB'lar (WAL, LSM-tree) yozishni sequential log shaklida qiladi.

### ❓ SSD to'lib qolsa nima bo'ladi?

**✅ Javob:** SSD deyarli to'lganda (95%+), garbage collection uchun bo'sh block topish qiyinlashadi. Natijada write amplification oshadi, tezlik pasayadi va cell'lar tezroq eskirodi. Shuning uchun SSD'da har doim biroz bo'sh joy (over-provisioning) qoldirish tavsiya etiladi — odatda ishlab chiqaruvchi buni avtomatik qiladi, lekin foydalanuvchi ham diskni oxirigacha to'ldirmagani ma'qul.

### ❓ Storage memory hierarchy'da qayerda turadi?

**✅ Javob:** Eng past qatlamda — eng sekin, lekin eng katta va yagona **doimiy** (persistent) daraja. Yuqoridan pastga: registers → cache → RAM → **SSD → HDD** → tarmoq. RAM bilan SSD orasida "volatile/persistent" chegarasi bor va tezlik farqi ~1000x. Shuning uchun tez-tez kerak ma'lumot RAM'da cache qilinadi.

---

## Masalalar

> Yechimlar: [`solutions/hardware/07-storage-devices.md`](../solutions/hardware/07-storage-devices.md)

**1. RAM vs Storage.** Do'stingiz "kompyuterimda 16 GB xotira bor, nega yana 512 GB SSD kerak?" deb so'radi. Volatile va persistent tushunchalari orqali tushuntiring: nega ikkalasi ham kerak va biri ikkinchisini almashtira olmaydi.

**2. HDD latency hisobi.** 7200 RPM HDD berilgan. (a) Bitta to'liq aylanish necha millisekund? (b) O'rtacha rotational latency qancha (aylanish vaqtining yarmi)? (c) Agar seek time ~9 ms bo'lsa, bitta random o'qishning taxminiy umumiy latency'sini baholang.

**3. HDD vs SSD tanlash.** Uch stsenariy uchun HDD yoki SSD tanlang va sababini yozing: (a) 50 TB video arxivi (kamdan-kam o'qiladi), (b) yuqori yukli e-commerce DB, (c) noutbukingizning asosiy OS diski. 

**4. Write amplification zanjiri.** Dastur 4 KB yozadi, lekin SSD page 16 KB, block esa 4 MB. Nega bu kichik yozish katta fizik ishga aylanishi mumkinligini o'z so'zlaringiz bilan qadamma-qadam tushuntiring. Wear leveling va garbage collection bu jarayonga qanday aralashadi?

**5. SATA vs NVMe.** Sizga bir xil NAND flash'li ikki SSD berildi: biri SATA, biri NVMe. (a) Katta video faylni ko'chirishda farq sezilarli bo'ladimi? (b) Minglab kichik parallel so'rovli DB yukida-chi? Har bir holatda nega, deb izohlang.

**6. Metrikani to'g'ri tanlash.** Quyidagi har bir tizim uchun eng muhim metrikani (latency / IOPS / throughput) tanlang va asoslang: (a) 4K video tahrirlash stansiyasi, (b) bank tranzaksiya tizimi (ko'p kichik yozish), (c) real-time o'yin serveri (past javob vaqti kerak).

**7. RAID rejasi.** Kompaniyada 4 ta bir xil disk bor. Muhim mijoz ma'lumotlari saqlanadi — himoya ham, oqilona tezlik ham kerak. RAID 0, 1, 5, 10 dan qaysi birini tanlaysiz? Nega qolganlari mos emas? RAID nega backup'ni almashtirmasligini ham eslatib qo'ying.

**8. Sequential vs Random.** Bir jadval ustida ikki so'rov: (a) `SELECT * FROM logs ORDER BY id` (butun jadval ketma-ket), (b) 10000 ta tasodifiy `id` bo'yicha bittalab lookup. Qaysi biri sequential, qaysi biri random I/O? HDD va SSD'da tezlik farqi qanday bo'lishini taxmin qiling va nega DB'lar sequential yozishni afzal ko'rishini bog'lang.

**9. I/O batching.** Ilovangiz bir sikllda 100000 marta 100 baytdan alohida disk yozuvini bajaradi va sekin. Muammoni storage metrikasi (IOPS/latency) tili bilan tushuntiring va tezlashtirishning kamida ikki usulini taklif qiling.

**10. SSD umri.** TLC SSD har bir cell'ni ~1000 marta yozishga chidaydi. Nega bir joyga qayta-qayta yozadigan dastur (masalan, doimiy yangilanadigan bitta hisoblagich fayli) SSD uchun xavfli va wear leveling bu muammoni qanday yumshatishini tushuntiring.

---

← [Hardware bo'limiga qaytish](./README.md)
