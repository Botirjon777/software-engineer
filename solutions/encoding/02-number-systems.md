# Sanoq Sistemalari va Bitwise — Yechimlar

Har bir masala uchun to'liq yechim va o'zbekcha izoh.

---

## 1. toBases (binary, octal, hex)

```js
function toBases(n) {
  return {
    bin: n.toString(2),
    oct: n.toString(8),
    hex: n.toString(16),
  };
}

toBases(255); // => { bin: '11111111', oct: '377', hex: 'ff' }
toBases(10);  // => { bin: '1010', oct: '12', hex: 'a' }
```

**Izoh:** `Number.prototype.toString(radix)` sonni istalgan base'da (2–36) string'ga aylantiradi. radix 2 — binary, 8 — octal, 16 — hex. Bu eng to'g'ridan-to'g'ri usul.

---

## 2. binToDec (qo'lda, parseInt'siz)

```js
function binToDec(str) {
  let result = 0;
  for (const ch of str) {
    result = result * 2 + (ch === '1' ? 1 : 0);
  }
  return result;
}

binToDec('1101'); // => 13
binToDec('11111111'); // => 255
```

**Izoh:** Bu Horner usuli. Har bir bitda mavjud natijani 2 ga ko'paytirib (chapga surish), keyingi bitni qo'shamiz. `'1101'`: 0→1→3(11)→6(110)→13(1101). Bu pozitsion sistema formulasining samarali shakli.

---

## 3. double / half (bitwise)

```js
const double = n => n << 1;   // n * 2
const half   = n => n >> 1;   // Math.floor(n / 2)

double(5);  // => 10
half(20);   // => 10
half(7);    // => 3 (pastga yaxlitlash)
```

**Izoh:** Chapga surish (`<<`) har bir bitni bir pozitsiya chapga ko'chiradi — bu 2 ga ko'paytirishga teng. O'ngga surish (`>>`) — 2 ga bo'lib pastga yaxlitlash. Bu darslik (klassik) bitwise optimizatsiyasi.

---

## 4. parity (faqat bitwise)

```js
function parity(n) {
  return (n & 1) === 0 ? 'even' : 'odd';
}

parity(4); // => 'even'
parity(7); // => 'odd'
parity(0); // => 'even'
```

**Izoh:** Sonning eng oxirgi (least significant) biti toq sonlarda doim 1, juft sonlarda 0. `n & 1` shu bitni ajratadi: 0 → juft, 1 → toq. `%` operatoridan tezroq va aynan shu maqsadda ishlatiladi.

---

## 5. Flag tizimi (bitmask)

```js
const READ    = 1; // 0b001
const WRITE   = 2; // 0b010
const EXECUTE = 4; // 0b100

const addFlag    = (perms, flag) => perms | flag;
const removeFlag = (perms, flag) => perms & ~flag;
const hasFlag    = (perms, flag) => (perms & flag) !== 0;

let p = 0;
p = addFlag(p, READ);     // => 1
p = addFlag(p, WRITE);    // => 3 (read + write)
hasFlag(p, WRITE);        // => true
hasFlag(p, EXECUTE);      // => false
p = removeFlag(p, READ);  // => 2 (faqat write)
```

