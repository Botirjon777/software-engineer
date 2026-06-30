# Data Encoding (Base64, Hex, URL encoding)

Binary ma'lumotni — rasm, fayl, shifrlangan baytlar — faqat matn (text) tashiy oladigan kanallar orqali uzatish kerak bo'lganda **data encoding** ishga tushadi. Bu bo'limda encoding nima ekani va nega kerakligi, **Base64** qanday ishlashi (3 byte → 4 char), **Base64URL**, **hex** va **URL/percent encoding**, hamda eng muhimi — encoding'ni encryption, compression va hashing'dan ajratish ko'rib chiqiladi. Oxirida Node.js'dagi amaliy vositalar (`Buffer`, `btoa`/`atob`, `TextEncoder`) va real use-case'lar bor.

## Mundarija

- [Encoding nima va nega kerak](#encoding-nima-va-nega-kerak)
- [Base64 qanday ishlaydi](#base64-qanday-ishlaydi)
- [Base64 qadama-qadam: "Man"](#base64-qadama-qadam-man)
- [Nega Base64 hajmni ~33% oshiradi](#nega-base64-hajmni-33-oshiradi)
- [Padding (=) nima uchun](#padding--nima-uchun)
- [Base64URL — URL-safe variant](#base64url--url-safe-variant)
- [Hex encoding](#hex-encoding)
- [URL / percent encoding](#url--percent-encoding)
- [encodeURIComponent vs encodeURI](#encodeuricomponent-vs-encodeuri)
- [Encoding vs Encryption vs Compression vs Hashing](#encoding-vs-encryption-vs-compression-vs-hashing)
- [JS/Node: btoa, atob va ularning cheklovi](#jsnode-btoa-atob-va-ularning-cheklovi)
- [Node: Buffer va TextEncoder](#node-buffer-va-textencoder)
- [Amaliy use-case'lar](#amaliy-use-caselar)
- [Intervyu savollari (Q&A)](#intervyu-savollari-qa)
- [Masalalar](#masalalar)

---

## Encoding nima va nega kerak

**💡 Tushuncha:** **Encoding** (kodlash) — bu ma'lumotni bir ko'rinishdan boshqa, ma'lum bir kanal yoki format uchun *xavfsiz* ko'rinishga o'tkazish. Data encoding kontekstida ko'pincha bu **binary ma'lumotni matn (ASCII text) ko'rinishiga** o'tkazishni anglatadi.

Nega kerak? Chunki ko'p kanal va formatlar **binary-safe emas** — ya'ni ixtiyoriy baytlarni (0x00, 0xFF kabi) buzilmasdan tashiy olmaydi:

- **Email (SMTP)** — tarixan faqat 7-bitli ASCII matn uchun mo'ljallangan.
- **URL** — faqat ma'lum belgilar to'plamiga ruxsat beradi; bo'sh joy, `&`, `#` maxsus ma'noga ega.
- **JSON** — string ichida ixtiyoriy baytni saqlay olmaydi, faqat valid Unicode matn.
- **HTML/XML** — `<`, `>`, `&` strukturaviy belgilardir.

```text
  Binary fayl (rasm)        Matn kanali (email/JSON/URL)
  ┌──────────────┐          ┌─────────────────────────┐
  │ 0x89 0x50 ...│  ──────▶ │ "iVBORw0KGgoAAAANS..."   │
  │ (xom baytlar)│  encode  │ (xavfsiz ASCII matn)     │
  └──────────────┘          └─────────────────────────┘
```

**⚠️ Ehtiyot bo'l:** Encoding **maxfiylik bermaydi**. Base64 — bu "kodlangan" bo'lsa-da, **shifrlangan emas**; istalgan odam uni bir soniyada qaytaradi (decode qiladi). Encoding'ni hech qachon "xavfsizlik" (security) deb o'ylamang.

---

## Base64 qanday ishlaydi

**💡 Tushuncha:** **Base64** — binary ma'lumotni 64 ta xavfsiz ASCII belgi yordamida ifodalovchi encoding. Asosiy g'oya: baytlarni **6-bitlik** guruhlarga bo'lib, har guruhni alifbodagi bitta belgiga moslab qo'yamiz.

Nega 6 bit? Chunki 2⁶ = 64 — alifbodagi belgilar soni. Alifbo (standart):

```text
Index:  0..25  → A..Z
        26..51 → a..z
        52..61 → 0..9
        62     → +
        63     → /
```

Mexanika: **3 byte = 24 bit**, va 24 = 6 × 4. Demak har **3 byte** aynan **4 ta Base64 belgi**ga aylanadi.

```text
  3 byte (24 bit)  →  4 ta 6-bitlik guruh  →  4 ta belgi
  [ byte1 ][ byte2 ][ byte3 ]
  [ 6 ][ 6 ][ 6 ][ 6 ]
```

---

## Base64 qadama-qadam: "Man"

**💡 Tushuncha:** Klassik misol — `"Man"` so'zini Base64'ga o'tkazish.

1-qadam — har belgining ASCII kodi va binary'si:

```text
M = 77  = 01001101
a = 97  = 01100001
n = 110 = 01101110
```

2-qadam — 24 bitni ketma-ket yozib, 6-bitlik guruhlarga bo'lamiz:

```text
01001101 01100001 01101110     (3 byte = 24 bit)
010011 010110 000101 101110    (4 ta 6-bitlik guruh)
```

3-qadam — har guruhni o'nlik songa, so'ng alifbo belgisiga:

```text
010011 = 19 → T
010110 = 22 → W
000101 = 5  → F
101110 = 46 → u
```

Natija: `"Man"` → `"TWFu"`.

```js
Buffer.from('Man').toString('base64'); // => 'TWFu'
```

---

## Nega Base64 hajmni ~33% oshiradi

**💡 Tushuncha:** Base64 har **3 byte** (24 bit) ma'lumotni **4 byte** (4 ta ASCII belgi) qilib chiqaradi. 4 / 3 ≈ 1.333, ya'ni hajm taxminan **33% oshadi**.

```text
3 byte input  →  4 byte output   (har char 1 byte ASCII)
hajm = 4/3 ≈ 1.333×  →  +33%
```

**⚠️ Ehtiyot bo'l:** Shuning uchun katta fayllarni Base64'da JSON ichida yuborish samarasiz — 33% ortiqcha trafik. Iloji bo'lsa binary'ni alohida (multipart, binary upload) yuboring.

---

## Padding (=) nima uchun

**💡 Tushuncha:** Input uzunligi har doim ham 3'ga bo'linavermaydi. Qolgan 1 yoki 2 baytni ham 4-belgili blokka to'ldirish uchun oxiriga **`=` (padding)** qo'shiladi.

| Input baytlar | Output belgi | Padding |
|---|---|---|
| 3 byte | 4 char | yo'q |
| 2 byte | 3 char + `=` | bitta `=` |
| 1 byte | 2 char + `==` | ikkita `==` |

```js
Buffer.from('M').toString('base64');   // => 'TQ=='   (1 byte → 2 char + ==)
Buffer.from('Ma').toString('base64');  // => 'TWE='   (2 byte → 3 char + =)
Buffer.from('Man').toString('base64'); // => 'TWFu'   (3 byte → 4 char)
```

`=` belgisi decode paytida "bu yerda haqiqiy ma'lumot tugadi" degan signal. Base64 output uzunligi har doim 4'ga karrali bo'ladi.

---

## Base64URL — URL-safe variant

**💡 Tushuncha:** Standart Base64'dagi `+` va `/` belgilari URL'da maxsus ma'noga ega (`/` — yo'l ajratuvchi, `+` — ba'zan bo'sh joy). **Base64URL** ularni almashtiradi:

| Standart | Base64URL |
|---|---|
| `+` | `-` (minus) |
| `/` | `_` (pastki chiziq) |
| `=` padding | odatda **olib tashlanadi** |

```js
// Node 16+ : 'base64url' encoding bor
Buffer.from('>>>?').toString('base64');     // => 'Pj4+Pw=='   (+ bor)
Buffer.from('>>>?').toString('base64url');  // => 'Pj4-Pw'     (- ga aylandi, padding yo'q)
```

**⚠️ Ehtiyot bo'l:** **JWT** aynan Base64URL ishlatadi. Shu sababdan JWT token ichida `+`, `/`, `=` bo'lmaydi.

---

## Hex encoding

**💡 Tushuncha:** **Hex (hexadecimal) encoding** — har baytni **2 ta hex belgi** (`0-9`, `a-f`) bilan ifodalaydi. 1 byte = 8 bit = 2 ta 4-bitlik nibble = 2 hex char.

```text
M = 77  = 0x4D  → "4d"
a = 97  = 0x61  → "61"
n = 110 = 0x6E  → "6e"
"Man" → "4d616e"
```

```js
Buffer.from('Man').toString('hex'); // => '4d616e'
```

Hex **hajmni 2× oshiradi** (har byte → 2 char). Base64'dan ko'ra ko'p joy oladi, lekin o'qish/debug qilish oson va hash/checksum'larni ko'rsatishda standart.

| Encoding | 1 byte → necha char | Hajm | Tipik ishlatilishi |
|---|---|---|---|
| Hex | 2 char | +100% (2×) | hash, debug, ranglar (`#ff0000`) |
| Base64 | ~1.33 char | +33% | email, JSON, data URI |

---

## URL / percent encoding

**💡 Tushuncha:** **URL encoding** (percent encoding) — URL'da ruxsat etilmagan yoki maxsus belgilarni `%XX` ko'rinishida (XX — baytning hex kodi) ifodalaydi.

- Bo'sh joy → `%20`
- `&` → `%26`
- `?` → `%3F`
- O'zbekcha/Unicode harf → UTF-8 baytlarining har biri `%XX` (masalan `ў` → `%D1%9E`)

```js
encodeURIComponent('salom dunyo & =');
// => 'salom%20dunyo%20%26%20%3D'
```

**Ruxsat etilgan (kodlanmaydigan) belgilar** odatda: harflar, raqamlar va `- _ . ! ~ * ' ( )`.

---

## encodeURIComponent vs encodeURI

**💡 Tushuncha:** JS'da ikkita asosiy funksiya bor va ular **turlicha** belgilarni kodlaydi:

| Funksiya | Maqsad | `& / ? : @ = +` kabilarni kodlaydimi |
|---|---|---|
| `encodeURI` | **butun URL**ni kodlash | YO'Q (ular URL strukturasi uchun kerak) |
| `encodeURIComponent` | URL'ning **bitta bo'lagi**ni (query qiymati) kodlash | HA (chunki bo'lak ichida ular ma'lumot) |

```js
const q = 'a&b c';
encodeURI('https://x.com/?q=a&b c');
// => 'https://x.com/?q=a&b%20c'   ('&' saqlanib qoldi — buzilgan query!)

`https://x.com/?q=${encodeURIComponent(q)}`;
// => 'https://x.com/?q=a%26b%20c' ('&' kodlandi — to'g'ri)
```

**⚠️ Ehtiyot bo'l:** Query parametr **qiymatini** kodlaganda doim `encodeURIComponent` ishlating. `encodeURI`'ni faqat tayyor butun URL'ni "tozalash" uchun ishlating.

---

## Encoding vs Encryption vs Compression vs Hashing

**💡 Tushuncha:** Bu eng ko'p chalkashadigan va intervyuda ko'p so'raladigan farq. Hammasi ma'lumotni o'zgartiradi, lekin maqsad va xossalari butunlay boshqa.

| Xossa | Encoding | Encryption | Compression | Hashing |
|---|---|---|---|---|
| Maqsad | format moslash | maxfiylik | hajmni kichraytirish | barmoq izi / yaxlitlik |
| Kalit (key) kerakmi | YO'Q | HA | YO'Q | YO'Q (HMAC bundan mustasno) |
| Qaytariladimi | HA (decode) | HA (kalit bilan) | HA (decompress) | YO'Q (one-way) |
| Output hajmi | kattalashadi (Base64 +33%) | ~ teng yoki katta | kichrayadi | sobit (fixed) |
| Xavfsizlik beradimi | YO'Q | HA | YO'Q | qisman (integrity) |
| Misol | Base64, hex, URL | AES, RSA | gzip, zlib | SHA-256, MD5 |

```text
Encoding:    "Hi" ⇄ "SGk="          (kalitsiz, oson qaytadi)
Encryption:  "Hi" ⇄ "x9$@.."        (kalit bilan, qaytadi)
Compression: 1000 byte ⇄ 200 byte   (kichrayadi, qaytadi)
Hashing:     "Hi" → "3639..." ⇏     (qaytmaydi, sobit uzunlik)
```

**⚠️ Ehtiyot bo'l:** "Men parolni Base64'da saqladim, xavfsiz" — bu **xato**. Base64 himoya emas. Maxfiylik kerak bo'lsa — encryption; parol uchun — hashing (keyingi bo'lim).

---

## JS/Node: btoa, atob va ularning cheklovi

**💡 Tushuncha:** Brauzerda (va Node 16+) ikkita global funksiya bor:

- `btoa(str)` — string → Base64 (**b**inary **to** **a**scii)
- `atob(str)` — Base64 → string (**a**scii **to** **b**inary)

```js
btoa('Man');   // => 'TWFu'
atob('TWFu');  // => 'Man'
```

**⚠️ Ehtiyot bo'l (asosiy cheklov):** `btoa` har bir belgini **bitta bayt** (Latin-1, 0–255) deb hisoblaydi. Shuning uchun Unicode (masalan o'zbekcha yoki emoji) bilan **xato beradi**:

```js
btoa('Salom 😀');
// => InvalidCharacterError!  (😀 bir baytga sig'maydi)
```

To'g'ri yo'l — avval matnni UTF-8 baytlarga (`TextEncoder`) aylantirish:

```js
const bytes = new TextEncoder().encode('Salom 😀');
const b64 = btoa(String.fromCharCode(...bytes)); // endi ishlaydi
```

Yoki Node'da umuman `btoa` o'rniga `Buffer` ishlating (quyida).

---

## Node: Buffer va TextEncoder

**💡 Tushuncha:** Node.js'da binary bilan ishlashning **standart va to'g'ri** yo'li — `Buffer`. U Unicode'ni UTF-8 sifatida to'g'ri eplaydi.

```js
// string → base64 / hex
Buffer.from('Salom 😀', 'utf8').toString('base64'); // => 'U2Fsb20g8J+YgA=='
Buffer.from('Salom 😀', 'utf8').toString('hex');    // => '53616c6f6d20f09f9880'

// base64 / hex → string (qaytarish)
Buffer.from('U2Fsb20g8J+YgA==', 'base64').toString('utf8'); // => 'Salom 😀'
Buffer.from('53616c6f6d', 'hex').toString('utf8');          // => 'Salom'
```

`TextEncoder`/`TextDecoder` (platformaga bog'liq emas, standart Web API) — matn ⇄ `Uint8Array` (UTF-8 baytlar):

```js
const u8 = new TextEncoder().encode('Salom'); // Uint8Array(5) [83,97,108,111,109]
new TextDecoder().decode(u8);                 // => 'Salom'
```

| Vosita | Qayerda | UTF-8 / Unicode |
|---|---|---|
| `Buffer` | Node | To'g'ri eplaydi |
| `btoa`/`atob` | brauzer + Node | Latin-1 cheklovi (Unicode'da buziladi) |
| `TextEncoder` | hamma joyda | Faqat matn ⇄ baytlar (Base64 emas) |

---

## Amaliy use-case'lar

**💡 Tushuncha:** Base64/encoding qayerda real ishlatiladi:

- **Data URI** — rasmni HTML/CSS ichiga to'g'ridan-to'g'ri joylash:
  ```html
  <img src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUg..." />
  ```
- **JWT** — uchta Base64URL bo'lak: `header.payload.signature`. Payload Base64URL — **shifrlanmagan**, istalgan odam o'qiy oladi.
- **Basic Auth header** — `user:password` ni Base64'da:
  ```text
  Authorization: Basic dXNlcjpwYXNz   ("user:pass" → base64)
  ```
  Bu HTTPS'siz **xavfsiz emas** — Base64 oddiy decode qilinadi.
- **Binary in JSON** — fayl/rasm baytlarini JSON ichida string sifatida tashish uchun Base64.

```js
// Basic auth header yasash
const token = Buffer.from('user:pass').toString('base64');
const header = `Basic ${token}`; // => 'Basic dXNlcjpwYXNz'
```

**⚠️ Ehtiyot bo'l:** JWT payload va Basic Auth — ikkalasi ham *faqat encoding*, encryption emas. Ularni xavfsiz qiladigan narsa — **HTTPS** va (JWT uchun) **signature**, Base64'ning o'zi emas.

---

## Intervyu savollari (Q&A)

### ❓ Encoding nima va nega kerak?

**✅ Javob:** Encoding — ma'lumotni ma'lum kanal/format uchun mos ko'rinishga o'tkazish; data encoding'da odatda binary'ni matnga aylantirish. Kerak, chunki email (SMTP), URL, JSON kabi ko'p kanallar **binary-safe emas** — ixtiyoriy baytlarni buzilmasdan tashiy olmaydi. Base64/hex orqali binary'ni xavfsiz ASCII matnга o'tkazamiz.

### ❓ Base64 qanday ishlaydi (qisqacha)?

**✅ Javob:** Baytlarni 6-bitlik guruhlarga bo'ladi (3 byte = 24 bit = 4 × 6 bit), har guruhni 64-belgili alifbodagi (`A-Z a-z 0-9 + /`) belgiga moslaydi. Demak 3 byte → 4 char.

### ❓ Nega Base64 hajmni ~33% oshiradi?

**✅ Javob:** Har 3 byte (24 bit) 4 ta ASCII belgiga (4 byte) aylanadi. 4/3 ≈ 1.333, ya'ni +33%.

### ❓ Padding `=` nima uchun kerak?

**✅ Javob:** Input uzunligi 3'ga bo'linmasa, oxirgi blokni 4 belgiga to'ldirish va decode paytida ma'lumot chegarasini bildirish uchun. 2 byte → bitta `=`, 1 byte → ikkita `==`.

### ❓ Base64 va Base64URL farqi?

**✅ Javob:** Base64URL — URL-safe variant: `+` → `-`, `/` → `_`, padding `=` odatda olib tashlanadi. JWT shuni ishlatadi, chunki standart `+`/`/` URL'da maxsus ma'noga ega.

### ❓ Hex va Base64 — qaysi biri ko'proq joy oladi?

**✅ Javob:** Hex 2× (har byte 2 char, +100%), Base64 ~1.33× (+33%). Hex katta, lekin o'qish oson — shu uchun hash/checksum ko'rsatishda hex standart.

### ❓ encodeURI va encodeURIComponent farqi?

**✅ Javob:** `encodeURI` — butun URL uchun, `& / ? : @ =` kabi struktura belgilarini kodlamaydi. `encodeURIComponent` — URL'ning bitta bo'lagi (query qiymati) uchun, ularni ham kodlaydi. Query qiymatiga doim `encodeURIComponent`.

### ❓ Encoding bilan encryption farqi nima?

**✅ Javob:** Encoding kalitsiz, maxfiylik bermaydi, istalgan odam qaytaradi (Base64). Encryption kalit bilan, maxfiylik beradi, faqat kalit egasi qaytaradi (AES). Base64'ni "xavfsizlik" deb ishlatish — keng tarqalgan xato.

### ❓ Hashing encoding'dan nimasi bilan farq qiladi?

**✅ Javob:** Hashing — **one-way** (qaytmaydi), sobit uzunlikdagi output beradi, barmoq izi/integrity uchun. Encoding — qaytariladigan (two-way), format moslash uchun.

### ❓ `btoa`/`atob`ning asosiy cheklovi nima?

**✅ Javob:** Ular har belgini bitta bayt (Latin-1, 0–255) deb oladi, shuning uchun Unicode (o'zbekcha, emoji) bilan xato beradi. To'g'ri yo'l — avval `TextEncoder` bilan UTF-8 baytlarga o'tkazish yoki Node'da `Buffer` ishlatish.

### ❓ Node'da stringни base64'ga qanday aylantirasiz?

**✅ Javob:** `Buffer.from(str, 'utf8').toString('base64')`. Qaytarish: `Buffer.from(b64, 'base64').toString('utf8')`. `Buffer` Unicode'ni to'g'ri eplaydi.

### ❓ JWT payload'i shifrlanganmi?

**✅ Javob:** Yo'q. JWT payload — faqat **Base64URL encoding**, istalgan odam decode qilib o'qiy oladi. Uni himoya qiladigan narsa — signature (buzib bo'lmaydi) va HTTPS. Maxfiy ma'lumotni JWT payload'iga **qo'ymang**.

### ❓ Basic Auth header xavfsizmi?

**✅ Javob:** O'zi xavfsiz emas — `user:pass` shunchaki Base64'da, oson decode qilinadi. Faqat **HTTPS** uni himoya qiladi (yo'lda shifrlangan tunnel orqali).

### ❓ Katta faylni JSON'da Base64'da yuborish — yaxshi g'oyami?

**✅ Javob:** Odatda yo'q — Base64 +33% trafik qo'shadi. Iloji bo'lsa binary'ni alohida (multipart/form-data, binary upload) yuborish samaraliroq. Base64'ni kichik narsalar (kichik rasm, inline data) uchun ishlating.

### ❓ TextEncoder nima qiladi?

**✅ Javob:** Matnni (string) UTF-8 baytlarga (`Uint8Array`) aylantiradi (`TextDecoder` — teskari). U Base64 qilmaydi — faqat matn ⇄ baytlar konversiyasi; Base64 kerak bo'lsa keyin `Buffer` yoki `btoa` bilan birga ishlatiladi.

---

## Masalalar

> Yechimlar: [solutions/encoding/03-data-encoding.md](../solutions/encoding/03-data-encoding.md)

1. **"Sun" ni Base64'ga qo'lda.** `"Sun"` so'zini qadama-qadam (ASCII → binary → 6-bitlik guruhlar → belgilar) Base64'ga o'tkazing. Har qadamni yozing va natijani `Buffer.from('Sun').toString('base64')` bilan tekshiring.

2. **Padding tahlili.** `"A"`, `"AB"`, `"ABC"`, `"ABCD"` so'zlarini Base64'ga o'tkazing. Qaysilarида nechta `=` chiqadi va nima uchun? Output uzunligi har doim 4'ga karrali ekanini tushuntiring.

3. **Hex ⇄ Base64.** `"Hello"` so'zini avval `hex`, so'ng `base64` ko'rinishida chiqaring. Ikkala natijaning uzunligini (char soni) solishtiring va nega farq qilishini izohlang.

4. **Base64URL.** Shunday matn toping (yoki yasang)ki, uning standart Base64'ida `+` yoki `/` bo'lsin. So'ng uni `base64url` bilan ham chiqaring va qaysi belgilar o'zgarganini ko'rsating.

5. **Unicode muammosi.** `btoa('Assalomu alaykum 🌙')` ni ishga tushiring — nima bo'ladi? So'ng `TextEncoder` (yoki `Buffer`) yordamida to'g'ri Base64 hosil qiling va uni qayta matnga qaytaring.

6. **Query qurish.** Foydalanuvchi qidiruvi `"node & deno = ?"` bo'lsa, undан to'g'ri URL query yasang. `encodeURI` va `encodeURIComponent` ikkalasi bilan urinib ko'ring va qaysi biri to'g'ri ishlashini, nega ekanini ko'rsating.

7. **Basic Auth.** `username = "admin"`, `password = "p@ss:1"` uchun to'g'ri `Authorization: Basic ...` header qiymatini yasang. Parol ichidagi `:` belgisi muammo tug'diradimi? Header'ni decode qilib tekshiring.

8. **Farqlar jadvali.** O'z so'zlaringiz bilan encoding, encryption, compression va hashing ni 4 ta mezon bo'yicha (kalit kerakmi, qaytadimi, output hajmi, maqsad) qiyoslovchi jadval tuzing va har biriga bittadan misol keltiring.

9. **Data URI.** Kichik matnli "rasm" o'rniga oddiy matnni (`"hi"`) `data:text/plain;base64,...` data URI ko'rinishiga keltiring. URI'ni qo'lda yozib, brauzer manzil satrida ochilganda nima ko'rinishini tushuntiring.

10. **Round-trip funksiya.** `encode(str)` va `decode(b64)` funksiyalarini yozing (Node `Buffer` bilan), ular Unicode matnni ham to'g'ri eplasin. `decode(encode(s)) === s` ni bir nechta input (o'zbekcha, emoji, bo'sh string) uchun tekshiring.

← [Encoding bo'limiga qaytish](./README.md)
