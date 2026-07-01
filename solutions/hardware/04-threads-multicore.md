# Thread'lar va Multicore — Yechimlar

Bu fayl [`hardware/04-threads-multicore.md`](../../hardware/04-threads-multicore.md) hujjatidagi "Masalalar" bo'limining yechimlarini o'z ichiga oladi. Avval o'zingiz yechishga urinib ko'ring.

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

**Savol:** "6 core 12 thread" — nechta physical core? SMT bormi? Nega 12x tezlik kafolatlanmaydi?

**✅ Javob:** **6 physical core.** 12 hardware thread = 6 core × 2, ya'ni **SMT (hyperthreading) ishlatilgan** (har core 2 thread ko'rsatadi).

12x tezlik kafolatlanmaydi, chunki faqat 6 haqiqiy core bor. Qolgan 6 "thread" SMT orqali — ular bir core'ning ALU, cache resurslarini baham ko'radi, alohida to'liq core emas. SMT foydasi odatda ~20-30%. Bundan tashqari Amdahl qonuni, sinxronizatsiya va memory bandwidth ham chegara qo'yadi.

## 2-masala

**Savol:** P = 0.90, 4 va 8 core uchun speedup. Cheksiz core'da maksimal?

**✅ Javob:** Formula: `Speedup(N) = 1 / ((1-P) + P/N)`, P = 0.90.

- **N = 4:** 1 / (0.10 + 0.90/4) = 1 / (0.10 + 0.225) = 1 / 0.325 ≈ **3.08x**
- **N = 8:** 1 / (0.10 + 0.90/8) = 1 / (0.10 + 0.1125) = 1 / 0.2125 ≈ **4.71x**
- **N → cheksiz:** 1 / (1 - P) = 1 / 0.10 = **10x** (maksimal chegara)

**💡 Tushuncha:** 10% serial qism cheksiz core bilan ham 10x'dan oshishga yo'l qo'ymaydi. Diminishing returns — 4'dan 8 core'ga o'tish faqat ~1.5x qo'shdi.

## 3-masala

**Savol:** Concurrency va parallelism uchun real misol.

**✅ Javob:**

- **Concurrency:** Bitta oshpaz bir vaqtda bir necha taomni "olib boradi" — birini pishirayotib, ikkinchisi qaynayotganda uchinchisini to'g'raydi. Bitta oshpaz (bitta core), lekin ishlar bir-birlashib boradi (navbatlash). Yoki: JS event loop I/O kutayotganda boshqa kodni ishlatishi.
- **Parallelism:** Ikki oshpaz ikki alohida taomni **bir vaqtda** pishiradi — ikkalasi bir paytda ishlaydi. Bu haqiqiy parallelism, chunki ikki mustaqil "core" bor. Yoki: 4 core'da 4 worker matritsani parallel hisoblashi.

Farq: concurrency — bitta resursda navbatlash; parallelism — bir necha resursda bir vaqtda bajarish.

## 4-masala

**Savol:** Ikki thread qo'shni xotiradagi hisoblagichlarni oshiryapti va kod sekin. Nega? Qanday tuzatasiz?

**✅ Javob:** Bu **false sharing**. `a` va `b` xotirada yaqin bo'lgani uchun bitta cache line'ga (64 bayt) tushishi mumkin. Thread 0 `a` ni oshirsa, butun line invalidate bo'ladi va thread 1 `b` ni (o'zgarmagan!) qaytadan yuklaydi. Core'lar bir-birini "urib turadi" (cache line ping-pong), parallel kod sekin, hatto ketma-ketdan yomon bo'ladi.

**Tuzatish:**
- O'zgaruvchilarni alohida cache line'ga joylash — orasiga padding qo'shish (masalan, har hisoblagichni 64 baytga tekislash / align qilish).
- Yoki har thread o'z lokal (thread-local) hisoblagichida ishlab, oxirida yig'ish.

