# CPU (Protsessor) — Yechimlar

Bu fayl `hardware/02-cpu.md` dagi masalalar yechimlarini o'z ichiga oladi.

## Mundarija

- [1-masala](#m1)
- [2-masala](#m2)
- [3-masala](#m3)
- [4-masala](#m4)
- [5-masala](#m5)
- [6-masala](#m6)
- [7-masala](#m7)
- [8-masala](#m8)
- [9-masala](#m9)

<a name="m1"></a>
## 1-masala

**Savol:** 3.2 GHz CPU: cycle/s va instruksiya/s (4 cycle/instr).

**✅ Javob:**
- Cycle/sekund = 3.2 GHz = 3.2 × 10⁹ = **3 200 000 000 cycle/s**.
- Instruksiya/sekund = 3.2×10⁹ ÷ 4 = **0.8 × 10⁹ = 800 million instr/s**.

**💡 Tushuncha:** Bu pipelining'siz taxmin. Pipelining bilan har cycle'da bittaga yaqin instruksiya tugaydi, shuning uchun amalda ~3.2 milliard instr/s'ga yaqinlashadi.

<a name="m2"></a>
## 2-masala

**Savol:** 8 core, 4 GHz, IPC = 3. Maksimal instruksiya/s?

**✅ Javob:** 4×10⁹ × 8 × 3 = **96 × 10⁹ = 96 milliard instr/s** (nazariy).

Amalda erishilmaydi, chunki:
- Cache miss'lar CPU'ni kuttiradi.
- Barcha core doim 100% band emas (dastur parallel bo'lmasligi mumkin).
- Pipeline hazard va branch misprediction cycle yo'qotadi.
- Thermal throttling tezlikni pasaytiradi.

<a name="m3"></a>
## 3-masala

**Savol:** `z = a + b` da ALU, CU, register roli.

**✅ Javob:**

```text
   1. CU instruksiyani fetch qilib decode qiladi ("bu ADD, operand a va b").
   2. Registerlar: a → R1, b → R2 ga yuklanadi (yoki avvaldan turibdi).
   3. ALU R1 + R2 ni hisoblaydi.
   4. Natija R3 register'ga writeback qilinadi.
   5. CU keyingi qadamda z ni RAM'ga STORE qilishni boshqaradi.
```

CU boshqaradi, registerlar qiymat saqlaydi, ALU hisoblaydi.

<a name="m4"></a>
## 4-masala

**Savol:** CISC va RISC taqqoslash.

**✅ Javob:**

| Mezon | CISC | RISC |
|-------|------|------|
| Instruksiya soni | Ko'p | Kam |
| Uzunlik | O'zgaruvchan | Bir xil (fixed) |
| Murakkablik | Bitta instr ko'p ish | Bitta instr oz ish |
| Energiya | Ko'proq | Kamroq |
| Misol | x86 / x86-64 | ARM, RISC-V |

<a name="m5"></a>
## 5-masala

**Savol:** 5 bosqichli pipeline, 5 instruksiya. Cycle soni?

**✅ Javob:**
- **Pipelining'siz:** 5 instr × 5 bosqich = **25 cycle**.
- **Pipelining bilan:** birinchi instr 5 cycle'da tugaydi, keyin har cycle'da bittadan. Formula: bosqich + (instr − 1) = 5 + (5−1) = **9 cycle**.

Farq: 25 → 9 cycle. Pipelining bosqichlarni parallel bajarib, ~2.8 baravar tezlashtirdi.

**💡 Tushuncha:** Instruksiya ko'p bo'lganda tejash yanada oshadi: 1000 instr → 5+999 = 1004 cycle (pipelining'siz 5000 o'rniga).

<a name="m6"></a>
## 6-masala

**Savol:** 90% branch prediction, misprediction 15 cycle, 1000 branch.

**✅ Javob:**
- Misprediction soni = 1000 × 10% = 100 ta.
- Jami jarima = 100 × 15 = **1500 cycle**.

**💡 Tushuncha:** Agar aniqlik 95%'ga ko'tarilsa: 50 × 15 = 750 cycle — ikki baravar kam. Shuning uchun branch prediction aniqligini oshirish muhim.

<a name="m7"></a>
## 7-masala

**Savol:** Nega L1 kichik lekin tezroq?

**✅ Javob:** Tezlik, hajm va narx muvozanati:
- Kichik xotira fizik jihatdan tezroq (signal kamroq masofa yuradi, qidiruv soddaroq).
- Tez (SRAM) xotira transistor ko'p talab qiladi — qimmat, shuning uchun kam bo'ladi.
- L3 katta, lekin sekinroq va CPU markazidan uzoqroq.

Shuning uchun ierarxiya: kichik+tez (L1) → katta+sekin (L3/RAM). Bu tezlik va narx o'rtasidagi eng yaxshi kelishuv.

<a name="m8"></a>
## 8-masala

**Savol:** Dastur 1 core ishlatyapti, 8 core bor. Muammo va yechim.

**✅ Javob:** Muammo — dastur **single-threaded** (bir oqim), shuning uchun faqat 1 core'dan foydalanadi, qolgan 7 core bo'sh turadi.

Yechim: dasturni **multi-threaded** yozish — ishni mustaqil qismlarga bo'lib, har qismni alohida thread/core'ga berish (masalan parallel loop, thread pool, yoki async ishlov). Shunda 8 core parallel ishlaydi.

**⚠️ Ehtiyot bo'l:** Har vazifa ham parallellashmaydi — ketma-ket bog'liqliklar (dependencies) bo'lsa, parallellik cheklanadi (Amdahl qonuni).

<a name="m9"></a>
## 9-masala

**Savol:** 4→5 GHz, TDP oshadi. Muammo va thermal throttling.

**✅ Javob:** Muammolar:
- Ko'proq clock speed → ko'proq issiqlik (TDP oshadi).
- Sovutish yetmasa CPU qizib ketadi.
- Ko'proq energiya sarfi, ko'proq elektr talab.

**Thermal throttling** — CPU belgilangan haroratdan oshsa, o'zini himoya qilish uchun avtomatik ravishda clock speed'ini pasaytiradi. Natijada tezlik 5 GHz'da qolmay, tushib ketadi — ya'ni yaxshi sovutishsiz yuqori GHz'dan to'liq foyda ololmaysiz.

← [Hardware bo'limiga qaytish](../../hardware/README.md)
