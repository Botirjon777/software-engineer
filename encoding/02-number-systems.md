# Sanoq Sistemalari va Bitwise Operatorlar

Decimal, binary, hex va octal qanday ishlaydi, ular orasida qanday konversiya qilinadi va bitwise operatorlar bilan bit darajasida ishlash.

## Mundarija

- [Pozitsion sanoq sistemasi](#pozitsion-sanoq-sistemasi)
- [Asosiy sanoq sistemalari](#asosiy-sanoq-sistemalari)
- [Nega kompyuter binary ishlatadi](#nega-kompyuter-binary-ishlatadi)
- [Nega dasturchilar hex ishlatadi](#nega-dasturchilar-hex-ishlatadi)
- [Bit va byte](#bit-va-byte)
- [Konversiya usullari](#konversiya-usullari)
- [JS: literal, parseInt, toString](#js-literal-parseint-tostring)
- [Bitwise operatorlar](#bitwise-operatorlar)
- [Bitwise amaliy use-case'lar](#bitwise-amaliy-use-caselar)
- [Two's complement (manfiy sonlar)](#twos-complement-manfiy-sonlar)
- [Signed vs unsigned](#signed-vs-unsigned)
- [Integer overflow](#integer-overflow)
- [Endianness](#endianness)
- [Intervyu savollari](#intervyu-savollari)
- [Masalalar](#masalalar)

---

## Pozitsion sanoq sistemasi

Pozitsion (positional) sanoq sistemasida raqamning qiymati uning **pozitsiyasiga** bog'liq. Har bir pozitsiya — asosning (base) darajasi (power).

Decimal (base-10) `345` soni:

```text
3 * 10²  +  4 * 10¹  +  5 * 10⁰
= 300    +  40       +  5      = 345
```

Umumiy formula (base `b`): har bir raqam `dᵢ` ga `b^i` ko'paytiriladi (i — o'ngdan boshlab 0).

**💡 Tushuncha:** Har qanday sanoq sistemasini shu formula bilan decimal'ga aylantirish mumkin — faqat base'ni almashtirish kerak.

---

## Asosiy sanoq sistemalari

| Sistema | Base | Raqamlar | JS prefiksi |
|---------|------|----------|-------------|
| Binary (ikkilik) | 2 | 0–1 | `0b` |
| Octal (sakkizlik) | 8 | 0–7 | `0o` |
| Decimal (o'nlik) | 10 | 0–9 | (yo'q) |
| Hexadecimal (o'n oltilik) | 16 | 0–9, A–F | `0x` |

Hex'da A–F harflari 10–15 ni bildiradi:

| Hex | A | B | C | D | E | F |
|-----|---|---|---|---|---|---|
| Decimal | 10 | 11 | 12 | 13 | 14 | 15 |

Bir xil son turli sistemada:

| Decimal | Binary | Octal | Hex |
|---------|--------|-------|-----|
| 10 | 1010 | 12 | A |
| 16 | 10000 | 20 | 10 |
| 255 | 11111111 | 377 | FF |

---

## Nega kompyuter binary ishlatadi

- Tranzistor (transistor) ikki holatda barqaror: o'chiq (0) yoki yoniq (1).
- Ikki holatni ajratish elektr signali bilan **ishonchli** (10 holatni ajratishdan ko'ra kamroq xatoga moyil).
- Mantiqiy amallar (AND, OR, NOT) to'g'ridan-to'g'ri tranzistorlar bilan amalga oshiriladi.

**💡 Tushuncha:** "Bit" = binary digit (0 yoki 1). Kompyuterdagi har bir son, matn, rasm — oxir-oqibat bitlar to'plami.

---

## Nega dasturchilar hex ishlatadi

Binary uzun va o'qish qiyin (`11111111`). Hex — binary'ning ixcham ko'rinishi:

- **1 hex raqam = aniq 4 bit** (chunki 2^4 = 16).
- **1 byte = 2 hex raqam** (8 bit).

```text
1111 1111  (binary)
  F    F    (hex)
= 0xFF = 255
```

Shu sababli hex tez-tez ishlatiladi: rang kodlari (`#FF5733`), memory address (`0x7FFE`), MAC-address, byte dump, Unicode (`U+1F600`).

| Hex raqam | Binary (4 bit) |
|-----------|----------------|
| 0 | 0000 |
| 4 | 0100 |
| 8 | 1000 |
| A | 1010 |
| F | 1111 |

**💡 Tushuncha:** Binary↔hex konversiyasi oson — binary'ni 4 bitdan guruhlab, har guruhni bitta hex raqamga aylantirasiz (yoki teskari). Decimal orqali o'tish shart emas.

---

## Bit va byte

- **Bit** — eng kichik birlik (0 yoki 1).
- **Byte** — 8 bit. 1 byte 256 (2^8) har xil qiymatni (0–255) saqlaydi.
- Nibble — 4 bit (= 1 hex raqam).

| Birlik | Bit soni | Diapazon (unsigned) |
|--------|----------|---------------------|
| Bit | 1 | 0–1 |
| Nibble | 4 | 0–15 |
| Byte | 8 | 0–255 |
| Word (16-bit) | 16 | 0–65535 |
| 32-bit | 32 | 0–4 294 967 295 |

---

## Konversiya usullari

**Binary → Decimal** (har bir 1-bitning pozitsiya qiymatini qo'shish):

```text
1011 (binary)
= 1*8 + 0*4 + 1*2 + 1*1
= 8 + 0 + 2 + 1 = 11
```

**Decimal → Binary** (2 ga bo'lib, qoldiqlarni teskari o'qish):

```text
13 / 2 = 6  qoldiq 1
 6 / 2 = 3  qoldiq 0
 3 / 2 = 1  qoldiq 1
 1 / 2 = 0  qoldiq 1
Teskari o'qiymiz: 1101
13 = 1101 (binary)
```

**Binary → Hex** (o'ngdan 4 bitdan guruhlash):

```text
1101 0110  (binary)
  D    6
= 0xD6
```

**Hex → Binary** (har hex raqamni 4 bitga):

```text
0x2F
2 -> 0010
F -> 1111
= 0010 1111
```

**Hex → Decimal:**

```text
0x2F = 2*16 + 15*1 = 32 + 15 = 47
```

**Decimal → Hex** (16 ga bo'lib, qoldiqlarni teskari):

```text
47 / 16 = 2  qoldiq 15 (F)
 2 / 16 = 0  qoldiq 2
Teskari: 2F
47 = 0x2F
```

---

## JS: literal, parseInt, toString

**Literal (kodda to'g'ridan-to'g'ri yozish):**

```js
0b1101;  // => 13   (binary)
0o17;    // => 15   (octal)
0xFF;    // => 255  (hex)
255;     // => 255  (decimal)
// hammasi bir xil Number turi — faqat yozilishi har xil
```

**String → Number — `parseInt(str, radix)`:**

```js
parseInt('1101', 2);  // => 13
parseInt('FF', 16);   // => 255
parseInt('2F', 16);   // => 47
parseInt('17', 8);    // => 15
Number('0xFF');       // => 255 (prefiks bilan)
```

**Number → String — `toString(radix)`:**

```js
(13).toString(2);   // => '1101'
(255).toString(16); // => 'ff'
(255).toString(2);  // => '11111111'
(47).toString(8);   // => '57'

// nol bilan to'ldirib 8-bit binary:
(5).toString(2).padStart(8, '0'); // => '00000101'
```

**⚠️ Ehtiyot bo'l:** `parseInt('08')` (radix'siz) ba'zi muhitlarda muammo bo'lishi mumkin — doim radix bering: `parseInt('08', 10)`.

---

## Bitwise operatorlar

Bitwise operatorlar sonni 32-bit signed integer deb, har bir bit ustida amal bajaradi.

| Operator | Nomi | Tavsif |
|----------|------|--------|
| `&` | AND | ikkala bit 1 bo'lsa → 1 |
| `\|` | OR | kamida bitta bit 1 bo'lsa → 1 |
| `^` | XOR | bitlar farq qilsa → 1 |
| `~` | NOT | har bir bitni teskari |
| `<<` | Left shift | bitlarni chapga surish |
| `>>` | Right shift (signed) | o'ngga surish, ishorani saqlab |
| `>>>` | Unsigned right shift | o'ngga surish, chapdan 0 to'ldirib |

**AND `&`** — maskalash (kerakli bitlarni ajratish):

```text
  1100  (12)
& 1010  (10)
  ----
  1000  (8)
```
```js
12 & 10; // => 8
```

**OR `|`** — bitlarni yoqish:

```text
  1100
| 1010
  ----
  1110  (14)
```
```js
12 | 10; // => 14
```

**XOR `^`** — farqni topish / toggle:

```text
  1100
^ 1010
  ----
  0110  (6)
```
```js
12 ^ 10; // => 6
5 ^ 5;   // => 0  (bir xil son XOR => 0)
```

**NOT `~`** — barcha bitlarni teskari (natija: `~x === -(x+1)`):

```js
~5;  // => -6
~0;  // => -1
```

**Left shift `<<`** — har bir surish 2 ga ko'paytiradi:

```js
1 << 3;  // => 8   (1 * 2³)
5 << 1;  // => 10  (5 * 2)
```

**Right shift `>>`** (signed) — 2 ga bo'lish (pastga yaxlitlash), ishora saqlanadi:

```js
20 >> 2;  // => 5   (20 / 4)
-20 >> 2; // => -5  (ishora bit saqlanadi)
```

**Unsigned right shift `>>>`** — chapga 0 to'ldiradi:

```js
-1 >>> 0; // => 4294967295  (manfiy sonni unsigned 32-bit deb ko'rish)
```

---

## Bitwise amaliy use-case'lar

**1. Even/odd tekshirish** (eng oxirgi bit):

```js
const isEven = n => (n & 1) === 0;
isEven(4); // => true
isEven(7); // => false
```

**2. Tez ko'paytirish/bo'lish (2 darajasiga):**

```js
n << 1;  // n * 2
n << 3;  // n * 8
n >> 1;  // Math.floor(n / 2)
```

**3. Flag / bitmask** (bir nechta boolean'ni bitta songa jamlash):

```js
const READ    = 1;  // 0b001
const WRITE   = 2;  // 0b010
const EXECUTE = 4;  // 0b100

let perms = READ | WRITE;        // 0b011 = 3 (read + write)

// bor-yo'qligini tekshirish:
(perms & WRITE) !== 0;           // => true
(perms & EXECUTE) !== 0;         // => false

// flag qo'shish:
perms |= EXECUTE;                // => 7 (0b111)

// flag o'chirish:
perms &= ~WRITE;                 // WRITE'ni olib tashlash

// flag toggle:
perms ^= READ;
```

**4. Ikki o'zgaruvchini vaqtinchalik o'zgaruvchisiz almashtirish (XOR swap):**

```js
let a = 5, b = 9;
a ^= b; b ^= a; a ^= b;
// endi a = 9, b = 5
```

**💡 Tushuncha:** Bitmask — permissions, feature flag, holat (state) to'plamlarini ixcham va tez saqlash uchun keng ishlatiladi (masalan Linux fayl ruxsatlari `rwx`).

---

## Two's complement (manfiy sonlar)

Kompyuter manfiy sonni **two's complement** usulida saqlaydi. Bu eng keng tarqalgan usul.

**Manfiy sonni topish (n bitda):** musbat sonni binary'da yozib, barcha bitlarni teskari qilib (NOT), 1 qo'shasiz.

`-5` ni 8-bitda:

```text
  5      = 0000 0101
NOT      = 1111 1010   (bitlarni teskari)
+1       = 1111 1011
-5       = 1111 1011  (8-bit two's complement)
```

Tekshirish — eng yuqori (left) bit **sign bit**: 1 bo'lsa manfiy.

**Nega two's complement:**
- Qo'shish/ayirish bir xil sxema bilan ishlaydi (musbat va manfiyni ajratmasdan).
- Faqat bitta `0` bor (`-0` muammosi yo'q).
- 8-bitda diapazon: -128 dan +127 gacha.

```text
8-bit signed:
0000 0000 =    0
0111 1111 =  127  (eng katta musbat)
1000 0000 = -128  (eng kichik manfiy)
1111 1111 =   -1
```

---

## Signed vs unsigned

| | Signed (ishorali) | Unsigned (ishorasiz) |
|--|-------------------|----------------------|
| Manfiy son | Ha (two's complement) | Yo'q |
| 8-bit diapazon | -128 … 127 | 0 … 255 |
| 32-bit diapazon | -2³¹ … 2³¹-1 | 0 … 2³²-1 |
| Eng yuqori bit | Sign bit | Oddiy qiymat biti |

```js
// JS bitwise operatorlari 32-bit SIGNED deb ko'radi:
-1 | 0;    // => -1
// unsigned'ga aylantirish uchun >>> 0:
-1 >>> 0;  // => 4294967295 (unsigned 32-bit)
```

**💡 Tushuncha:** Bir xil byte'lar (`1111 1111`) signed'da `-1`, unsigned'da `255`. Qiymat o'zgarmaydi — faqat **talqin** (interpretation) o'zgaradi.

---

## Integer overflow

Overflow — son saqlanadigan bit sonidan oshib ketsa, "aylanib" (wrap around) ketishi.

```text
8-bit unsigned (max 255):
255 + 1 = 256 = 1 0000 0000  (9 bit!)
8-bit'ga sig'maydi -> eng chap bit yo'qoladi -> 0000 0000 = 0
```

```text
8-bit signed (max 127):
127 + 1 -> 1000 0000 = -128  (musbat'dan manfiyga sakrash!)
```

**JS xususiyati:** JS sonlari 64-bit double (IEEE 754). Butun sonlar uchun xavfsiz diapazon `Number.MAX_SAFE_INTEGER` = 2⁵³-1.

```js
Number.MAX_SAFE_INTEGER;        // => 9007199254740991
Number.MAX_SAFE_INTEGER + 1;    // => 9007199254740992
Number.MAX_SAFE_INTEGER + 2;    // => 9007199254740992 (!) aniqlik yo'qoladi

// katta butun sonlar uchun BigInt:
9007199254740991n + 2n;         // => 9007199254740993n (to'g'ri)
```

**⚠️ Ehtiyot bo'l:** Bitwise operatorlar JS'da 32-bit'ga "qisib qo'yadi" — `2**31 | 0` manfiy chiqadi. Katta sonlarda bitwise ishlatmang yoki `BigInt` (`&`, `|` BigInt bilan ham ishlaydi) dan foydalaning.

---

## Endianness

Endianness — bir necha byte'li sonni xotirada qaysi tartibda saqlash.

`0x12345678` (4 byte) ni saqlash:

| | Byte tartibi (past manzildan boshlab) |
|--|----------------------------------------|
| **Big-endian** | `12 34 56 78` (eng katta byte birinchi) |
| **Little-endian** | `78 56 34 12` (eng kichik byte birinchi) |

- **Big-endian** — "human-readable" tartib; tarmoq protokollarida (network byte order) ishlatiladi.
- **Little-endian** — x86/x64 protsessorlarda (Intel, AMD) ishlatiladi.

```js
// DataView bilan endianness'ni nazorat qilish:
const buf = new ArrayBuffer(4);
const view = new DataView(buf);
view.setUint32(0, 0x12345678, true);  // true = little-endian
new Uint8Array(buf); // => [0x78, 0x56, 0x34, 0x12]

view.setUint32(0, 0x12345678, false); // false = big-endian
new Uint8Array(buf); // => [0x12, 0x34, 0x56, 0x78]
```

**⚠️ Ehtiyot bo'l:** Binary ma'lumotni mashinalar o'rtasida almashganda (fayl, tarmoq) endianness'ni kelishish shart — aks holda son buzilib o'qiladi.

---

## Intervyu savollari

### ❓ Pozitsion sanoq sistemasi nima?

**✅ Javob:** Raqamning qiymati uning pozitsiyasiga bog'liq bo'lgan sistema. Har bir pozitsiya base'ning darajasiga (power) ko'paytiriladi. Masalan decimal `345` = 3·10² + 4·10¹ + 5·10⁰. Bir xil formula har qanday base uchun ishlaydi.

### ❓ Nega kompyuter binary ishlatadi?

**✅ Javob:** Tranzistor ikki holatda (o'chiq/yoniq) ishonchli ishlaydi; ikki holatni elektr signali bilan ajratish 10 holatdan ko'ra kamroq xatoga moyil. Mantiqiy amallar (AND/OR/NOT) ham to'g'ridan-to'g'ri tranzistorlar bilan bajariladi.

### ❓ Nega dasturchilar hex ishlatadi?

**✅ Javob:** Hex — binary'ning ixcham ko'rinishi: 1 hex raqam = aniq 4 bit, 1 byte = 2 hex raqam. Binary↔hex konversiya oson (4 bitdan guruhlash). Shuning uchun rang kodlari, memory address, byte dump, Unicode code point hex'da yoziladi.

### ❓ 1 byte nechta qiymatni saqlay oladi?

**✅ Javob:** 1 byte = 8 bit, 2⁸ = 256 har xil qiymat. Unsigned'da 0–255, signed'da (two's complement) -128 dan +127 gacha.

### ❓ Binary'ni hex'ga qanday aylantirasiz?

**✅ Javob:** Binary raqamni o'ngdan boshlab 4 bitdan guruhlab, har bir guruhni bitta hex raqamga aylantirasiz. Masalan `1101 0110` → `D6` → `0xD6`. Decimal orqali o'tish shart emas.

### ❓ JavaScript'da string'ni binary son sifatida qanday o'qiysiz?

**✅ Javob:** `parseInt(str, radix)` — `parseInt('1101', 2)` → 13, `parseInt('FF', 16)` → 255. Teskari yo'nalish `num.toString(radix)` — `(255).toString(2)` → `'11111111'`. Literal sifatida `0b`, `0o`, `0x` prefikslari ham bor.

### ❓ `&`, `|`, `^` operatorlari nima qiladi?

**✅ Javob:** `&` (AND) — ikkala bit 1 bo'lsa 1 (maskalash uchun). `|` (OR) — kamida bittasi 1 bo'lsa 1 (bit yoqish). `^` (XOR) — bitlar farq qilsa 1 (toggle/farq topish). Misol: `12 & 10 = 8`, `12 | 10 = 14`, `12 ^ 10 = 6`.

### ❓ `<<` va `>>` qanday matematik amalga teng?

**✅ Javob:** `n << k` = `n * 2^k` (chapga surish — ko'paytirish). `n >> k` = `Math.floor(n / 2^k)` (signed o'ngga surish — bo'lish, ishorani saqlab). Masalan `5 << 1 = 10`, `20 >> 2 = 5`.

### ❓ `>>` va `>>>` farqi nima?

**✅ Javob:** `>>` (signed/arithmetic shift) ishorani saqlaydi — chapdan sign bit bilan to'ldiradi, shuning uchun manfiy son manfiy qoladi. `>>>` (unsigned/logical shift) chapdan doim 0 to'ldiradi. `-1 >> 1 = -1`, lekin `-1 >>> 1 = 2147483647`.

### ❓ Bitmask nima va qayerda ishlatiladi?

**✅ Javob:** Bitmask — bir nechta boolean flag'ni bitta integer'ning bitlarida saqlash. `|` bilan flag qo'shasiz, `& flag` bilan tekshirasiz, `& ~flag` bilan o'chirasiz, `^` bilan toggle qilasiz. Permissions (Linux `rwx`), feature flag, holat to'plamlarida ishlatiladi — ixcham va tez.

### ❓ Sonni juft yoki toq ekanini bitwise bilan qanday tekshirasiz?

**✅ Javob:** `n & 1` — agar natija 0 bo'lsa juft (even), 1 bo'lsa toq (odd). Chunki eng oxirgi bit toq sonlarda har doim 1. `(n & 1) === 0` → juft.

### ❓ Two's complement nima va nega ishlatiladi?

**✅ Javob:** Manfiy sonni saqlash usuli: musbat sonning bitlarini teskari qilib (NOT), 1 qo'shasiz. Afzalligi: qo'shish/ayirish musbat va manfiy uchun bir xil sxemada ishlaydi, `-0` muammosi yo'q. Eng yuqori bit — sign bit (1 = manfiy).

### ❓ Signed va unsigned farqi nima?

**✅ Javob:** Signed manfiy sonlarni saqlaydi (two's complement), eng yuqori bit — sign bit; 8-bitda -128…127. Unsigned faqat musbat, butun byte qiymat sifatida; 8-bitda 0…255. Bir xil bitlar talqinга qarab har xil son: `1111 1111` signed `-1`, unsigned `255`.

### ❓ Integer overflow nima?

**✅ Javob:** Son saqlanadigan bit sonidan oshganda "aylanib" ketishi. 8-bit unsigned'da `255 + 1 = 0`, signed'da `127 + 1 = -128`. JS sonlari 64-bit double — `MAX_SAFE_INTEGER` (2⁵³-1) dan keyin aniqlik yo'qoladi; katta butun sonlar uchun `BigInt`.

### ❓ Endianness nima?

**✅ Javob:** Ko'p byte'li sonni xotirada qaysi tartibda saqlash. Big-endian — eng katta byte birinchi (`12 34 56 78`), tarmoq protokollarida. Little-endian — eng kichik byte birinchi (`78 56 34 12`), x86/x64 protsessorlarda. Binary ma'lumot almashganda kelishish shart.

### ❓ JS'da nega `2**31 | 0` manfiy chiqadi?

**✅ Javob:** Bitwise operatorlar operand'ni 32-bit **signed** integer'ga aylantiradi. `2³¹` 32-bit signed'da eng yuqori (sign) bitni yoqadi, shuning uchun u manfiy songa aylanadi. Katta sonlarda bitwise ishlatmaslik yoki `BigInt` ishlatish kerak.

---

## Masalalar

> Yechimlar: [../../solutions/encoding/02-number-systems.md](../../solutions/encoding/02-number-systems.md)

1. **(Oson)** Decimal sonni qabul qilib, uning binary, octal va hex ko'rinishlarini obyekt sifatida qaytaradigan `toBases(n)` funksiyasini yozing. Misol: `toBases(255)` → `{ bin: '11111111', oct: '377', hex: 'ff' }`.

2. **(Oson)** Binary string'ni (masalan `'1101'`) decimal songa aylantiruvchi `binToDec(str)` funksiyasini `parseInt`'siz, qo'lda (har bit qiymatini qo'shib) yozing.

3. **(Oson)** Berilgan sonni 2 ga bitwise yordamida ko'paytiradigan va bo'ladigan ikki funksiya (`double(n)`, `half(n)`) yozing.

4. **(O'rta)** Sonning juft yoki toq ekanini `% ` operatorisiz, faqat bitwise bilan aniqlaydigan `parity(n)` funksiyasini yozing (`'even'` yoki `'odd'` qaytarsin).

5. **(O'rta)** Bayroqlar (flag) tizimini yarating: `READ`, `WRITE`, `EXECUTE` konstantalari va `addFlag`, `removeFlag`, `hasFlag` funksiyalarini bitwise yordamida amalga oshiring.

6. **(O'rta)** Decimal sonni qo'lda (toString'siz) hex string'ga aylantiruvchi `decToHex(n)` funksiyasini yozing (16 ga bo'lib, qoldiqlarni teskari yig'ib).

7. **(O'rta)** Berilgan butun sonning ichida nechta bit 1 ekanini sanaydigan `popcount(n)` funksiyasini bitwise yordamida yozing. Misol: `popcount(13)` (1101) → 3.

8. **(Qiyin)** Vaqtinchalik o'zgaruvchisiz, faqat XOR yordamida ikki sonni almashtiradigan `swap(arr, i, j)` funksiyasini yozing.

9. **(Qiyin)** 8-bitli son uchun two's complement manfiy ko'rinishini hisoblaydigan `toTwosComplement(n)` funksiyasini yozing (`-5` → `'11111011'`).

10. **(Qiyin)** 32-bitli sonning byte'larini teskari tartibga keltirib endianness'ni almashtiradigan `swapEndian(n)` funksiyasini bitwise (shift + mask) yordamida yozing. Misol: `swapEndian(0x12345678)` → `0x78563412`.

---

← [Encoding bo'limiga qaytish](./README.md)
