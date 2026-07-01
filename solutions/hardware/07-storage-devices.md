# Storage (HDD, SSD, NVMe) — Yechimlar

Bu fayl [`hardware/07-storage-devices.md`](../../hardware/07-storage-devices.md) dagi masalalarning yechimlari. Avval o'zingiz yechishga harakat qiling, keyin bu yerga qarang.

---

**1. RAM vs Storage.**

RAM va Storage ikki xil vazifa bajaradi va biri ikkinchisini almashtira olmaydi:

- **RAM (16 GB)** — **volatile**: quvvat o'chsa hamma narsa yo'qoladi. Juda tez (ns). Bu — protsessor ayni damda ishlatayotgan "ishchi stol": ochiq dasturlar, ochiq fayllar shu yerda turadi.
- **SSD (512 GB)** — **persistent**: quvvat o'chsa ham saqlanadi. Sekinroq (µs), lekin katta va doimiy. Bu — OS, dasturlar, fayllar va DB doimiy yashaydigan "shkaf".

Nega ikkalasi kerak: kompyuter o'chib yonganda hamma narsa qayerdan keladi? RAM bo'sh — demak, storage'dan. Agar faqat RAM bo'lsa, har o'chirishda OS ham, fayllar ham yo'qolardi. Agar faqat storage bo'lsa, protsessor har amal uchun sekin diskga borishi kerak bo'lardi — kompyuter juda sekin ishlardi. Shuning uchun: **storage — doimiy uy, RAM — tez ishchi joy**. Dastur ishga tushganda storage'dan RAM'ga ko'chiriladi.

---

**2. HDD latency hisobi.**

- **(a) To'liq aylanish vaqti.** 7200 RPM = daqiqada 7200 aylanish = sekundiga 7200/60 = 120 aylanish. Bitta aylanish = 1/120 s = **8.33 ms**.
- **(b) O'rtacha rotational latency.** Kerakli sector o'rtacha yarim aylanishda keladi: 8.33 / 2 = **~4.16 ms**.
- **(c) Umumiy random read latency.** seek time + rotational latency (+ozgina transfer) ≈ 9 ms + 4.16 ms ≈ **~13 ms**.

Xulosa: bitta random o'qish ~13 ms. Sekundiga ~1000/13 ≈ 77 ta random o'qish (~77 IOPS). Bu SSD'ning ~100000+ IOPS'iga nisbatan juda kam — shu HDD'ning random I/O'dagi zaifligi.

---

**3. HDD vs SSD tanlash.**

