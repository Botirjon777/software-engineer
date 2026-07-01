# Xotira Ierarxiyasi — Yechimlar

Bu fayl [`hardware/03-memory-hierarchy.md`](../../hardware/03-memory-hierarchy.md) hujjatidagi "Masalalar" bo'limining yechimlarini o'z ichiga oladi. Avval o'zingiz yechishga urinib ko'ring, keyin bu yerga qarang.

## Mundarija

- [1-masala](#1-masala)
- [2-masala](#2-masala)
- [3-masala](#3-masala)
- [4-masala](#4-masala)
- [5-masala](#5-masala)
- [6-masala](#6-masala)
- [7-masala](#7-masala)
- [8-masala](#8-masala)

## 1-masala

**Savol:** Darajalarni eng tezdan eng sekingacha tartiblang: RAM, L1 cache, HDD, registers, SSD, L3 cache.

**✅ Javob:**

| O'rin | Daraja | Taxminiy latency |
|---|---|---|
| 1 (eng tez) | Registers | < 1 ns |
| 2 | L1 cache | ~1 ns |
| 3 | L3 cache | ~15 ns |
| 4 | RAM | ~100 ns |
| 5 | SSD | ~100 us |
| 6 (eng sekin) | HDD | ~10 ms |

**💡 Tushuncha:** Nisbatlar muhim. Registers'dan RAM'gacha ~100x farq, RAM'dan SSD'gacha yana ~1000x, SSD'dan HDD'gacha yana ~100x. Har qadam katta sakrash.

## 2-masala

**Savol:** 64 baytlik cache line'ga nechta 4 baytlik `int` sig'adi? Ketma-ket aylanishda har nechta elementdan keyin miss?

**✅ Javob:** 64 / 4 = **16 ta `int`** bitta cache line'ga sig'adi.

Ketma-ket aylanganda: birinchi elementga murojaat miss keltiradi (line yuklanadi), keyingi 15 tasi hit. Ya'ni har **16 elementda taxminan 1 miss**. Cache miss ratio ~1/16 ≈ 6.25%.

**💡 Tushuncha:** Bu spatial locality'ning kuchi — bitta miss narxiga 16 ta elementni "olib kelasiz". Prefetching esa bu miss'ni ham ko'pincha yashiradi.

## 3-masala

**Savol:** Row-major saqlanadigan tilda A va B sikllardan qaysi biri cache-friendly?

**✅ Javob:** **A cache-friendly.**

- **A:** ichki sikl `j` — `m[i][j]` da `j` o'zgarganda manzil ketma-ket oshadi (bir qatorning yonma-yon elementlari). Spatial locality — ko'p hit.
- **B:** ichki sikl `i` — `m[i][j]` da `i` o'zgarganda manzil har safar bir qator (N element) ga sakraydi. Har murojaat yangi cache line — ko'p miss.

Katta N da A, B'dan bir necha barobar tezroq. Ikkalasi bir xil natija beradi.

## 4-masala

**Savol:** Nega DRAM refresh kerak, SRAM'ga yo'q? Narx/hajmga ta'siri?

**✅ Javob:** DRAM har bitni bitta kondensatorda saqlaydi; kondensator zaryadi vaqt o'tishi bilan oqib ketadi, shuning uchun ma'lumot yo'qolmasligi uchun uni doimiy **refresh** (qayta zaryadlash) kerak. SRAM esa har bitni ~6 tranzistorli triggerda saqlaydi — u zaryad ushlab turadi, refresh shart emas.

Ta'siri: DRAM'ning bit uchun 1 tranzistor + 1 kondensatori zich va arzon — shuning uchun katta hajmli asosiy xotira. SRAM'ning 6 tranzistori ko'p joy va energiya oladi — qimmat, kam zich, lekin tez. Shuning uchun SRAM faqat kichik cache'lar uchun.

## 5-masala

**Savol:** Virtual memory RAM'dan ko'proq xotira ishlatishga qanday imkon beradi? Performance xavfi?

**✅ Javob:** Virtual memory har jarayonga katta virtual manzil fazosi beradi. Faol ishlatilmayotgan sahifalarni (page) diskka **swap** qiladi va kerak bo'lganda qaytaradi. Shunday qilib dastur fizik RAM'dan ko'proq xotira "ishlatgandek" bo'ladi.

Xavf: agar working set RAM'dan katta bo'lib, sahifalar doimiy diskka/diskdan ko'chsa — **thrashing** boshlanadi. Disk latency RAM'dan ~1000x sekin bo'lgani uchun dastur o'ta sekinlashadi, tizim "muzlab qolgandek" ko'rinadi.

## 6-masala

**Savol:** Hit ratio 95% dan 99% ga oshsa, miss ratio necha barobar kamayadi? Nega muhim?

**✅ Javob:** Miss ratio 5% dan 1% ga tushadi — bu **5 barobar** kamayish (5% / 1% = 5).

Nima uchun muhim: miss juda qimmat (RAM ~100 ns, hit ~1 ns). Umumiy vaqtni asosan miss'lar belgilaydi. Miss'lar 5x kamaysa, o'rtacha murojaat vaqti sezilarli tushadi.

**Taxminiy hisob** (hit 1 ns, miss jami ~100 ns):
- 95% hit: 0.95×1 + 0.05×100 = 5.95 ns o'rtacha
- 99% hit: 0.99×1 + 0.01×100 = 1.99 ns o'rtacha

Ya'ni ~3x tezroq — atigi 4% hit ratio o'zgarishidan.

## 7-masala

**Savol:** Linked list va array — to'liq aylanishda cache jihatidan qaysi tezroq?

**✅ Javob:** **Array tezroq.**

- **Array:** elementlar xotirada uzluksiz (contiguous) yotadi. Ketma-ket aylanganda spatial locality ishlaydi — bir cache line'da ko'p element, prefetching ham yordam beradi. Kam miss.
- **Linked list:** node'lar xotirada tarqoq (har biri `malloc` bilan har xil joyda) bo'lishi mumkin. Har `next` ko'rsatkichni kuzatish yangi, bog'liq bo'lmagan manzilga sakrash — ko'p cache miss. Qo'shimcha, har node'da ko'rsatkich uchun joy sarflanadi.

Shuning uchun ketma-ket aylanishda array odatda linked list'dan sezilarli tez.

## 8-masala

**Savol:** 10 GB fayl, 8 GB RAM. Sequential streaming va memory-mapped — nega random access'dan yaxshi?

**✅ Javob:**

**Sequential streaming:** faylni bo'lak-bo'lak (masalan, 64 KB blok) ketma-ket o'qib qayta ishlab, tashlab yuborish. Bir vaqtda faqat kichik bufer RAM'da — 8 GB RAM yetadi. Ketma-ket o'qish tez: OS prefetch qiladi, disk kallagi siljimaydi.

**Memory-mapped:** faylni virtual manzil fazosiga xaritalash. OS kerakli sahifalarni avtomatik yuklaydi (paging), ishlatilmaganlarini chiqaradi. Kod fayl bilan xotira massivi kabi ishlaydi.

**Nega random access yomon:** butun 10 GB'ni RAM'ga sig'dirib bo'lmaydi (8 GB), shuning uchun tasodifiy sakrashlar doimiy page fault / disk seek keltiradi — o'ta sekin. Sequential esa cache line, prefetching va (HDD'da) seek'siz o'qishdan foyda ko'radi.

**💡 Tushuncha:** Katta ma'lumotni "oqim" (stream) sifatida ketma-ket qayta ishlash — RAM'dan katta datasetlar uchun asosiy naqsh. Bu locality tamoyilining amaliy natijasi.

← [Hardware bo'limiga qaytish](../../hardware/README.md)
