# Character Encoding: ASCII, Unicode va UTF-8

Belgi (character) kompyuterda raqamga aylantirilib saqlanadi — bu jarayonni boshqaradigan qoidalar to'plami character encoding deyiladi.

## Mundarija

- [Belgi kompyuterda qanday saqlanadi](#belgi-kompyuterda-qanday-saqlanadi)
- [ASCII](#ascii)
- [Extended ASCII va Latin-1](#extended-ascii-va-latin-1)
- [Nega ASCII yetarli emas](#nega-ascii-yetarli-emas)
- [Unicode nima](#unicode-nima)
- [Character set vs encoding farqi](#character-set-vs-encoding-farqi)
- [UTF-8](#utf-8)
- [UTF-16 va UTF-32](#utf-16-va-utf-32)
- [Byte vs character farqi](#byte-vs-character-farqi)
- [BOM (Byte Order Mark)](#bom-byte-order-mark)
- [Encoding/decoding jarayoni](#encodingdecoding-jarayoni)
- [Mojibake](#mojibake)
- [JS amaliyot: TextEncoder, Buffer, code point](#js-amaliyot-textencoder-buffer-code-point)
- [Intervyu savollari](#intervyu-savollari)
- [Masalalar](#masalalar)

---

## Belgi kompyuterda qanday saqlanadi

Kompyuter faqat sonlar bilan ishlaydi — aniqrog'i, 0 va 1 (bit) bilan. Demak `A` harfini saqlash uchun unga bir son (number) biriktirish kerak. Bu sonni **code point** deyiladi, sonni esa bitlarga aylantirish qoidasi **encoding** deyiladi.

**💡 Tushuncha:** "Belgi" (character) — bu mavhum tushuncha (masalan, lotincha `A` harfi). U kompyuterda saqlanishi uchun avval **raqam** (code point) ga, keyin esa **byte**larga aylantiriladi. Bu ikki bosqichni aralashtirib yubormaslik kerak.

```text
'A'  --(character set)-->  65  --(encoding)-->  01000001
belgi                    code point              byte(lar)
```

---

## ASCII

**ASCII** (American Standard Code for Information Interchange) — eng eski va eng asosiy standart. 1963-yilda yaratilgan.

- **7-bit**: har bir belgi 7 bit bilan ifodalanadi.
- 2^7 = **128 belgi** (code point: 0–127).
- Har bir belgi 1 byte ichida saqlanadi (8-bit), eng yuqori bit (most significant bit) 0 bo'ladi.

ASCII belgilar ikki turga bo'linadi:

| Turi | Diapazon | Misol |
|------|----------|-------|
| **Control characters** (boshqaruv) | 0–31 va 127 | `\0` (null, 0), `\n` (newline, 10), `\t` (tab, 9), `\r` (carriage return, 13), DEL (127) |
| **Printable characters** (ko'rinadigan) | 32–126 | space (32), `0`–`9` (48–57), `A`–`Z` (65–90), `a`–`z` (97–122) |

ASCII jadvalidan parcha (decimal / hex / binary / belgi):

| Decimal | Hex  | Binary    | Belgi |
|---------|------|-----------|-------|
| 32      | 0x20 | 0100000   | space |
| 48      | 0x30 | 0110000   | `0`   |
| 57      | 0x39 | 0111001   | `9`   |
| 65      | 0x41 | 1000001   | `A`   |
| 90      | 0x5A | 1011010   | `Z`   |
| 97      | 0x61 | 1100001   | `a`   |
| 122     | 0x7A | 1111010   | `z`   |

**💡 Tushuncha:** Foydali ikki qoida — `'A'` = 65, `'a'` = 97 (farqi 32, ya'ni bitta bit). Raqamlar `'0'` = 48 dan boshlanadi.

```js
'A'.charCodeAt(0); // => 65
'a'.charCodeAt(0); // => 97
'0'.charCodeAt(0); // => 48
String.fromCharCode(65); // => 'A'

// kichik harfni katta harfga: 32 ni ayirish
String.fromCharCode('a'.charCodeAt(0) - 32); // => 'A'
```

---

## Extended ASCII va Latin-1

ASCII'da 1 byte (8 bit) bo'lsa ham, faqat 7 bit ishlatilgan. Qolgan 1 bit ozod — u bilan yana 128 belgi (code point 128–255) qo'shish mumkin. Bu **Extended ASCII** deyiladi.

Muammo: 128–255 oralig'i uchun bitta standart bo'lmagan. Har bir til/region o'z **code page**ini ishlatgan:

- **Latin-1** (ISO 8859-1) — G'arbiy Yevropa: `é`, `ñ`, `ü`, `ç` va h.k.
- **ISO 8859-5** — kirill alifbosi.
- **Windows-1251** — kirill (Windows).

```text
Latin-1'da:  0xE9 = 'é'
Windows-1251'da: 0xE9 = 'й'
```

**⚠️ Ehtiyot bo'l:** Bir xil byte (`0xE9`) turli code page'larda turli belgini bildiradi. Shu sababli faylni qaysi encoding bilan yozilganini bilmasdan o'qib bo'lmaydi — bu mojibake'ning asosiy sababi.

---

## Nega ASCII yetarli emas

- ASCII faqat ingliz alifbosini qamraydi (128 belgi).
- O'zbek (`ў`, `ғ`, `қ`, `ҳ`), rus, arab, xitoy, yapon alifbolari sig'maydi.
- Emoji (`😀`), matematik belgilar (`∑`, `π`), valyutalar (`€`, `₿`) yo'q.
- Extended ASCII muammoni hal qilmadi, chunki har xil code page'lar bir-biriga mos kelmasdi va bari bir 256 belgi ham xitoy tiliga (minglab ieroglif) yetmasdi.

**Yechim** — barcha tillarni bitta standartga jamlash: **Unicode**.

---

## Unicode nima

**Unicode** — dunyodagi (deyarli) barcha tildagi belgilarni qamragan yagona **character set** (belgilar to'plami). Har bir belgiga noyob raqam — **code point** beradi.

- Code point `U+` prefiksi va hex bilan yoziladi: **`U+0041`** = `A` (= decimal 65).
- Hozir 1.1 milliondan ortiq code point sig'imi bor (U+0000 dan U+10FFFF gacha).

```text
U+0041  = 'A'
U+0061  = 'a'
U+0608  = 'ؘ' (arab)
U+0444  = 'ф' (kirill)
U+1F600 = '😀' (emoji)
```

**BMP (Basic Multilingual Plane):** U+0000 dan U+FFFF gacha bo'lgan asosiy tekislik — eng ko'p ishlatiladigan belgilar (lotin, kirill, arab, ko'p CJK belgilar) shu yerda. U+FFFF dan yuqorisi **supplementary planes** deyiladi (emoji, kam uchraydigan ieroglif va h.k.).

**💡 Tushuncha:** Unicode — bu faqat "belgi → raqam" jadvali. U raqamni byte'ga **qanday** aylantirishni aytmaydi. Buni encoding (UTF-8, UTF-16, UTF-32) qiladi.

---

## Character set vs encoding farqi

Bu intervyuda eng ko'p chalkashtiriladigan tushuncha.

| | Character set (charset) | Encoding |
|--|------------------------|----------|
| Nima | Belgi ↔ raqam (code point) jadvali | Raqam ↔ byte qoidasi |
| Misol | Unicode, ASCII | UTF-8, UTF-16, UTF-32 |
| Javob beradi | "`😀` qaysi raqam?" → U+1F600 | "U+1F600 qaysi byte'lar?" → `F0 9F 98 80` |

**💡 Tushuncha:** ASCII — bu ham charset, ham encoding (ikkalasi ham, chunki code point to'g'ridan-to'g'ri byte). Unicode esa faqat charset — uning bir nechta encoding'i bor (UTF-8/16/32).

---

## UTF-8

**UTF-8** — Unicode code point'ni byte'larga aylantiruvchi eng mashhur encoding. Bugungi web'ning ~98% UTF-8 ishlatadi.

**Variable-length (o'zgaruvchan uzunlik):** belgi 1 dan 4 byte gacha joy egallaydi:

| Code point diapazoni | Byte soni | Byte pattern (binary) |
|---------------------|-----------|------------------------|
| U+0000 – U+007F (ASCII) | 1 | `0xxxxxxx` |
| U+0080 – U+07FF | 2 | `110xxxxx 10xxxxxx` |
| U+0800 – U+FFFF | 3 | `1110xxxx 10xxxxxx 10xxxxxx` |
| U+10000 – U+10FFFF | 4 | `11110xxx 10xxxxxx 10xxxxxx 10xxxxxx` |

`x` bitlariga code point'ning bitlari joylashtiriladi.

**Misol — `€` (U+20AC) ni kodlash:**

```text
U+20AC = 0010 0000 1010 1100  (binary, 14 bit)
3 byte kerak (U+0800–U+FFFF oralig'ida).
Pattern: 1110xxxx 10xxxxxx 10xxxxxx
Bitlarni joylashtiramiz:
  1110 0010  10 000010  10 101100
=  E2         82          AC
Natija: E2 82 AC
```

```js
new TextEncoder().encode('€'); // => Uint8Array [226, 130, 172] = 0xE2 0x82 0xAC
```

**Nega UTF-8 eng mashhur:**

1. **ASCII bilan backward-compatible** — har qanday ASCII matni (0–127) UTF-8'da aynan bir xil 1 byte bo'ladi. Eski ASCII fayl avtomatik to'g'ri UTF-8 fayldir.
2. **Joy tejaydi** — ingliz/kod matni uchun 1 byte (UTF-16/32 ko'proq joy oladi).
3. **Self-synchronizing** — boshlovchi byte (leading byte) pattern'idan belgi qancha byte ekanini bilib bo'ladi; oraliq byte'lar `10` bilan boshlanadi, shu sababli oqim o'rtasidan ham qayta sinxronlanish mumkin.
4. **Endianness muammosi yo'q** — byte tartibi qat'iy belgilangan.

---

## UTF-16 va UTF-32

**UTF-16:**
- Belgi **2 yoki 4 byte** (16-bit birliklar — "code unit").
- BMP belgilar (U+0000–U+FFFF) → 1 ta code unit (2 byte).
- BMP dan tashqari (emoji va h.k.) → **surrogate pair** (2 ta code unit = 4 byte).

**Surrogate pair:** U+FFFF dan katta code point ikkita maxsus 16-bit qiymatga bo'linadi:
- High surrogate: `0xD800`–`0xDBFF`
- Low surrogate: `0xDC00`–`0xDFFF`

```text
'😀' U+1F600 = surrogate pair: D83D DE00
```

**💡 Tushuncha:** JavaScript string'lari ichki holatda UTF-16 ishlatadi. Shuning uchun emoji `.length` da 2 chiqadi (pastga qarang).

**UTF-32:**
- Har bir belgi qat'iy **4 byte** (fixed-length).
- Oddiy: code point = qiymat (konversiya yo'q).
- Lekin juda ko'p joy oladi (ASCII harf ham 4 byte) — shuning uchun kam ishlatiladi.

| Encoding | Byte/belgi | ASCII compat | Asosiy ishlatilishi |
|----------|-----------|--------------|---------------------|
| UTF-8 | 1–4 (variable) | Ha | Web, fayllar, API |
| UTF-16 | 2 yoki 4 | Yo'q | JS/Java/.NET ichki string |
| UTF-32 | 4 (fixed) | Yo'q | Ichki qayta ishlash (kamdan-kam) |

---

## Byte vs character farqi

Bu — JS dasturchilar uchun klassik xato manbai.

```js
'A'.length;   // => 1
'😀'.length;  // => 2  (!) surrogate pair sababli

// byte uzunligi yana boshqacha:
new TextEncoder().encode('😀').length; // => 4 (UTF-8 da 4 byte)
```

`'😀'.length === 2` chunki JS string uzunligini **UTF-16 code unit** soni bilan hisoblaydi, character soni bilan emas.

**To'g'ri belgi (code point) sonini olish:**

```js
[...'😀'].length;          // => 1  (spread/iterator code point bo'yicha yuradi)
Array.from('😀').length;   // => 1

// index bo'yicha yurish surrogate'ni buzadi:
'😀'[0]; // => '\uD83D' (yarim belgi, buzilgan)
```

**⚠️ Ehtiyot bo'l:** String'ni `str[i]` yoki `for (let i=0...)` bilan kesib (substring/slice) bo'lsangiz, emoji yoki BMP-tashqari belgini ikkiga bo'lib qo'yishingiz mumkin. Code point bilan ishlash uchun `for...of` yoki spread ishlatish kerak.

---

## BOM (Byte Order Mark)

**BOM** — faylning boshiga qo'yiladigan maxsus belgi (U+FEFF). Ikki vazifasi bor:

1. Encoding'ni belgilash (bu fayl UTF-8/16/32 mi).
2. UTF-16/32 da **byte order** (little vs big endian) ni ko'rsatish.

| Encoding | BOM byte'lari |
|----------|---------------|
| UTF-8 | `EF BB BF` |
| UTF-16 BE | `FE FF` |
| UTF-16 LE | `FF FE` |
| UTF-32 LE | `FF FE 00 00` |

**⚠️ Ehtiyot bo'l:** UTF-8'da BOM odatda **kerak emas** (UTF-8 da byte order muammosi yo'q). Ammo ba'zi muharrirlar (masalan eski Windows Notepad) BOM qo'shib qo'yadi — natijada fayl boshida ko'rinmas `﻿` paydo bo'ladi va JSON.parse, shell script (`#!`) yoki CSV o'qishda xato beradi.

```js
const text = '﻿hello';
text.charCodeAt(0); // => 65279 (0xFEFF) — ko'rinmas BOM!
text.replace(/^﻿/, ''); // BOM'ni tozalash
```

---

## Encoding/decoding jarayoni

```text
ENCODE:   "salom"  (string/character)  -->  [73 61 6C 6F 6D]  (byte[])
DECODE:   [73 61 6C 6F 6D]  (byte[])    -->  "salom"  (string)
```

- **Encode** = belgi → byte (yozish, yuborish, saqlash uchun).
- **Decode** = byte → belgi (o'qish, ko'rsatish uchun).

**Muhim qoida:** byte'lar o'zida encoding haqida ma'lumot saqlamaydi. Decode qilishda **qaysi encoding bilan encode qilinganini** bilishingiz shart. Aks holda mojibake chiqadi.

---

## Mojibake

**Mojibake** (yaponcha "文字化け" — "belgi o'zgarishi") — matn noto'g'ri encoding bilan decode qilinganda chiqadigan buzilgan, o'qib bo'lmaydigan belgilar.

Klassik misol: `Привет` so'zi UTF-8'da yozilib, Windows-1251 deb o'qilsa → `ÐŸÑ€Ð¸Ð²ÐµÑ‚`.

**Sabablari:**

1. Encode bir encoding'da, decode boshqa encoding'da (UTF-8 yozilgan, Latin-1 o'qilgan).
2. Double-encoding (matn ikki marta encode qilingan).
3. Database connection charset noto'g'ri (`latin1` bo'lib turgan ustunga UTF-8 yozilgan).
4. HTTP `Content-Type` header'da charset noto'g'ri yoki yo'q.
5. Faylni saqlashda va o'qishda har xil encoding ishlatilgan.

**⚠️ Ehtiyot bo'l:** Mojibake'ning oldini olish uchun: hamma joyda UTF-8 ishlatish, `Content-Type: text/html; charset=utf-8` ko'rsatish, DB ulanishini `utf8mb4` (MySQL) qilish, fayllarni doim UTF-8 (BOM'siz) saqlash.

---

## JS amaliyot: TextEncoder, Buffer, code point

**Brauzer / Node (standart) — TextEncoder/TextDecoder (faqat UTF-8):**

```js
const enc = new TextEncoder();           // doim UTF-8
const bytes = enc.encode('Salom 😀');    // => Uint8Array [...]
bytes.length;                            // => 10 (6 ASCII + 4 emoji)

const dec = new TextDecoder('utf-8');
dec.decode(bytes);                       // => 'Salom 😀'

// boshqa encoding'ni faqat decode qilish mumkin:
new TextDecoder('windows-1251').decode(someBytes);
```

**Node.js — Buffer (ko'p encoding qo'llab-quvvatlaydi):**

```js
const buf = Buffer.from('Salom', 'utf-8');
buf;                       // => <Buffer 53 61 6c 6f 6d>
buf.toString('hex');       // => '53616c6f6d'
buf.toString('base64');    // => 'U2Fsb20='
buf.length;                // => 5 (byte soni)

Buffer.from('U2Fsb20=', 'base64').toString('utf-8'); // => 'Salom'
```

**Code point bilan ishlash:**

```js
// charCodeAt — UTF-16 code unit (0–65535) qaytaradi:
'😀'.charCodeAt(0); // => 55357 (high surrogate, to'liq emas!)

// codePointAt — to'liq Unicode code point qaytaradi:
'😀'.codePointAt(0); // => 128512 (0x1F600) ✓

// code point -> string:
String.fromCodePoint(128512); // => '😀'
String.fromCodePoint(0x1F600); // => '😀'

// for...of code point bo'yicha yuradi (to'g'ri):
for (const ch of 'a😀b') console.log(ch); // => 'a', '😀', 'b'  (3 ta)

// index bo'yicha (noto'g'ri):
const s = 'a😀b';
for (let i = 0; i < s.length; i++) console.log(s[i]); // => 'a', '\uD83D', '\uDE00', 'b' (4 ta!)
```

**💡 Tushuncha:** Unicode-xavfsiz string ishlash uchun: hisoblashda `[...str].length`, kesishda spread/`Array.from`, code point uchun `codePointAt`/`fromCodePoint`, iteratsiyada `for...of`.

---

## Intervyu savollari

### ❓ Character set bilan encoding o'rtasidagi farq nima?

**✅ Javob:** Character set (charset) — bu belgi va raqam (code point) o'rtasidagi moslik jadvali (masalan, Unicode: `A` → U+0041). Encoding — bu raqamni (code point) byte'larga aylantirish qoidasi (masalan, UTF-8: U+0041 → `41`). Unicode — charset, UTF-8 esa uning encoding'i. ASCII esa ikkalasini ham o'zida birlashtirgan.

### ❓ ASCII nima va nechta belgi qamraydi?

**✅ Javob:** ASCII — 7-bitli encoding, 128 belgini (code point 0–127) qamraydi. 0–31 va 127 — control characters (`\n`, `\t`, null), 32–126 — printable (harflar, raqamlar, belgilar). Har bir belgi 1 byte ichida, eng yuqori bit 0 bilan saqlanadi.

### ❓ Nega ASCII yetarli emas?

**✅ Javob:** ASCII faqat ingliz alifbosini qamraydi. Boshqa tillar (kirill, arab, xitoy), emoji, maxsus belgilar sig'maydi. Extended ASCII (256 belgi) ham yetmaydi va har xil code page'lar bir-biriga mos kelmaydi. Shu sababli Unicode kerak bo'ldi.

### ❓ Unicode code point nima va qanday yoziladi?

**✅ Javob:** Code point — Unicode'da har bir belgiga berilgan noyob raqam. `U+` prefiksi va hex bilan yoziladi: `U+0041` (= `A`, decimal 65), `U+1F600` (= `😀`). Diapazon U+0000 dan U+10FFFF gacha.

### ❓ BMP nima?

**✅ Javob:** Basic Multilingual Plane — Unicode'ning U+0000–U+FFFF oralig'idagi asosiy tekisligi. Eng ko'p ishlatiladigan belgilar (lotin, kirill, arab, ko'p CJK) shu yerda. U+FFFF dan yuqorisi supplementary planes (emoji, kam uchraydigan belgilar), ular UTF-16'da surrogate pair talab qiladi.

### ❓ UTF-8 belgilarni qanday kodlaydi?

**✅ Javob:** Variable-length — code point qiymatiga qarab 1–4 byte ishlatadi. ASCII (U+0000–U+007F) → 1 byte (`0xxxxxxx`). Kattaroq code point'lar 2–4 byte: leading byte byte sonini bildiradi (`110…`, `1110…`, `11110…`), davom byte'lari `10…` bilan boshlanadi. Code point bitlari `x` o'rinlariga joylashtiriladi.

### ❓ Nega UTF-8 eng mashhur encoding?

**✅ Javob:** (1) ASCII bilan backward-compatible — eski ASCII fayllar avtomatik to'g'ri UTF-8. (2) Joy tejaydi — ingliz/kod matni 1 byte. (3) Self-synchronizing — byte oqimi o'rtasidan tiklanish mumkin. (4) Endianness muammosi yo'q — byte tartibi qat'iy. Web'ning ~98% UTF-8.

### ❓ UTF-8 ASCII bilan qanday backward-compatible?

**✅ Javob:** UTF-8'da U+0000–U+007F (ASCII diapazoni) belgilar aynan 1 byte bilan, ASCII'dagi bir xil qiymat bilan kodlanadi. Demak har qanday haqiqiy ASCII fayl bir vaqtning o'zida to'g'ri UTF-8 fayl hamdir — qayta konversiya kerak emas.

### ❓ UTF-16'da surrogate pair nima?

**✅ Javob:** UTF-16 BMP belgilarini 1 ta 16-bit code unit bilan ifodalaydi. U+FFFF dan katta code point'lar (emoji va h.k.) bitta 16-bit'ga sig'maydi — ular ikkita maxsus code unit'ga bo'linadi: high surrogate (`0xD800`–`0xDBFF`) va low surrogate (`0xDC00`–`0xDFFF`). Bu juftlik surrogate pair deyiladi.

### ❓ Nega `'😀'.length === 2` JavaScript'da?

**✅ Javob:** JS string'lari ichki holatda UTF-16. `.length` UTF-16 code unit sonini qaytaradi, character sonini emas. `😀` (U+1F600) BMP'dan tashqarida, shuning uchun surrogate pair (2 code unit) bilan saqlanadi → `.length` 2. To'g'ri belgi sonini olish uchun `[...'😀'].length` (= 1) ishlatish kerak.

### ❓ `charCodeAt` va `codePointAt` farqi nimada?

**✅ Javob:** `charCodeAt(i)` — i-pozitsiyadagi UTF-16 code unit'ni (0–65535) qaytaradi; surrogate pair'da faqat yarmini beradi. `codePointAt(i)` — to'liq Unicode code point'ni qaytaradi (surrogate pair'ni birlashtirib). `'😀'.charCodeAt(0)` → 55357 (noto'g'ri), `'😀'.codePointAt(0)` → 128512 (to'g'ri).

### ❓ BOM nima va qachon muammo tug'diradi?

**✅ Javob:** BOM (Byte Order Mark, U+FEFF) — fayl boshidagi maxsus belgi; encoding va (UTF-16/32 uchun) byte order'ni bildiradi. UTF-8'da byte order muammosi yo'q, shuning uchun BOM kerak emas. Ammo ba'zi muharrirlar UTF-8'ga BOM (`EF BB BF`) qo'shadi — bu ko'rinmas `﻿` JSON.parse, shell `#!`, CSV o'qishda xatoga olib keladi.

### ❓ Mojibake nima va sabablari?

**✅ Javob:** Mojibake — matn noto'g'ri encoding bilan decode qilinganda chiqadigan buzilgan belgilar (`Привет` → `ÐŸÑ€Ð¸Ð²ÐµÑ‚`). Sabablari: encode va decode encoding'lari mos kelmasligi, double-encoding, DB connection charset xatosi, HTTP `Content-Type` charset yo'qligi yoki noto'g'riligi. Oldini olish: hamma joyda UTF-8.

### ❓ Encode va decode farqi nima?

**✅ Javob:** Encode — string (belgi) ni byte'larga aylantirish (saqlash/yuborish uchun). Decode — byte'larni string'ga qayta aylantirish (o'qish/ko'rsatish uchun). Byte'lar o'zida encoding ma'lumotini saqlamaydi, shuning uchun decode qilishda qaysi encoding ishlatilganini bilish shart.

### ❓ String'ni xavfsiz belgi-belgi qayta ishlash uchun nima qilasiz?

**✅ Javob:** `str[i]` yoki index bo'yicha tsikl surrogate pair'ni buzadi (yarim emoji). To'g'ri yo'l: `for...of` (code point bo'yicha yuradi), `[...str]` yoki `Array.from(str)` (spread), uzunlik uchun `[...str].length`, code point uchun `codePointAt`. Bu Unicode-xavfsiz ishlashni ta'minlaydi.

### ❓ JavaScript'da matnni byte'larga aylantirish uchun nima ishlatasiz?

**✅ Javob:** Brauzer/Node'da `TextEncoder` (faqat UTF-8) — `new TextEncoder().encode(str)` → `Uint8Array`. Teskari yo'nalish `TextDecoder` (boshqa encoding'larni ham decode qila oladi). Node.js'da `Buffer` ko'p encoding'ni qo'llab-quvvatlaydi: `Buffer.from(str, 'utf-8')`, `buf.toString('hex'/'base64')`.

---

## Masalalar

> Yechimlar: [../../solutions/encoding/01-character-encoding.md](../../solutions/encoding/01-character-encoding.md)

1. **(Oson)** Berilgan belgi uchun uning ASCII code point'ini (decimal) va hex ko'rinishini chiqaradigan funksiya yozing. Misol: `info('A')` → `{ decimal: 65, hex: '0x41' }`.

2. **(Oson)** Faqat ASCII printable belgilar (32–126) dan iborat string'ni tekshiradigan `isAscii(str)` funksiyasini yozing.

3. **(Oson)** Kichik harfli string'ni ASCII arifmetikasi yordamida (charCodeAt + 32 ayirish) katta harfga aylantiruvchi `toUpper(str)` yozing — `toUpperCase()` ishlatmasdan.

4. **(O'rta)** String'dagi haqiqiy belgilar (code point) sonini to'g'ri qaytaruvchi `charCount(str)` yozing, shunda `charCount('a😀b') === 3`.

5. **(O'rta)** Bitta Unicode code point (masalan `0x1F600`) ni UTF-8 byte massiviga qo'lda (TextEncoder'siz) kodlaydigan `encodeUtf8(codePoint)` funksiyasini yozing.

6. **(O'rta)** String'ni boshidagi BOM (`﻿`) belgisini olib tashlaydigan `stripBom(str)` funksiyasini yozing, qolgan holatlarda string'ni o'zgarishsiz qaytarsin.

7. **(Qiyin)** UTF-8 byte massivini qabul qilib, undagi belgilar (code point) sonini sanaydigan `countCodePoints(bytes)` funksiyasini yozing — leading byte pattern'lariga qarab (continuation byte'larni `10…` sanamasdan).

8. **(Qiyin)** Mojibake'ni simulyatsiya qiling: bir string'ni UTF-8'da encode qilib, keyin uni Latin-1 (har bir byte = 1 belgi) deb decode qiluvchi funksiya yozing va natijani ko'rsating.

9. **(Qiyin)** Surrogate pair'ni qo'lda hisoblang: berilgan code point (> 0xFFFF) uchun high va low surrogate qiymatlarini hisoblaydigan `toSurrogatePair(cp)` funksiyasini yozing.

10. **(Qiyin)** String'ni berilgan maksimal **byte** uzunligiga (UTF-8) ko'ra kesadigan, lekin hech qachon belgini (multi-byte) yarmidan bo'lmaydigan `truncateBytes(str, maxBytes)` funksiyasini yozing.

---

← [Encoding bo'limiga qaytish](./README.md)