- **(a) 50 TB video arxivi (kamdan-kam o'qiladi) → HDD.** Katta hajm, arzon narx muhim; tezlik va latency muhim emas (ko'pincha sequential va kam kirish). SSD bu hajmda juda qimmat bo'lardi.
- **(b) Yuqori yukli e-commerce DB → SSD (imkoni bo'lsa NVMe).** DB ko'p random kichik I/O qiladi; HDD'ning seek time'i bu yerda halokatli. IOPS va past latency hal qiluvchi.
- **(c) Noutbuk OS diski → SSD.** OS va dasturlar tez yuklanishi, tizim ravon ishlashi kerak; noutbuk ko'chib yuradi (zarba) — SSD'da mexanik qism yo'q, chidamli. Bugungi kunda deyarli har doim SSD.

---

**4. Write amplification zanjiri.**

Qadamma-qadam:

1. Dastur atigi **4 KB** yozmoqchi, lekin u tegadigan **page 16 KB**. Page — yozishning minimal birligi, shuning uchun kamida butun 16 KB page bilan ishlash kerak.
2. Agar bu page allaqachon ma'lumot bilan to'lgan bo'lsa, uni **joyida qayta yozib bo'lmaydi** — NAND'da avval o'chirish kerak. Lekin o'chirish faqat **block (4 MB)** darajasida bo'ladi.
3. Shuning uchun controller: butun 4 MB block'dagi hali kerakli page'larni boshqa bo'sh block'ga **ko'chiradi**, eski block'ni **o'chiradi**, so'ng yangi ma'lumotni yozadi.
4. Natija: 4 KB mantiqiy yozish → megabaytlab fizik o'qish/ko'chirish/o'chirish/yozishga aylandi. Bu **write amplification**.

Aralashuv:
- **Garbage collection** — fon rejimida ishlatilmayotgan page'larni yig'ib, block'larni oldindan tozalab, bo'sh block tayyorlaydi. Shunda yozish paytida to'liq ko'chirish-o'chirish sikli kutilmaydi.
- **Wear leveling** — bu yozishlarni turli block'larga taqsimlaydi, shunda hech bir block boshqasidan tezroq eskirmaydi. U write amplification'ni to'g'ridan-to'g'ri kamaytirmaydi, lekin uning zararini (bir joyning tez o'lishini) tekislaydi.

---

**5. SATA vs NVMe.**

- **(a) Katta video fayl (sequential).** Ha, farq sezilarli. SATA ~600 MB/s bilan cheklangan; NVMe 3000–7000 MB/s bera oladi. Katta sequential o'tkazishda NVMe bir necha barobar tez tugatadi. (Garchi HDD emas, SSD bo'lgani uchun ikkalasi ham HDD'dan tez.)
- **(b) Minglab kichik parallel so'rov (DB).** Bu yerda NVMe afzalligi yanada kattaroq. SATA/AHCI faqat **bitta buyruq navbati** — parallel so'rovlar navbatda kutadi. NVMe **minglab parallel navbat**ni qo'llab, SSD'ning ko'p NAND chipini bir vaqtda ishlata oladi. Natijada IOPS va latency SATA'ga nisbatan sezilarli yaxshi.

Xulosa: bir xil NAND bo'lsa ham, interfeys (quvur) tezlikni cheklaydi. NVMe SSD'ning ichki parallelizmini ochib beradi.

---

**6. Metrikani to'g'ri tanlash.**

- **(a) 4K video tahrirlash → Throughput (MB/s).** Katta ketma-ket fayllar o'qiladi/yoziladi; muhimi — sekundiga qancha bayt o'tishi.
- **(b) Bank tranzaksiya tizimi → IOPS.** Ko'p kichik, tez-tez yozishlar; muhimi — sekundiga qancha operatsiya bajarilishi.
- **(c) Real-time o'yin serveri → Latency.** Har bir so'rov imkon qadar tez javob berishi kerak; muhimi — bitta operatsiyaning vaqti, o'rtacha o'tkazuvchanlik emas.

Eslatma: metrikalar bog'liq, lekin tizimning "og'riq nuqtasi" qaysi ekaniga qarab birini asosiy qilib olamiz.

---

**7. RAID rejasi.**

**Tanlov: RAID 10** (4 disk uchun ideal).

Nega:
- **RAID 10** — ikki mirror juftini stripe qiladi: ham himoya (har juftda nusxa bor), ham tezlik (stripe). 4 disk uchun mukammal moslik. Muhim ma'lumot uchun eng ishonchli tanlov.
- **RAID 0** — mos emas: tez, lekin **himoya yo'q**. Muhim mijoz ma'lumoti uchun juda xavfli (1 disk = hamma yo'q).
- **RAID 1** — himoya bor, lekin 4 diskni to'liq ishlatmaydi va RAID 10'chalik tezlik bermaydi.
- **RAID 5** — himoya va hajm samaradorligi yaxshi, lekin yozish parity hisobi tufayli sekinroq va katta disklarda tiklanish (rebuild) xavfli/sekin. RAID 10 tez va sodda.

**Muhim eslatma:** RAID **backup emas**. RAID disk **nosozligidan** himoya qiladi, lekin xato o'chirish, virus yoki buzilgan yozuv barcha disklarga (nusxaga ham) tarqaladi. Alohida backup baribir kerak.

---

**8. Sequential vs Random.**

- **(a) `SELECT * FROM logs ORDER BY id`** — butun jadval tartib bilan o'qiladi → **sequential I/O**.
- **(b) 10000 ta tasodifiy `id` bo'yicha bittalab lookup** — turli joylardan sakrab o'qiladi → **random I/O**.

Tezlik farqi:
- **HDD'da:** (a) tez — kallak deyarli siljimaydi. (b) juda sekin — har lookup uchun seek + rotational latency (~13 ms × 10000 ≈ ~130 s). Farq o'nlab-yuzlab barobar.
- **SSD'da:** (b) HDD'ga qaraganda juda tez (seek yo'q), lekin baribir (a) sequential'dan biroz sekinroq — chunki har operatsiyada controller/navbat xarajati bor.

Bog'liqlik: har ikki qurilmada ham sequential > random. Shuning uchun DB'lar (WAL, LSM-tree) yozishni ketma-ket **log** shaklida qiladi, keyin fon rejimida tartiblaydi — sequential yozish tezroq va disk uchun samarali.

---

**9. I/O batching.**

**Muammo (metrika tili bilan):** 100000 ta alohida kichik yozuv = 100000 ta alohida I/O operatsiyasi. Har biri **latency** (syscall + controller + navbat) to'laydi va **IOPS** limitiga uriladi. Umumiy bayt kichik (100000 × 100 B = 10 MB) bo'lsa ham, sekin — chunki muammo throughput emas, balki **operatsiyalar soni va har biridagi latency**.

Tezlashtirish usullari:
1. **Batching** — 100000 ta kichik yozuvni bitta (yoki bir nechta) katta sequential yozuvga birlashtirish. Bir I/O = bir latency.
2. **Buffering** — RAM'da to'plab, keyin katta blok bilan yozish (masalan buffered writer).
3. **Asenkron / parallel I/O** — NVMe'da ko'p so'rovni parallel navbatga qo'yish (bitta-bitta kutmaslik).
4. **Sequential qilib yozish** — random yozishdan qochib, log/ketma-ket forma tanlash.

Asosiy g'oya: **ko'p kichik I/O'ni kamroq katta I/O'ga aylantirish**.

---

**10. SSD umri.**

**Nega xavfli:** TLC cell ~1000 marta yozishga chidaydi. Agar dastur **bitta aniq faylga (masalan hisoblagich)** doimiy qayta-qayta yozsa, va agar shu bir joydagi cell'lar qayta-qayta ishlatilsa, ular chidamlilik limitiga tez yetadi. O'sha cell'lar "o'lgach", SSD sig'imi kamayadi yoki xatolar boshlanadi — SSD umri qisqaradi. Ustiga, har yozish write amplification tufayli fizik jihatdan bittadan ko'proq bo'lishi mumkin.

**Wear leveling qanday yumshatadi:** Controller mantiqiy manzilni (fayl doim "bir joyda" ko'ringan) fizik NAND'dagi **turli block'larga** xaritalaydi. Ya'ni dastur "bitta faylga" yozgandek ko'rsa ham, controller har yozishni fizik jihatdan **boshqa, kam ishlatilgan block'ga** joylashtiradi. Shunday qilib yozish yuki butun disk bo'ylab teng taqsimlanadi va hech bir cell boshqalardan tezroq eskirmaydi — SSD umri sezilarli uzayadi.

(Amaliy maslahat: baribir shunday "issiq" fayllarni RAM'da buferlab, kamroq flush qilish disk uchun foydali.)

---

← [Hardware bo'limiga qaytish](../../hardware/README.md)