**Izoh:** Har bir flag — alohida bit. `|` (OR) bilan bit yoqamiz (qo'shish), `& ~flag` bilan bitni o'chiramiz (`~flag` shu bitdan boshqa hammasini 1 qiladi), `& flag` bilan bit yoqilgan-yoqilmaganini tekshiramiz. Bu Linux fayl ruxsatlari (`rwx`) bilan bir xil g'oya.

---

## 6. decToHex (qo'lda)

```js
function decToHex(n) {
  if (n === 0) return '0';
  const digits = '0123456789abcdef';
  let result = '';
  while (n > 0) {
    const rem = n % 16;
    result = digits[rem] + result;  // qoldiqni oldiga qo'shamiz
    n = Math.floor(n / 16);
  }
  return result;
}

decToHex(255); // => 'ff'
decToHex(47);  // => '2f'
decToHex(16);  // => '10'
```

**Izoh:** 16 ga bo'lib qoldiqlarni teskari tartibda yig'amiz. Qoldiq 0–15 — `digits` jadvalidan mos hex belgini olamiz (10→`a`, 15→`f`). Yangi qoldiqni doim natijaning **oldiga** qo'shganimiz uchun teskari o'qish avtomatik bo'ladi.

---

## 7. popcount (1 bitlar soni)

```js
function popcount(n) {
  let count = 0;
  while (n !== 0) {
    n &= (n - 1);  // eng o'ngdagi 1-bitni o'chiradi
    count++;
  }
  return count;
}

popcount(13); // => 3  (1101)
popcount(255); // => 8 (11111111)
popcount(0);  // => 0
```

**Izoh:** Bu Brian Kernighan algoritmi. `n & (n - 1)` operatsiyasi sonning eng o'ngdagi 1-bitini o'chiradi. Necha marta o'chirish kerak bo'lsa — shuncha 1-bit bor. Oddiy "har bitni tekshirish" usulidan tezroq, chunki faqat yoqilgan bitlar bo'yicha aylanadi.

---

## 8. XOR swap

```js
function swap(arr, i, j) {
  if (i === j) return;       // bir xil indeks bo'lsa, XOR 0 qiladi
  arr[i] ^= arr[j];
  arr[j] ^= arr[i];
  arr[i] ^= arr[j];
}

const a = [5, 9];
swap(a, 0, 1); // a => [9, 5]
```

**Izoh:** XOR xossasi: `x ^ y ^ y === x`. Uch qadamda vaqtinchalik o'zgaruvchisiz almashtirish bo'ladi. `i === j` tekshiruvi muhim — aks holda element o'zini-o'zi XOR qilib 0 bo'lib qoladi. Amalda oddiy destructuring (`[a[i], a[j]] = [a[j], a[i]]`) afzal, lekin XOR swap bitwise tushunchasini ko'rsatadi.

---

## 9. toTwosComplement (8-bit)

```js
function toTwosComplement(n) {
  // manfiy sonni 8-bit unsigned ko'rinishiga keltiramiz:
  const value = n < 0 ? (256 + n) : n;   // -5 -> 251
  return (value & 0xFF).toString(2).padStart(8, '0');
}

toTwosComplement(-5); // => '11111011'
toTwosComplement(5);  // => '00000101'
toTwosComplement(-1); // => '11111111'
toTwosComplement(-128); // => '10000000'
```

**Izoh:** 8-bit two's complement'da `-n` qiymati `256 - n` ga teng. `-5` → `256 - 5 = 251` → binary `11111011`. `& 0xFF` bilan faqat pastki 8 bitni saqlaymiz, `padStart(8, '0')` bilan to'liq 8 xona qilamiz. Eng chap bit 1 bo'lsa — manfiy son.

---

## 10. swapEndian (32-bit)

```js
function swapEndian(n) {
  return (
    ((n & 0x000000FF) << 24) |   // byte 0 -> byte 3
    ((n & 0x0000FF00) << 8)  |   // byte 1 -> byte 2
    ((n & 0x00FF0000) >>> 8) |   // byte 2 -> byte 1
    ((n & 0xFF000000) >>> 24)    // byte 3 -> byte 0
  ) >>> 0;                       // unsigned 32-bit qilib qaytaramiz
}

swapEndian(0x12345678).toString(16); // => '78563412'
swapEndian(0x000000FF).toString(16); // => 'ff000000'
```

**Izoh:** Har bir byte'ni mask (`& 0xFF...`) bilan ajratib, kerakli pozitsiyaga shift qilamiz: 1-byte 24 bit chapga, 4-byte 24 bit o'ngga va h.k. Hammasini `|` bilan birlashtiramiz. Oxirgi `>>> 0` natijani unsigned 32-bit deb talqin qiladi (aks holda eng yuqori bit yoqilsa JS manfiy son ko'rsatadi). Bu big-endian ↔ little-endian almashinishini qo'lda amalga oshiradi.

---

← [Encoding bo'limiga qaytish](../../encoding/README.md)