## 5-masala

**Savol:** I/O-bound serverga 100 worker_threads yaratildi. Nega yomon? Qachon worker foydali?

**✅ Javob:** I/O-bound ish uchun worker_threads **kerak emas** — Node.js'ning event loop'i I/O ni asinxron, single-thread'da samarali boshqaradi. 100 worker:
- Har biri xotira (stack, heap) va yaratish xarajati keltiradi.
- Ular fizik core'dan (masalan 8) ancha ko'p — over-subscription, doimiy context switch.
- Xabar almashish (serialization) qo'shimcha overhead.
Natijada tezlashish o'rniga sekinlashish mumkin.

**Worker qachon foydali:** ish **CPU-bound** bo'lganda (og'ir hisob-kitob main thread'ni bloklaganda) — masalan, rasm/video qayta ishlash, katta matematik hisob. Worker soni core soniga yaqin bo'lgani optimal.

## 6-masala

**Savol:** Context switch'da hardware'da nima saqlanadi/tiklanadi? Nega "sovuq cache" yashirin xarajat?

**✅ Javob:** Saqlanadi va tiklanadi: barcha **registerlar**, **program counter** (keyingi buyruq manzili), **stack pointer**, holat/flag registerlari. Eski thread holati xotiraga saqlanadi, yangi thread holati xotiradan tiklanadi.

**Sovuq cache:** eski thread ishlaganda cache'ni o'z ma'lumotlari bilan to'ldirgan (issiq). Yangi thread ishga tushganda unga kerakli ma'lumot cache'da yo'q — ko'p cache miss'dan boshlaydi (cache "sovuq"). TLB ham yangilanadi. Bu xarajat context switch'ning bevosita register saqlash vaqtidan tashqarida bo'lgani uchun "yashirin".

## 7-masala

**Savol:** MESI'da core line'ni Modified qilsa, boshqa nusxalarga nima bo'ladi va nega?

**✅ Javob:** Boshqa barcha core'lardagi shu cache line nusxalari **Invalid (I)** holatiga o'tadi. Sababi: Modified holat "bu core'da yagona yangi (o'zgartirilgan) nusxa bor, RAM ham eskirgan" degani. Agar boshqa core'larda eski nusxa qolsa, ular noto'g'ri (stale) qiymatni ko'rardi — cache coherence buzilardi. Shuning uchun o'zgartirish oldidan protsessor bus orqali "invalidate" signali yuboradi va boshqa nusxalar yaroqsiz qilinadi. Endi boshqa core o'sha manzilni so'rasa, u yangi (Modified) qiymatni olishga majbur.

## 8-masala

**Savol:** To'liq CPU-bound, yaxshi parallellashuvchi ish. 8 core / 16 thread mashinada nechta thread? Nega 8 ga yaqin, 16 emas?

**✅ Javob:** Odatda **physical core soniga yaqin — ~8 thread** yaxshi boshlang'ich nuqta (yoki o'lchab optimal topish).

Nega 16 emas: sof CPU-bound (I/O kutish yo'q) ishda thread'lar doim ALU'ni band qiladi — SMT'ning foydasi kam, chunki SMT bo'sh vaqtdan (RAM kutish) foydalanadi, bunday ish esa deyarli kutmaydi. 16 worker'da har juftlik bitta core resurslarini bo'lishishga majbur bo'lib, bir-birini sekinlashtiradi, cache bosimi va context switch ortadi. Shuning uchun ~8 (yoki 8-16 orasida o'lchab) ko'pincha optimal.

**💡 Tushuncha:** Qoida: I/O-bound ish uchun thread'ni ko'proq qilish mumkin (kutish paytida boshqasi ishlaydi); CPU-bound ish uchun thread soni physical core soniga yaqin bo'lgani yaxshi. Har doim o'lchab (benchmark) tekshiring.

← [Hardware bo'limiga qaytish](../../hardware/README.md)
