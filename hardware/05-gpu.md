# GPU va Parallel Computing

GPU (Graphics Processing Unit) — dastlab ekranga rasm chizish uchun yaratilgan, keyin esa har qanday **massiv parallel** hisob-kitob uchun asosiy vositaga aylangan protsessor. Bu bo'limda GPU nima uchun paydo bo'ldi, uning arxitekturasi CPU'dan nima bilan farq qiladi, SIMT modeli qanday ishlaydi va nega machine learning aynan GPU'ga tayanadi — hammasini tushunamiz.

**💡 Tushuncha:** CPU — bir nechta juda kuchli ishchi (latency-oriented). GPU — minglab oddiy ishchi (throughput-oriented). Bitta murakkab vazifani CPU tez bajaradi; million bir xil oddiy vazifani GPU tez bajaradi.

## Mundarija

- [GPU nima va nega paydo bo'ldi](#gpu-nima-va-nega-paydo-bolgan)
- [GPU vs CPU arxitekturasi](#gpu-vs-cpu-arxitekturasi)
- [SIMD va SIMT](#simd-va-simt)
- [CUDA core, thread, warp, block](#cuda-core-thread-warp-block)
- [Throughput vs Latency](#throughput-vs-latency)
- [Qaysi masalalar GPU'ga mos](#qaysi-masalalar-gpuga-mos)
- [VRAM va PCIe bottleneck](#vram-va-pcie-bottleneck)
- [Host va Device: CPU va GPU birga](#host-va-device-cpu-va-gpu-birga)
- [GPGPU, TPU va ML](#gpgpu-tpu-va-ml)
- [Savol-Javob](#savol-javob)
- [Masalalar](#masalalar)

---

## GPU nima va nega paydo bo'lgan

1990-yillarda 3D o'yinlar va grafika ehtiyoji o'sdi. Ekrandagi har bir **piksel** uchun bir xil turdagi hisob-kitob kerak: yorug'lik, rang, joylashuv. Millionlab piksel, hammasi bir-biridan **mustaqil** — bu esa **parallel** ishga juda mos.

CPU bunday ishni ketma-ket bajaradi va sekin. Shuning uchun grafika uchun maxsus chip — GPU yaratildi: u kam kuchli, lekin juda ko'p **core**ga ega bo'lib, minglab pikselni bir vaqtda hisoblaydi.

Keyinchalik dasturchilar angladi: grafikadan tashqari ko'p masalalar ham "ko'p mustaqil oddiy hisob" ko'rinishida (matritsa, ML, kripto). Shundan **GPGPU** (General-Purpose GPU) tug'ildi.

## GPU vs CPU arxitekturasi

Asosiy farq — **transistor byudjeti qayerga sarflanadi**. CPU transistorlarining katta qismini boshqaruvga (control), cache va branch prediction'ga sarflaydi. GPU esa deyarli hammasini **ALU** (hisoblash birliklari)ga sarflaydi.

```
        CPU (latency-oriented)              GPU (throughput-oriented)
   ┌───────────────────────────┐     ┌───────────────────────────────┐
   │  [Core] [Core]            │     │ ▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪  │
   │  [Core] [Core]            │     │ ▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪  │
   │                           │     │ ▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪  │
   │  [KATTA CACHE]            │     │ ▪▪▪▪▪▪▪▪▪▪▪▪ minglab core ▪▪▪  │
   │  [CONTROL] [BRANCH PRED.] │     │ ▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪  │
   │                           │     │ [kichik cache] [oddiy control]│
   └───────────────────────────┘     └───────────────────────────────┘
   Kam sonli, kuchli core          Minglab, oddiy core
```

| Xususiyat | CPU | GPU |
|-----------|-----|-----|
| Core soni | 4–64 | minglab (masalan 5000+) |
| Bitta core kuchi | Juda yuqori | Past |
| Cache | Katta (MB) | Kichik (core boshiga) |
| Control logic | Murakkab (out-of-order, branch prediction) | Oddiy |
| Clock chastotasi | Yuqori (~5 GHz) | Pastroq (~2 GHz) |
| Optimizatsiya | Latency (bitta vazifa tez) | Throughput (ko'p vazifa umumiy tez) |
| Branch (if/else) | Yaxshi | Yomon |
| Mos ish | Ketma-ket, murakkab mantiq | Parallel, bir xil hisob |

## SIMD va SIMT

**SIMD** (Single Instruction, Multiple Data) — bitta instruksiya bir vaqtda ko'p ma'lumot ustida bajariladi. Masalan, `[1,2,3,4] + [5,6,7,8]` bitta "qo'shish" komandasi bilan hammasiga tatbiq etiladi. CPU'larda ham SIMD bor (SSE, AVX registrlari).

```
   Oddiy (SISD):          SIMD:
   a1 + b1 = c1           ┌──────────────────────┐
   a2 + b2 = c2           │ [a1 a2 a3 a4]        │
   a3 + b3 = c3    →      │ +[b1 b2 b3 b4]  bitta │
   a4 + b4 = c4           │ =[c1 c2 c3 c4]  komanda│
   (4 qadam)              └──────────────────────┘ (1 qadam)
```

**SIMT** (Single Instruction, Multiple Threads) — NVIDIA'ning modeli. Bu SIMD'ga o'xshaydi, lekin har bir "yo'l" alohida **thread** sifatida ko'riladi. Bitta instruksiya bir guruh thread (**warp**) ustida bir vaqtda bajariladi, lekin har thread o'z ma'lumoti va o'z indeksiga ega.

**⚠️ Ehtiyot bo'l:** SIMT'da agar warp ichidagi thread'lar `if/else`da turli yo'lga ketsa — **warp divergence** yuzaga keladi: GPU ikkala yo'lni ketma-ket bajaradi, natijada parallellik yo'qoladi va sekinlashadi.

## CUDA core, thread, warp, block

**CUDA core** (NVIDIA) yoki **stream processor** (AMD) — GPU ichidagi eng kichik hisoblash birligi, bitta thread'ning arifmetikasini bajaradi.

GPU parallelism modeli iyerarxik:

```
   Grid (butun masala)
   ├── Block 0 ──┬── Warp 0 (32 thread)
   │             ├── Warp 1 (32 thread)
   │             └── ...
   ├── Block 1 ──┬── Warp 0
   │             └── ...
   └── Block N
        │
        └── Har block: shared memory'ni bo'lishadi, sinxronlana oladi
```

| Termin | Ma'nosi |
|--------|---------|
| **Thread** | Eng kichik bajarilish birligi; bitta ma'lumot elementiga ishlaydi |
| **Warp** | 32 ta thread guruhi; bir vaqtda bir xil instruksiyani bajaradi (SIMT) |
| **Block** | Warp'lar to'plami; umumiy **shared memory** va sinxronizatsiyaga ega |
| **Grid** | Barcha block'lar; butun masalani qamrab oladi |

**💡 Tushuncha:** Dasturchi "har bir element uchun bitta thread yozaman" deb o'ylaydi (masalan, matritsa har katagi uchun). GPU o'zi bu thread'larni warp/block'larga taqsimlab, mavjud core'larga joylaydi.

## Throughput vs Latency

- **Latency** — bitta vazifa qancha tez tugaydi (CPU maqsadi).
- **Throughput** — vaqt birligida qancha vazifa tugaydi (GPU maqsadi).

GPU bitta thread'ni tez tugatishga urinmaydi. Buning o'rniga, thread xotiradan ma'lumot kutayotganda (latency), GPU shu zahoti **boshqa warp**ga o'tadi. Minglab thread borligi sababli har doim ishga tayyor thread topiladi — shunday qilib GPU xotira kutishini "yashiradi" (**latency hiding**) va core'larni band tutadi.

```
   CPU: cache miss bo'lsa → core kutadi (bo'sh turadi)
   GPU: warp kutsa → boshqa warp'ga o'tadi → core hech qachon bo'sh turmaydi
```

## Qaysi masalalar GPU'ga mos

**✅ GPU'ga mos (parallel, mustaqil, bir xil hisob):**

| Masala | Nega mos |
|--------|----------|
| Grafika / rendering | Har piksel mustaqil |
| Matritsa ko'paytirish | Ko'p mustaqil ko'paytma-qo'shma |
| ML / Deep Learning | Neyron tarmoq = ulkan matritsa amallar |
| Kriptovalyuta mining | Ko'p mustaqil hash |
| Rasm/video qayta ishlash | Har piksel ustida bir xil filtr |
| Ilmiy simulyatsiya | Ko'p mustaqil hujayra hisobi |

**⚠️ Ehtiyot bo'l — GPU'ga MOS EMAS:**

- **Ketma-ket** masalalar: har qadam oldingisiga bog'liq (masalan, oddiy rekursiya, qayta ishlashda kuchli data dependency).
- **Branch-heavy** kod: ko'p `if/else`, har thread turli yo'lda — warp divergence.
- **Kichik** masalalar: ma'lumotni GPU'ga ko'chirish narxi hisobdan qimmatroq bo'lib qoladi.

## VRAM va PCIe bottleneck

GPU o'zining alohida xotirasiga ega — **VRAM** (Video RAM, masalan GDDR6 yoki HBM). U juda tez (bandwidth CPU RAM'dan ancha yuqori), lekin CPU RAM'idan **alohida**.

Muammo: ma'lumot avval CPU RAM'da bo'ladi, uni GPU ustida ishlash uchun **VRAM'ga ko'chirish** kerak, natijani esa qaytarish kerak. Bu ko'chirish **PCIe** bus orqali ketadi va sekin — bu **bottleneck**.

```
   ┌──────────┐   PCIe (sekin)   ┌──────────┐
   │   CPU    │ ───────────────► │   GPU    │
   │  + RAM   │ ◄─────────────── │  + VRAM  │
   └──────────┘   ma'lumot        └──────────┘
                  ko'chirish
   RAM↔VRAM ko'chirish = eng qimmat qadam!
```

**⚠️ Ehtiyot bo'l:** GPU hisob juda tez bo'lsa ham, agar dastur doim RAM↔VRAM ko'chirsa — umumiy tezlik PCIe'ga bog'lanib qoladi. Qoida: ma'lumotni bir marta VRAM'ga yuklab, u yerda iloji boricha ko'p ish bajaring, kam ko'chiring.

## Host va Device: CPU va GPU birga

GPU o'zi mustaqil ishlamaydi — CPU uni **boshqaradi**. Terminologiya:

- **Host** — CPU va uning RAM'i (dasturni boshqaradi).
- **Device** — GPU va uning VRAM'i (og'ir parallel ishni bajaradi).

Tipik oqim:

```
   1. Host (CPU): ma'lumotni tayyorlaydi
   2. Host → Device: ma'lumotni VRAM'ga ko'chiradi
   3. Host: GPU'da "kernel" (parallel funksiya) ishga tushiradi
   4. Device (GPU): minglab thread bilan hisoblaydi
   5. Device → Host: natijani RAM'ga qaytaradi
```

CPU og'ir ishni GPU'ga topshirib, o'zi boshqa vazifalar bilan davom etadi (asinxron).

## GPGPU, TPU va ML

**GPGPU** (General-Purpose computing on GPU) — GPU'ni grafikadan tashqari umumiy hisob uchun ishlatish. Buni CUDA (NVIDIA) yoki OpenCL kabi platformalar orqali dasturlanadi.

**TPU** (Tensor Processing Unit) — Google'ning maxsus chipi, faqat ML uchun (ayniqsa matritsa/tensor ko'paytirish). GPU umumiyroq, TPU esa tor, lekin ML'da yanada samarali.

**Nega ML uchun GPU?** Deep learning'ning yuragi — **matritsa ko'paytirish** (matmul). Neyron tarmoqning har qatlami minglab-millionlab mustaqil ko'paytma-qo'shmadan iborat. Bu aynan GPU eng yaxshi bajaradigan ish: ko'p, mustaqil, bir xil hisob. Shu sababli ML training CPU'da soatlab, GPU'da esa daqiqalarda ketishi mumkin.

---

## Savol-Javob

### ❓ GPU nima uchun paydo bo'lgan?

**✅ Javob:** 3D grafika uchun. Ekrandagi millionlab piksel bir-biridan mustaqil va bir xil hisobni talab qiladi — bu parallel ishga mos. CPU buni ketma-ket sekin bajargani uchun maxsus parallel chip — GPU yaratildi.

### ❓ GPU va CPU arxitekturasidagi asosiy farq nima?

**✅ Javob:** Transistor byudjeti qayerga ketishida. CPU'ni kamsonli lekin kuchli core, katta cache va murakkab control tashkil qiladi (latency uchun). GPU esa transistorlarni minglab oddiy ALU'ga sarflaydi (throughput uchun). CPU bitta vazifani tez, GPU ko'p vazifani umumiy tez bajaradi.

### ❓ SIMD nima?

**✅ Javob:** Single Instruction, Multiple Data — bitta instruksiya bir vaqtda ko'p ma'lumot ustida bajariladi. Masalan, to'rt sonli ikki vektorni bitta "qo'shish" komandasi bilan qo'shish.

### ❓ SIMT SIMD'dan nimasi bilan farq qiladi?

**✅ Javob:** SIMT (Single Instruction, Multiple Threads) — NVIDIA modeli. SIMD'ga o'xshaydi, lekin har "yo'l" alohida thread sifatida ko'riladi va o'z indeksi/ma'lumotiga ega. Bitta instruksiya warp (32 thread) ustida bir vaqtda ishlaydi, lekin thread'lar mustaqil ko'rinadi.

### ❓ Warp nima va warp divergence nima?

**✅ Javob:** Warp — 32 ta thread guruhi, bir vaqtda bir xil instruksiyani bajaradi. Warp divergence — warp ichidagi thread'lar `if/else`da turli yo'lga ketganda GPU ikkala yo'lni ketma-ket bajarishi; bu parallellikni buzadi va sekinlashtiradi.

### ❓ Thread, block va grid o'rtasidagi munosabat qanday?

**✅ Javob:** Thread — eng kichik birlik (bitta element). Warp — 32 thread. Block — warp'lar to'plami, umumiy shared memory va sinxronizatsiyaga ega. Grid — barcha block'lar, butun masalani qamrab oladi.

### ❓ Throughput-oriented va latency-oriented nima farqi?

**✅ Javob:** Latency-oriented (CPU) — bitta vazifani imkon qadar tez tugatish. Throughput-oriented (GPU) — vaqt birligida imkon qadar ko'p vazifa tugatish, bitta vazifa sekinroq bo'lsa ham.

### ❓ GPU xotira kutishini qanday "yashiradi"?

**✅ Javob:** Latency hiding orqali. Warp xotiradan ma'lumot kutayotganda GPU boshqa tayyor warp'ga o'tadi. Minglab thread borligi sababli core hech qachon bo'sh turmaydi.

### ❓ Qanday masalalar GPU'ga mos?

**✅ Javob:** Parallel, mustaqil, bir xil hisobni takrorlaydigan masalalar: grafika, matritsa ko'paytirish, ML/deep learning, kripto mining, rasm/video qayta ishlash, ilmiy simulyatsiya.

### ❓ Qanday masalalar GPU'ga mos emas?

**✅ Javob:** Ketma-ket (har qadam oldingisiga bog'liq), branch-heavy (ko'p if/else, warp divergence) va kichik masalalar (ko'chirish narxi hisobdan qimmat).

### ❓ VRAM nima va PCIe bottleneck nima?

**✅ Javob:** VRAM — GPU'ning o'z tez xotirasi (CPU RAM'idan alohida). Ma'lumotni RAM↔VRAM ko'chirish PCIe bus orqali ketadi va sekin. Agar dastur doim ko'chirsa, umumiy tezlik PCIe'ga bog'lanib qoladi — bu bottleneck.

### ❓ Host va device nima?

**✅ Javob:** Host — CPU va uning RAM'i (dasturni boshqaradi). Device — GPU va VRAM (og'ir parallel ishni bajaradi). CPU ma'lumotni VRAM'ga yuklaydi, kernel'ni ishga tushiradi, natijani qaytarib oladi.

### ❓ GPGPU nima?

**✅ Javob:** General-Purpose computing on GPU — GPU'ni grafikadan tashqari umumiy hisob uchun ishlatish, CUDA yoki OpenCL orqali dasturlanadi.

### ❓ TPU nima va GPU'dan farqi?

**✅ Javob:** Tensor Processing Unit — Google'ning ML uchun maxsus chipi (matritsa/tensor ko'paytirishga ixtisoslashgan). GPU umumiyroq, TPU tor lekin ML'da yanada samarali.

### ❓ Nega ML uchun GPU ishlatiladi?

**✅ Javob:** Deep learning yuragi — matritsa ko'paytirish, ya'ni minglab-millionlab mustaqil bir xil hisob. Bu aynan GPU eng yaxshi bajaradigan ish. Shu sababli training CPU'ga qaraganda GPU'da o'nlab/yuzlab marta tez.

### ❓ CUDA core va stream processor nima?

**✅ Javob:** GPU ichidagi eng kichik hisoblash birligi — bitta thread arifmetikasini bajaradi. NVIDIA "CUDA core", AMD "stream processor" deydi.

---

## Masalalar

> Yechimlar: [solutions/hardware/05-gpu.md](../solutions/hardware/05-gpu.md)

1. O'z so'zlaringiz bilan tushuntiring: nega GPU'da minglab oddiy core bor, CPU'da esa kam lekin kuchli core? Har birining afzalligi qayerda?

2. Quyidagi masalalarni "GPU'ga mos" yoki "GPU'ga mos emas" deb tasniflang va sababini yozing: (a) 4K rasmga blur filtri, (b) fibonachchi sonini rekursiv hisoblash, (c) 10000x10000 matritsa ko'paytirish, (d) foydalanuvchi kiritgan matnni parsing qilish.

3. Warp divergence nima ekanini `if (threadId % 2 == 0)` misolida tushuntiring. Bunday kodni qanday qayta yozib divergence'ni kamaytirish mumkin?

4. Bir dastur GPU'da hisobni 2ms'da bajaradi, lekin RAM↔VRAM ko'chirish 50ms oladi. Umumiy vaqt qancha va bottleneck qayerda? Buni qanday yaxshilash mumkin?

5. SIMD va SIMT o'rtasidagi farqni jadval ko'rinishida tushuntiring (kamida 3 nuqta).

6. Thread → warp → block → grid iyerarxiyasini o'zingizning ASCII diagrammangizda chizing va har darajaga bir jumla izoh bering.

7. "Latency hiding" GPU'da qanday ishlashini tushuntiring. Nega bu CPU'da bir xil darajada samarali emas?

8. Nega deep learning aynan GPU'da tez o'qitiladi? Neyron tarmoq va matritsa ko'paytirish o'rtasidagi bog'liqlik orqali tushuntiring.

---

← [Hardware bo'limiga qaytish](./README.md)
