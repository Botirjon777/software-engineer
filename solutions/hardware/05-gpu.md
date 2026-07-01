# GPU va Parallel Computing — Yechimlar

Bu fayl [hardware/05-gpu.md](../../hardware/05-gpu.md) mashqlarining yechimlari. Avval o'zingiz yechib ko'ring, keyin solishtiring.

## Mundarija

- [1-masala: Ko'p oddiy vs kam kuchli core](#1-masala)
- [2-masala: Masalalarni tasniflash](#2-masala)
- [3-masala: Warp divergence](#3-masala)
- [4-masala: PCIe bottleneck hisobi](#4-masala)
- [5-masala: SIMD vs SIMT jadval](#5-masala)
- [6-masala: Thread iyerarxiyasi diagrammasi](#6-masala)
- [7-masala: Latency hiding](#7-masala)
- [8-masala: Nega GPU ML uchun](#8-masala)

---

## 1-masala

**Savol:** Nega GPU'da minglab oddiy core, CPU'da kam lekin kuchli core?

**Yechim:** Ikki xil maqsad, ikki xil dizayn:

- **CPU (latency-oriented):** kam sonli core, lekin har biri kuchli — katta cache, out-of-order bajarish, branch prediction. Maqsad: bitta murakkab, ketma-ket vazifani imkon qadar tez tugatish. Afzallik: operatsion tizim, biznes mantiq, ko'p tarmoqli `if/else` bo'lgan kodlar.

- **GPU (throughput-oriented):** transistorlar minglab oddiy ALU'ga sarflanadi. Har core kuchsiz (kichik cache, oddiy control), lekin ular birgalikda million bir xil hisobni parallel bajaradi. Afzallik: grafika, matritsa, ML.

Xulosa: bittani tez qilish kerak bo'lsa — CPU; millionni birga qilish kerak bo'lsa — GPU.

## 2-masala

**Savol:** Masalalarni GPU'ga mos/mos emas deb tasniflang.

**Yechim:**

| Masala | Tasnif | Sabab |
|--------|--------|-------|
| (a) 4K rasmga blur filtri | ✅ Mos | Har piksel mustaqil, bir xil hisob — massiv parallel |
| (b) Fibonachchi rekursiv | ❌ Mos emas | Ketma-ket, har qadam oldingisiga bog'liq (data dependency) |
| (c) 10000×10000 matritsa ko'paytirish | ✅ Mos | Ko'p mustaqil ko'paytma-qo'shma, GPU'ning "ideal" ishi |
| (d) Matn parsing | ❌ Mos emas | Branch-heavy, ketma-ket, mustaqil parallellik yo'q |

## 3-masala

**Savol:** `if (threadId % 2 == 0)` misolida warp divergence.

**Yechim:** Warp — 32 thread bir vaqtda bir instruksiyani bajaradi. `threadId % 2 == 0` da juft thread'lar bir yo'lga, toq thread'lar boshqa yo'lga ketadi. GPU bitta warp uchun ikkala yo'lni ham **ketma-ket** bajarishi kerak: avval juft thread'lar ishlaydi (toqlar kutadi), keyin toqlar (juftlar kutadi). Natijada parallellik ~2 barobar yo'qoladi.

**Yaxshilash:** Thread'larni shunday qayta guruhlash kerakki, bitta warp ichidagi hamma thread bir xil yo'ldan ketsin. Masalan, ma'lumotni juft/toq bo'yicha oldindan ajratib qo'yish, yoki indekslashni warp chegarasiga moslash — shunda har warp faqat bitta tarmoqni bajaradi, divergence bo'lmaydi.

## 4-masala

**Savol:** Hisob 2ms, ko'chirish 50ms. Umumiy vaqt va bottleneck?

**Yechim:**
- Umumiy vaqt ≈ 50ms (ko'chirish) + 2ms (hisob) = **52ms**.
- Bottleneck: **RAM↔VRAM ko'chirish (PCIe)** — u umumiy vaqtning ~96% ini egallaydi. GPU tez bo'lsa ham, foydasi ko'rinmaydi.

**Yaxshilash usullari:**
- Ma'lumotni bir marta VRAM'ga yuklab, u yerda ko'p amal bajarish (kam ko'chirish).
- Ko'chirish va hisobni **overlap** qilish (asinxron stream: bir qism ko'chayotganda oldingi qism hisoblanadi).
- Iloji bo'lsa, natijalarni ham VRAM'da saqlab, faqat kerakligini qaytarish.

## 5-masala

**Savol:** SIMD vs SIMT jadval.

**Yechim:**

| Nuqta | SIMD | SIMT |
|-------|------|------|
| Model | Bitta instruksiya vektor (ko'p ma'lumot) ustida | Bitta instruksiya ko'p thread ustida |
| Dasturchi ko'rinishi | Vektor registrlari bilan ishlaydi | Har element uchun alohida thread yozadi |
| Divergence | Yo'q (qat'iy vektor) | Bor (warp divergence mumkin) |
| Moslashuvchanlik | Kamroq | Ko'proq (har thread o'z indeksi/yo'li) |
| Misol | CPU AVX, SSE | NVIDIA CUDA warp'lari |

## 6-masala

**Savol:** Thread → warp → block → grid diagrammasi.

**Yechim:**

```
   GRID  ── butun masala (masalan, matritsa hammasi)
    │
    ├── BLOCK  ── warp'lar to'plami; shared memory + sinxronizatsiya
    │     │
    │     ├── WARP  ── 32 thread, bir vaqtda bir instruksiya (SIMT)
    │     │     │
    │     │     └── THREAD ── bitta element ustida ishlaydi
```

- **Thread:** eng kichik birlik, bitta ma'lumot elementiga ishlaydi.
- **Warp:** 32 thread, birgalikda bir instruksiya bajaradi.
- **Block:** warp'lar guruhi, umumiy shared memory va sinxronizatsiya.
- **Grid:** barcha block, butun masalani qamrab oladi.

## 7-masala

**Savol:** Latency hiding GPU'da qanday ishlaydi? Nega CPU'da bir xil emas?

**Yechim:** GPU'da minglab thread bor. Warp xotiradan ma'lumot kutayotganda (yuqori latency), GPU shu zahoti boshqa **tayyor** warp'ga o'tadi va core'ni band tutadi. Shunday qilib xotira kutishi "yashiriladi" — core hech qachon bo'sh turmaydi.

CPU'da bu darajada samarali emas, chunki:
- CPU'da thread'lar juda kam (core soni cheklangan).
- CPU o'rniga katta cache va prefetch bilan latency'ni kamaytirishga urinadi, "yashirish" emas.
- Kontekst almashish CPU'da qimmatroq; GPU esa warp almashishni deyarli tekin qiladi (registrlar oldindan ajratilgan).

## 8-masala

**Savol:** Nega deep learning GPU'da tez o'qitiladi?

**Yechim:** Neyron tarmoqning har qatlami — bu asosan **matritsa ko'paytirish** (input × weights). Bu esa minglab-millionlab **mustaqil** ko'paytma-qo'shmadan iborat. Aynan shu — ko'p, mustaqil, bir xil hisob — GPU eng yaxshi bajaradigan ish turi.

- CPU bu amallarni kam core bilan ketma-ket bajaradi → sekin.
- GPU minglab core bilan parallel bajaradi → o'nlab/yuzlab marta tez.

Shu sababli katta modellarni o'qitish CPU'da kunlar, GPU'da esa soatlar yoki daqiqalar oladi. TPU esa aynan shu matritsa amali uchun yanada ixtisoslashgan.

---

← [Hardware bo'limiga qaytish](../../hardware/README.md)
