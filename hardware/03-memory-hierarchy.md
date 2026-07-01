# Xotira Ierarxiyasi

Kompyuterda xotira bir xil emas. Bir tomonda protsessorga eng yaqin, ko'z ochib yumguncha ishlaydigan, ammo juda kichik va qimmat xotira bor. Boshqa tomonda esa terabaytlab ma'lumot sig'adigan, arzon, lekin sekin disk bor. Bu ikki chekka orasida bir necha daraja (level) joylashgan — buni **memory hierarchy** (xotira ierarxiyasi) deb ataymiz.

Bu hujjatda nega bunday ko'p darajali tuzilma kerakligini, har bir darajaning tezligi va hajmini, `cache` qanday ishlashini va — eng muhimi — dasturchi sifatida bu bilimdan kodni tezlashtirish uchun qanday foydalanishni ko'rib chiqamiz.

## Mundarija

- [Nega bir necha darajali xotira?](#nega-bir-necha-darajali-xotira)
- [Ierarxiya piramidasi](#ierarxiya-piramidasi)
- [Har darajaning tezligi va hajmi](#har-darajaning-tezligi-va-hajmi)
- [Cache qanday ishlaydi](#cache-qanday-ishlaydi)
- [Locality: temporal va spatial](#locality-temporal-va-spatial)
- [Cache-friendly kod](#cache-friendly-kod)
- [RAM: DRAM vs SRAM](#ram-dram-vs-sram)
- [Virtual memory va paging](#virtual-memory-va-paging)
- [Memory-mapped va sequential access](#memory-mapped-va-sequential-access)
- [Dasturchi uchun amaliy xulosalar](#dasturchi-uchun-amaliy-xulosalar)
- [Savol-javob](#savol-javob)
- [Masalalar](#masalalar)

## Nega bir necha darajali xotira?

Ideal dunyoda bizga cheksiz hajmli, cheksiz tez va tekin xotira kerak edi. Lekin fizika va iqtisod bunga yo'l qo'ymaydi. Uch xususiyat bir-biriga qarama-qarshi turadi:

**💡 Tushuncha:** Xotira dizaynida uchta narsa doim bir-biriga zid — **tezlik (speed)**, **hajm (capacity)** va **narx (cost)**. Tez xotira qimmat va kichik bo'ladi; arzon xotira sekin bo'ladi. Bularning barchasini bir vaqtda olib bo'lmaydi — bu **trade-off**.

Yechim quyidagicha: biz **oz miqdorda tez xotirani** protsessorga yaqin qo'yamiz va **ko'p miqdorda sekin xotirani** uzoqroqda saqlaymiz. Agar dastur ishlatayotgan ma'lumotning aksariyati tez xotirada bo'lsa (bu real hayotda ko'pincha shunday — pastda `locality` haqida o'qing), tizim o'rtacha hisobda deyarli tez xotira tezligida, lekin katta hajm narxida ishlaydi.

```text
Tez + kichik + qimmat  <--->  Sekin + katta + arzon
        registers                         HDD / tarmoq
```

## Ierarxiya piramidasi

Xotira darajalarini piramida shaklida tasavvur qiling. Yuqoriga chiqqan sari — tezroq, kichikroq, qimmatroq. Pastga tushgan sari — sekinroq, kattaroq, arzonroq.

```text
                 ^  tezroq, kichikroq, qimmatroq
                / \
               /   \      REGISTERS       (~1 KB,   <1 ns)
              /-----\
             /       \    L1 CACHE         (~64 KB,  ~1 ns)
            /---------\
           /           \  L2 CACHE         (~512 KB, ~4 ns)
          /-------------\
         /               \ L3 CACHE        (~8-32 MB, ~15 ns)
        /-----------------\
       /                   \ RAM (DRAM)     (~16-64 GB, ~100 ns)
      /---------------------\
     /                       \ SSD          (~1 TB,   ~100 us)
    /-------------------------\
   /                           \ HDD        (~4 TB,   ~10 ms)
  /-----------------------------\
 /                               \ TARMOQ    (~cheksiz, ~10-100+ ms)
+---------------------------------+
                 v  sekinroq, kattaroq, arzonroq
```

Ma'lumot yuqoriga ("protsessorga yaqin") ko'chsa — tez ishlaydi. Har bir daraja o'zidan pastdagi darajaning "keshi" (cache) vazifasini bajaradi: RAM — diskning keshi, L1 — L2 ning keshi va hokazo.

## Har darajaning tezligi va hajmi

Latency raqamlarini his qilish uchun jadval. Raqamlar taxminiy (protsessorga qarab o'zgaradi), lekin **nisbatlar** muhim — ular yodda qolishi kerak.

| Daraja | Odatiy hajm | Latency (taxminiy) | Volatile? | Izoh |
|---|---|---|---|---|
| Registers | ~1-4 KB | < 1 ns (~0.3 ns) | ha | CPU ichida, ALU'ga to'g'ridan-to'g'ri ulangan |
| L1 cache | 32-64 KB | ~1 ns (~4 tsikl) | ha | Har core'da alohida (odatda) |
| L2 cache | 256 KB - 1 MB | ~4 ns (~12 tsikl) | ha | Har core'da yoki juftlik uchun |
| L3 cache | 8-32 MB | ~15 ns (~40 tsikl) | ha | Core'lar o'rtasida umumiy (shared) |
| RAM (DRAM) | 8-128 GB | ~100 ns | ha | Asosiy xotira, quvvat o'chsa yo'qoladi |
| SSD | 256 GB - 4 TB | ~100 us (~100000 ns) | yo'q | Doimiy saqlash (persistent) |
| HDD | 1-16 TB | ~10 ms (~10000000 ns) | yo'q | Mexanik, aylanuvchi disk |
| Tarmoq (network) | cheksiz | 10-100+ ms | — | Boshqa serverga so'rov |

**💡 Tushuncha:** Miqyosni tasavvur qilish uchun latency'larni "inson vaqti"ga o'girib ko'ring. Agar L1 cache'ga murojaat = **1 soniya** bo'lsa, unda RAM'ga murojaat ~1.5-2 daqiqa, SSD ~1 kun, HDD ~oy-yillar, tarmoq esa ~o'n yillar kabi tuyuladi. Shuning uchun diskka yoki tarmoqqa borishdan iloji boricha qochish kerak.

**⚠️ Ehtiyot bo'l:** Bu raqamlar aniq emas — ular "buyruq tartibi" (order of magnitude) uchun. Real qiymatlar CPU modeli, DRAM turi (DDR4/DDR5), SSD interfeysi (SATA vs NVMe) va hokazoga bog'liq. Muhimi: har daraja o'zidan yuqoridagidan ~10x yoki undan ko'proq sekinroq.

## Cache qanday ishlaydi

`Cache` — protsessor bilan RAM o'rtasidagi kichik, tez SRAM xotira. Uning maqsadi: RAM'ga borishlar sonini kamaytirish.

CPU biror manzildagi ma'lumotni so'raganda:

```text
CPU manzil so'raydi
        |
        v
   L1'da bormi? --ha--> HIT: darhol qaytar (~1 ns)
        | yo'q
        v
   L2'da bormi? --ha--> qaytar (~4 ns), L1'ga ko'chir
        | yo'q
        v
   L3'da bormi? --ha--> qaytar (~15 ns)
        | yo'q
        v
   RAM'dan o'qi (~100 ns), keshlarga ko'chir  <-- MISS (qimmat!)
```

- **Cache hit** — so'ralgan ma'lumot cache'da bor edi. Tez.
- **Cache miss** — cache'da yo'q edi, pastroq darajaga borish kerak bo'ldi. Sekin.

**💡 Tushuncha:** Cache RAM'dan ma'lumotni **bittalab bayt** emas, balki **cache line** (odatda **64 bayt**) blokida oladi. Siz bitta `int` (4 bayt) so'rasangiz ham, protsessor uning atrofidagi 64 baytni birdan cache'ga tortadi. Bu bejiz emas — pastdagi "spatial locality" buni tushuntiradi.

```text
Siz array[0] (4 bayt) so'radingiz. Cache 64 baytni oladi:

RAM:  [ a[0] a[1] a[2] ... a[15] | a[16] ... ]
        \______ 64 bayt = 1 cache line ______/
        \___________________/
         bitta murojaatda cache'ga keladi

Endi a[1]..a[15] uchun murojaat = HIT (tekin!)
```

## Locality: temporal va spatial

Cache ishlashining butun mantig'i **locality of reference** ("murojaat lokalligi") tamoyiliga asoslanadi. Ikki tur bor:

**Temporal locality (vaqt bo'yicha):** Yaqinda ishlatilgan ma'lumot yaqin kelajakda yana ishlatilishi ehtimoli katta. Misol: siklda ishlatiladigan hisoblagich (`i`), tez-tez chaqiriladigan funksiya.

**Spatial locality (fazoviy):** Bir manzildagi ma'lumot ishlatilsa, unga yaqin manzillar ham tez orada ishlatilishi ehtimoli katta. Misol: array elementlarini ketma-ket aylanib chiqish.

```text
Temporal:  bir joyni QAYTA-QAYTA ishlatish
           t1: a  t2: a  t3: a   --> a keshda qoladi

Spatial:   YONMA-YON joylarni ishlatish
           a[0] a[1] a[2] a[3]   --> hammasi bitta cache line'da
```

**💡 Tushuncha:** Cache line 64 bayt bo'lgani sabab, spatial locality "tekin" tezlik beradi. Bir cache line ichidagi keyingi elementlar allaqachon keshda — ularni o'qish uchun RAM'ga borish shart emas.

## Cache-friendly kod

Endi eng amaliy qism. Bir xil natijani beradigan, lekin cache tufayli **tezligi keskin farq qiladigan** ikki kod. Klassik misol — 2D matritsani aylanib chiqish.

C tilida (va ko'p tillarda) 2D array **row-major** tartibda xotirada saqlanadi: bir qatorning elementlari yonma-yon yotadi.

```text
matrix[3][3] xotirada (row-major):

manzil:  0    1    2    3    4    5    6    7    8
qiymat: m00  m01  m02  m10  m11  m12  m20  m21  m22
        \___ qator 0 ___/\___ qator 1 __/\__ qator 2 _/
```

**Row-major aylanish (CACHE-FRIENDLY):**

```js
// Tashqi sikl - qator, ichki sikl - ustun
let sum = 0;
for (let i = 0; i < N; i++) {
  for (let j = 0; j < N; j++) {
    sum += matrix[i][j];   // ketma-ket manzillar -> spatial locality
  }
}
// Har cache line'dagi 16 ta element ketma-ket ishlatiladi -> ko'p HIT
```

**Column-major aylanish (CACHE-UNFRIENDLY):**

```js
// Tashqi sikl - ustun, ichki sikl - qator
let sum = 0;
for (let j = 0; j < N; j++) {
  for (let i = 0; i < N; i++) {
    sum += matrix[i][j];   // manzil har safar N ga "sakraydi"
  }
}
// Har murojaat yangi cache line -> ko'p MISS -> bir necha barobar sekin
```

**💡 Tushuncha:** Ikkala kod ham bir xil elementlarni qo'shadi va bir xil natija beradi. Farq faqat **tartibda**. Katta matritsalarda row-major variant column-major'dan **2x-10x tezroq** bo'lishi mumkin — faqat cache'dan yaxshi foydalangani uchun.

**⚠️ Ehtiyot bo'l:** Til va kutubxonaga qarab saqlash tartibi farq qiladi. C, C++, Python (NumPy default), JS massivlari — **row-major**. Fortran, MATLAB, R — **column-major**. Kodni yozishdan oldin tilingiz qaysi tartibda saqlashini biling, aks holda "cache-friendly" deb yozgan kodingiz aslida teskari bo'lib chiqishi mumkin.

## RAM: DRAM vs SRAM

Asosiy xotira (main memory) — bu **RAM** (Random Access Memory). Uning ikki asosiy turi bor:

| Xususiyat | SRAM (Static) | DRAM (Dynamic) |
|---|---|---|
| Ishlatilishi | Cache (L1/L2/L3) | Asosiy xotira (RAM plankalari) |
| Tezlik | Juda tez | Sekinroq |
| Zichlik (hajm) | Past (kam sig'adi) | Yuqori (ko'p sig'adi) |
| Narx (bit uchun) | Qimmat | Arzon |
| Tuzilishi | ~6 tranzistor / bit | 1 tranzistor + 1 kondensator / bit |
| Refresh kerakmi? | Yo'q | Ha (kondensator zaryadi oqib ketadi) |

**💡 Tushuncha:** DRAM "dynamic" deyilishining sababi — u ma'lumotni kichik kondensatorlarda saqlaydi, kondensator zaryadi vaqt o'tishi bilan oqib ketadi, shuning uchun uni doimiy **refresh** qilib turish kerak. SRAM refresh talab qilmaydi, shuning uchun tezroq, ammo har bit uchun ko'proq tranzistor kerak — bu esa qimmat va kam zich.

**💡 Tushuncha:** Ikkala RAM turi ham **volatile** (uchuvchan) — quvvat o'chganda ma'lumot yo'qoladi. Doimiy saqlash (persistence) uchun SSD/HDD kabi **non-volatile** qurilmalar kerak. Shuning uchun kompyuter o'chib qayta yonganda RAM'dagi hamma narsa yo'qoladi, disk esa saqlanib qoladi.

## Virtual memory va paging

Har bir dastur o'zini xotiraning yagona egasi deb "o'ylaydi" va uzluksiz manzil fazosiga (address space) ega bo'ladi. Aslida bu illyuziya — buni **virtual memory** yaratadi.

**💡 Tushuncha:** Dastur ishlatadigan manzillar **virtual manzillar**. OS va CPU (MMU — Memory Management Unit) ularni **fizik manzillarga** aylantiradi. Bu tarjima **page** (sahifa) deb ataladigan bloklar orqali amalga oshadi (odatda **4 KB**).

```text
Virtual xotira                    Fizik RAM
+-------------+                   +-------------+
| page 0      | --- xarita --->   | frame 5     |
| page 1      | --------\         | frame 2     |
| page 2      | ---\     \--->    | ...         |
+-------------+     \             +-------------+
                     \-----> DISK (swap) da saqlanishi mumkin
```

- **Page fault:** Dastur so'ragan sahifa RAM'da yo'q (diskka "swap" qilingan yoki hali yuklanmagan). OS uni diskdan RAM'ga yuklaydi — bu **juda sekin** (disk latency!).
- **TLB (Translation Lookaside Buffer):** Virtual→fizik manzil tarjimasini har safar hisoblash qimmat. Shuning uchun oxirgi tarjimalar **TLB** deb ataladigan kichik, tez keshda saqlanadi. TLB hit — tez; TLB miss — sahifa jadvalidan (page table) izlash kerak, sekinroq.

**⚠️ Ehtiyot bo'l:** Page fault — bu xato emas, oddiy mexanizm. Ammo agar dasturingiz RAM'dan ko'proq xotira ishlatib, doimiy diskka swap qilinsa, bu **thrashing** holatiga olib keladi va dastur o'ta sekinlashadi. Bu sodir bo'lganda tizim "muzlab qolgandek" ko'rinadi.

## Memory-mapped va sequential access

**Memory-mapped file:** Faylni to'g'ridan-to'g'ri jarayon manzil fazosiga "xaritalash". Fayl bilan xuddi xotira massivi kabi ishlash mumkin — OS kerakli sahifalarni avtomatik yuklaydi (paging orqali). Katta fayllar bilan qulay va tez.

**Nega "sequential access" (ketma-ket murojaat) tez?**

```text
Sequential (tez):   a[0] a[1] a[2] a[3] a[4] ...
                    ->->->->->  bitta yo'nalishda oldinga

Random (sekin):     a[7] a[0] a[19] a[3] a[42] ...
                    sakrash-sakrash  har xil joyga
```

Ketma-ket murojaat tez, chunki:

1. **Cache line** — yonma-yon elementlar bitta cache line'da keladi (spatial locality).
2. **Prefetching** — protsessor ketma-ket naqshni "sezadi" va keyingi cache line'larni oldindan (siz so'ramasdan) tortib qo'yadi.
3. **HDD'da** — o'qish kallagi (head) siljimaydi, disk aylanib ma'lumot ketma-ket o'qiladi; random access esa kallakni har safar boshqa joyga surishga majbur qiladi (seek time).

**💡 Tushuncha:** Shuning uchun katta ma'lumotni qayta ishlashda uni ketma-ket (streaming) o'qish, tasodifiy sakrashlardan ancha tez. Ma'lumotlar tuzilmasini tanlashda ham buni hisobga oling: linked list (tarqoq) vs array (ketma-ket).

## Dasturchi uchun amaliy xulosalar

- **Locality'dan foydalaning.** Ma'lumotni ketma-ket, bir joyda saqlab, ketma-ket o'qing. Array — cache uchun linked list'dan yaxshiroq.
- **Sikl tartibiga e'tibor bering.** 2D ma'lumotni tilingiz saqlash tartibiga mos aylantiring (row-major tilda ichki sikl ustun bo'ylab).
- **Ma'lumot tuzilmasini zich (compact) tuting.** Kerakmas maydonlar cache line'ni "isrof" qiladi. Tez-tez ishlatiladigan maydonlarni yonma-yon joylashtiring.
- **Diskka/tarmoqqa borishni kamaytiring.** Ular ierarxiyaning eng sekin qismi. Keshlash (caching), batching, sequential I/O yordam beradi.
- **Ishchi to'plam (working set) hajmini bilib turing.** Agar ma'lumotingiz cache'ga sig'sa — tez; RAM'ga sig'sa — o'rtacha; RAM'dan oshsa (swap) — falokat.
- **O'lchang, taxmin qilmang.** Cache effektlari intuitiv emas. Profiler bilan o'lchab tekshiring.

## Savol-javob

### ❓ Nima uchun bitta katta, tez xotira o'rniga bir necha darajali xotira ishlatiladi?

**✅ Javob:** Chunki tezlik, hajm va narx bir-biriga zid (trade-off). Tez xotira (SRAM) qimmat va kichik; arzon xotira (DRAM, disk) sekin. Bir necha daraja ishlatib, oz miqdorda tez xotirani protsessorga yaqin qo'yamiz. Locality tufayli ma'lumotning aksariyati tez darajada topiladi, natijada tizim o'rtacha tez ishlaydi, lekin katta hajmni arzon narxda saqlaydi.

### ❓ Cache hit va cache miss nima?

**✅ Javob:** Cache **hit** — CPU so'ragan ma'lumot cache'da bor edi, tez qaytariladi. Cache **miss** — cache'da yo'q edi, quyiroq (sekinroq) darajaga borish kerak bo'ldi. Miss ratio qancha past bo'lsa, dastur shuncha tez ishlaydi.

### ❓ Cache line nima va nega uning hajmi muhim?

**✅ Javob:** Cache line — cache va RAM o'rtasida ko'chiriladigan eng kichik blok (odatda 64 bayt). CPU bitta baytni so'rasa ham, butun 64 baytlik line'ni oladi. Bu spatial locality'dan foydalanadi: yonma-yon elementlarga keyingi murojaatlar tekin hit bo'ladi. Shuning uchun ma'lumotni zich va ketma-ket saqlash foydali.

### ❓ Temporal va spatial locality o'rtasidagi farq nima?

**✅ Javob:** Temporal — bir joyni qayta-qayta ishlatish (vaqt bo'yicha yaqinlik), masalan sikl hisoblagichi. Spatial — yonma-yon joylashgan manzillarni ishlatish (fazoda yaqinlik), masalan array'ni ketma-ket aylanish. Cache ikkalasidan ham foydalanadi.

### ❓ Row-major va column-major aylanish tezligi nega farq qiladi?

**✅ Javob:** Chunki 2D array xotirada bir tartibda (masalan, row-major — qatorlar yonma-yon) saqlanadi. Xotiradagi tartibga mos aylanganda (ichki sikl bir cache line bo'ylab) ko'p cache hit bo'ladi. Teskari aylanish har murojaatda yangi cache line'ga sakraydi — ko'p miss, bir necha barobar sekin. Natija bir xil, tezlik esa keskin farqli.

### ❓ DRAM va SRAM o'rtasidagi asosiy farqlar qanday?

**✅ Javob:** SRAM tez, qimmat, kam zich, refresh talab qilmaydi — cache uchun ishlatiladi. DRAM sekinroq, arzon, ko'p zich, doimiy refresh kerak (kondensator zaryadi oqib ketadi) — asosiy RAM uchun ishlatiladi. Ikkalasi ham volatile.

### ❓ "Volatile xotira" nima degani?

**✅ Javob:** Volatile (uchuvchan) xotira quvvat o'chganda ma'lumotni yo'qotadi — RAM (DRAM/SRAM) shunday. Non-volatile (SSD/HDD) esa quvvatsiz ham ma'lumotni saqlaydi. Shuning uchun muhim ma'lumotni doimiy saqlash uchun diskka yozish kerak.

### ❓ Virtual memory nima uchun kerak?

**✅ Javob:** U har bir jarayonga alohida, uzluksiz manzil fazosi illyuziyasini beradi, jarayonlarni bir-biridan izolyatsiya qiladi (xavfsizlik), va RAM'dan ko'proq xotira ishlatish imkonini beradi (diskka swap orqali). CPU/OS virtual manzillarni fizik manzillarga page'lar orqali tarjima qiladi.

### ❓ Page fault nima va u nega qimmat?

**✅ Javob:** Page fault — dastur so'ragan sahifa hozir RAM'da yo'q (diskka swap qilingan yoki hali yuklanmagan). OS uni diskdan RAM'ga yuklaydi. Bu qimmat, chunki disk latency RAM'dan ~1000x sekin. Agar bu tez-tez takrorlansa (thrashing), dastur o'ta sekinlashadi.

### ❓ TLB nima vazifa bajaradi?

**✅ Javob:** TLB (Translation Lookaside Buffer) — virtual→fizik manzil tarjimalarini saqlaydigan kichik, tez kesh. TLB hit bo'lsa tarjima darhol tayyor; TLB miss bo'lsa page table'dan izlash kerak (sekinroq). U manzil tarjimasini tezlashtiradi.

### ❓ Nega sequential access random access'dan tez?

**✅ Javob:** Uch sabab: (1) yonma-yon elementlar bitta cache line'da keladi — spatial locality; (2) protsessor ketma-ket naqshni sezib, keyingi ma'lumotni oldindan tortadi (prefetching); (3) HDD'da o'qish kallagi siljimaydi (seek time yo'q). Random access bularning hech biridan foyda ko'rmaydi.

### ❓ Memory-mapped file nima?

**✅ Javob:** Faylni jarayon manzil fazosiga xaritalash. Fayl bilan xuddi xotira massivi kabi ishlash mumkin; OS kerakli sahifalarni paging orqali avtomatik yuklaydi/saqlaydi. Katta fayllar bilan qulay va samarali I/O beradi.

### ❓ Cache o'zi qaysi xotira turidan yasaladi?

**✅ Javob:** Cache SRAM'dan yasaladi — u tez, refresh talab qilmaydi. Shuning uchun cache RAM'dan (DRAM) ancha tez, ammo qimmat va kam sig'imli.

### ❓ "Working set" nima va nega u muhim?

**✅ Javob:** Working set — dastur ma'lum vaqt oralig'ida faol ishlatayotgan xotira miqdori. Agar u cache'ga sig'sa — juda tez; RAM'ga sig'sa — o'rtacha; RAM'dan oshsa — swap boshlanadi va dastur sekinlashadi. Kodni working set'ni kichik tutishga qarab optimallashtirish samarali.

### ❓ Prefetching nima?

**✅ Javob:** Prefetching — protsessor kelajakda kerak bo'ladigan ma'lumotni siz so'ramasdan oldindan cache'ga tortib qo'yishi. U murojaat naqshini (masalan, ketma-ketlikni) sezadi va keyingi cache line'larni tayyorlab qo'yadi. Sequential access shuning uchun tez.

### ❓ Ma'lumot tuzilmasini tanlashda cache'ni qanday hisobga olish kerak?

**✅ Javob:** Zich, ketma-ket joylashgan tuzilmalar (array, `struct` massivi) cache uchun yaxshi. Tarqoq, ko'rsatkichlar bilan bog'langan tuzilmalar (linked list, tree) har murojaatda cache miss keltirishi mumkin. Iloji bo'lsa contiguous (uzluksiz) xotirani afzal ko'ring.

## Masalalar

> Yechimlar: [solutions/hardware/03-memory-hierarchy.md](../solutions/hardware/03-memory-hierarchy.md)

1. Ierarxiyaning quyidagi darajalarini eng tezdan eng sekingacha tartiblang: RAM, L1 cache, HDD, registers, SSD, L3 cache. Har biriga taxminiy latency yozing.

2. Bir cache line 64 bayt. `int` 4 bayt. Bitta cache line'ga nechta `int` sig'adi? Agar siz array'ni ketma-ket aylansangiz, har nechta elementdan keyin bitta cache miss kutiladi (boshqa hamma narsa teng bo'lsa)?

3. Quyidagi ikki sikldan qaysi biri cache-friendly va nega? Tilingiz row-major saqlaydi deb faraz qiling.
   ```js
   // A
   for (let i=0;i<N;i++) for (let j=0;j<N;j++) s += m[i][j];
   // B
   for (let j=0;j<N;j++) for (let i=0;i<N;i++) s += m[i][j];
   ```

4. Nima uchun DRAM'ni doimiy "refresh" qilish kerak, SRAM'ni esa yo'q? Bu ularning narxi va hajmiga qanday ta'sir qiladi?

5. Bir dastur RAM'dan ko'proq xotira ishlatmoqchi bo'ldi. Virtual memory buni qanday imkon beradi va bunda qanday performance xavfi bor?

6. "Cache hit ratio 95% dan 99% ga oshdi" — bu miss ratio'ni necha barobar kamaytiradi? Nega bu kichik ko'rinadigan farq katta ahamiyatga ega bo'lishi mumkin?

7. Linked list va array bir xil sonli butun sonlarni saqlaydi. Ularni to'liq aylanib chiqishda cache nuqtai nazaridan qaysi biri tezroq va nega?

8. Sizga 10 GB fayl berildi, uni qayta ishlash kerak, lekin RAM 8 GB. Sequential streaming o'qish va memory-mapped yondashuvlarni tavsiflab, nega ular random access'dan yaxshiroq ekanini tushuntiring.

← [Hardware bo'limiga qaytish](./README.md)
