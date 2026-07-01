# Kompyuter Arxitekturasi — Yechimlar

Bu fayl `hardware/01-computer-architecture.md` dagi masalalar yechimlarini o'z ichiga oladi.

## Mundarija

- [1-masala](#m1)
- [2-masala](#m2)
- [3-masala](#m3)
- [4-masala](#m4)
- [5-masala](#m5)
- [6-masala](#m6)
- [7-masala](#m7)
- [8-masala](#m8)

<a name="m1"></a>
## 1-masala

**Savol:** `x = 10`, `y = 20`, `z = x + y` ni assembly ko'rinishida yozing.

**✅ Javob:** Taxminan 5 ta instruksiya kerak (ikki qiymatni yuklash, qo'shish, natijani saqlash).

```text
   LOAD  R1, #10      ; 10 ni R1 register'ga (x)
   STORE x, R1        ; x ni xotiraga saqlash (ixtiyoriy)
   LOAD  R2, #20      ; 20 ni R2 register'ga (y)
   STORE y, R2        ; y ni xotiraga saqlash (ixtiyoriy)
   ADD   R3, R1, R2   ; R1 + R2 = R3  (10 + 20 = 30)
   STORE z, R3        ; natijani z ga saqlash
```

**💡 Tushuncha:** Minimal variantda register'da ushlab, faqat oxirida saqlash mumkin: `LOAD R1,#10 / LOAD R2,#20 / ADD R3,R1,R2 / STORE z,R3` — 4 ta instruksiya. Compiler odatda shunday optimallashtiradi.

<a name="m2"></a>
## 2-masala

**Savol:** von Neumann va Harvard taqqoslash.

**✅ Javob:**

| Mezon | von Neumann | Harvard |
|-------|-------------|---------|
| Xotira | Bitta (birlashgan) | Ikkita (alohida) |
| Bus | Bitta | Ikkita |
| Afzalligi | Sodda, arzon, moslashuvchan | Instr+data bir vaqtda o'qish, tezroq |
| Kamchiligi | Bottleneck (bitta bus) | Murakkab, qimmat |
| Afzalroq holat | Umumiy PC/server | Mikrokontroller, DSP, CPU cache |

von Neumann — soddaligi va arzonligi tufayli umumiy kompyuterlar uchun. Harvard — tezlik kritik bo'lган embedded/DSP tizimlarda. Amalda CPU'lar cache darajasida "modified Harvard" ishlatadi.

<a name="m3"></a>
## 3-masala

**Savol:** 7, 12, 25, 64, 100 ni binary'ga aylantiring.

**✅ Javob:**

| Decimal | Binary | Bit soni |
|---------|--------|----------|
| 7 | 0000 0111 | 3 (min) |
| 12 | 0000 1100 | 4 |
| 25 | 0001 1001 | 5 |
| 64 | 0100 0000 | 7 |
| 100 | 0110 0100 | 7 |

**💡 Tushuncha:** n ni ifodalash uchun kerakли minimal bit = ⌊log₂(n)⌋ + 1. Masalan 64 = 2⁶, shuning uchun 7 bit (100 0000).

<a name="m4"></a>
## 4-masala

**Savol:** `a + b` ni instruction cycle bosqichlari bilan yozing.

**✅ Javob:**

```text
   1. FETCH:     PC ko'rsatgan manzildagi "ADD" instruksiyasini
                 RAM'dan olib IR'ga joylash. Komponent: Control Unit, PC, IR.
   2. DECODE:    IR'dagi kod ochiladi — bu ADD, operand'lar R1 (a), R2 (b).
                 Komponent: Control Unit.
   3. EXECUTE:   ALU R1 + R2 ni hisoblaydi. Komponent: ALU, registers.
   4. WRITEBACK: Natija R3 (yoki accumulator)'ga yoziladi.
                 Komponent: registers.
```

Keyin PC keyingi instruksiyaga o'tadi va tsikl takrorlanadi.

<a name="m5"></a>
## 5-masala

**Savol:** Register 1 ns, RAM ~300 baravar sekin. RAM'dan qancha vaqt?

**✅ Javob:** RAM'dan ~300 ns (1 ns × 300). Bu farq muhim, chunki dastur ko'p marta xotiraga murojaat qiladi. Agar har amal RAM'ni kutsa, dastur ~300 baravar sekin bo'lardi. Shu sabab cache va register'lar mavjud — RAM'ga borishni kamaytirish uchun.

**💡 Tushuncha:** Yaxshi dastur ma'lumotni ketma-ket (cache-friendly) ishlatib, RAM murojaatini minimallashtiradi.

<a name="m6"></a>
## 6-masala

**Savol:** Abstraksiya qatlamlari transistordan dasturgacha.

**✅ Javob:**

```text
   1. Transistor      — elektr o'chirgichi (0/1 hosil qiladi).
   2. Logic gate      — transistorlardan AND/OR/NOT mantiqi.
   3. CPU (mikroarx.) — gate'lardan ALU, register, control unit.
   4. ISA             — CPU tushunadigan instruksiyalar to'plami.
   5. OS              — hardware'ni boshqaradi, resurs taqsimlaydi.
   6. Til/Compiler    — yuqori kodni machine code'ga aylantiradi.
   7. Dastur          — foydalanuvchi masalasini yechadi.
```

Har qatlam pastdagini yashiradi — bu murakkablikni boshqarish imkonini beradi.

<a name="m7"></a>
## 7-masala

**Savol:** Nega binary, decimal emas?

**✅ Javob:** Kamida ikki fizik sabab:
1. **Transistor faqat ikki holatli** — tok bor (1) / yo'q (0). Ikki holatni yaratish oson va tabiiy.
2. **Shovqinga chidamlilik** — ikki daraja o'rtasidagi keng oraliq shovqindan xatolikni kamaytiradi. 10 xil kuchlanish darajasida kichik shovqin ham 3 ni 4 deb o'qishga olib kelardi.

Qo'shimcha: binary mantiq (Boolean algebra) apparatda sodda va arzon amalga oshadi.

<a name="m8"></a>
## 8-masala

**Savol:** 1, 2, 4 bayt bilan nechta qiymat?

**✅ Javob:** Formula: **n bayt = 2^(8n) xil qiymat**.

| Hajm | Bit | Qiymatlar soni |
|------|-----|----------------|
| 1 bayt | 8 | 2⁸ = 256 |
| 2 bayt | 16 | 2¹⁶ = 65 536 |
| 4 bayt | 32 | 2³² = 4 294 967 296 |

**💡 Tushuncha:** 4 bayt (32 bit) — bu klassik `int` o'lchami; u ~4.3 milliardgacha musbat son yoki ~±2.1 milliard ishorali son ifodalaydi.

← [Hardware bo'limiga qaytish](../../hardware/README.md)
