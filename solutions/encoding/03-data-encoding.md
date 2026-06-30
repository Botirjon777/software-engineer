# Data Encoding — Yechimlar

Bu fayl [`encoding/03-data-encoding.md`](../../encoding/03-data-encoding.md) dagi "Masalalar" bo'limining yechimlarini izohlari bilan beradi. Avval mustaqil urinib ko'ring, keyin solishtiring.

---

## 1. "Sun" ni Base64'ga qo'lda

```text
S = 83  = 01010011
u = 117 = 01110101
n = 110 = 01101110

24 bit:        01010011 01110101 01101110
6-bitlik:      010100 110111 010101 101110
o'nlik:        20     55     21     46
alifbo:        U      3      V      u
```

```js
Buffer.from('Sun').toString('base64'); // => 'U3Vu'
```

**Izoh:** 3 byte → 4 char, padding yo'q (uzunlik 3'ga bo'linadi). Alifbo: 20→U, 55→3 (52–61 → 0–9, demak 55−52=3), 21→V, 46→u.

---

## 2. Padding tahlili

```js
Buffer.from('A').toString('base64');    // => 'QQ=='   (1 byte → 2 char + ==)
Buffer.from('AB').toString('base64');   // => 'QUI='   (2 byte → 3 char + =)
Buffer.from('ABC').toString('base64');  // => 'QUJD'   (3 byte → 4 char)
Buffer.from('ABCD').toString('base64'); // => 'QUJDRA==' (4 byte = 3+1 → 8 char, == oxirida)
```

**Izoh:** Base64 input'ni 3 baytlik bloklarga bo'ladi. Qoldiq 1 byte bo'lsa `==`, 2 byte bo'lsa `=`. `"ABCD"` = 3+1 byte: birinchi blok `QUJD`, qolgan 1 byte (`D`) → `RA==`. Output har doim 4'ga karrali, chunki har blok aniq 4 char chiqaradi.

---

## 3. Hex ⇄ Base64

```js
Buffer.from('Hello').toString('hex');    // => '48656c6c6f'   (10 char)
Buffer.from('Hello').toString('base64'); // => 'SGVsbG8='     (8 char)
```

**Izoh:** `"Hello"` — 5 byte. Hex: har byte 2 char → 10 char (+100%). Base64: 5 byte = 1 blok (3) + 2 byte qoldiq → 4 + 4 = 8 char (oxirgi blokda bitta `=`). Base64 zichroq, chunki u ~1.33× (hex 2×).

---

## 4. Base64URL

```js
// '>>>?' standart base64'ida + va / chiqaradi
Buffer.from('>>>?').toString('base64');    // => 'Pj4+Pw=='
Buffer.from('>>>?').toString('base64url'); // => 'Pj4-Pw'

// '?>?>' kabi /' chiqaruvchi misol:
Buffer.from([0xff, 0xff, 0xff]).toString('base64');    // => '////'
Buffer.from([0xff, 0xff, 0xff]).toString('base64url'); // => '____'
```

**Izoh:** `+` → `-`, `/` → `_`, padding `=` olib tashlanadi. `0xff 0xff 0xff` = 24 bit hammasi 1 → har 6-bitlik guruh `111111` = 63 → standartda `/`, URL'da `_`.

---

## 5. Unicode muammosi

```js
// btoa to'g'ridan-to'g'ri ishlamaydi:
// btoa('Assalomu alaykum 🌙'); // => InvalidCharacterError

// To'g'ri yo'l (brauzer / universal): TextEncoder
const bytes = new TextEncoder().encode('Assalomu alaykum 🌙');
const b64 = btoa(String.fromCharCode(...bytes));

// Qaytarish:
const back = new TextDecoder().decode(
  Uint8Array.from(atob(b64), c => c.charCodeAt(0))
);
back; // => 'Assalomu alaykum 🌙'

// Node'da soddaroq:
const b64n = Buffer.from('Assalomu alaykum 🌙', 'utf8').toString('base64');
Buffer.from(b64n, 'base64').toString('utf8'); // => 'Assalomu alaykum 🌙'
```

**Izoh:** `btoa` har belgini bitta bayt (0–255) deb oladi; emoji (🌙) bunga sig'maydi → xato. Avval matnni UTF-8 baytlarga (`TextEncoder` yoki `Buffer`) o'tkazib, so'ng Base64 qilish kerak.

---

## 6. Query qurish

```js
const q = 'node & deno = ?';

// NOTO'G'RI — encodeURI '&' va '=' ni saqlaydi (query buziladi):
encodeURI(`https://x.com/?q=${q}`);
// => 'https://x.com/?q=node%20&%20deno%20=%20?'  ('&','=' kodlanmadi)

// TO'G'RI — qiymatni encodeURIComponent bilan:
`https://x.com/?q=${encodeURIComponent(q)}`;
// => 'https://x.com/?q=node%20%26%20deno%20%3D%20%3F'
```

**Izoh:** `encodeURI` URL strukturasi belgilarini (`& = ?`) saqlaydi — bu butun URL'ni kodlash uchun mo'ljallangan. Query *qiymati* uchun aynan o'sha belgilarni kodlash kerak, shuning uchun `encodeURIComponent`.

---

## 7. Basic Auth

```js
const user = 'admin';
const pass = 'p@ss:1';
const token = Buffer.from(`${user}:${pass}`).toString('base64');
const header = `Basic ${token}`;
// => 'Basic YWRtaW46cEBzczox'

// Tekshirish (decode):
const decoded = Buffer.from(token, 'base64').toString('utf8'); // => 'admin:p@ss:1'
const idx = decoded.indexOf(':');
const u = decoded.slice(0, idx);     // => 'admin'
const p = decoded.slice(idx + 1);    // => 'p@ss:1'
```

**Izoh:** Parol ichidagi `:` muammo emas — **birinchi** `:` username/password chegarasi, qolgani parolga tegishli. Shu sababli `split(':')` o'rniga *birinchi* `:` bo'yicha ajratish kerak (`indexOf`). Eslatma: Basic Auth faqat HTTPS bilan xavfsiz.

---

## 8. Farqlar jadvali

| Mezon | Encoding | Encryption | Compression | Hashing |
|---|---|---|---|---|
| Kalit kerakmi | Yo'q | Ha | Yo'q | Yo'q (HMAC'da ha) |
| Qaytadimi | Ha (decode) | Ha (kalit bilan) | Ha (decompress) | Yo'q (one-way) |
| Output hajmi | Kattalashadi (+33%) | ~teng/katta | Kichrayadi | Sobit |
| Maqsad | Format moslash | Maxfiylik | Joy tejash | Barmoq izi/integrity |
| Misol | Base64, hex | AES, RSA | gzip, zlib | SHA-256, MD5 |

**Izoh:** Asosiy farqlar — kalit kerakmi va qaytariladimi. Base64'ni "xavfsizlik" deb ishlatish keng tarqalgan xato: u kalitsiz va oson qaytadi.

---

## 9. Data URI

```js
const b64 = Buffer.from('hi').toString('base64'); // => 'aGk='
const uri = `data:text/plain;base64,${b64}`;
// => 'data:text/plain;base64,aGk='
```

```text
data:text/plain;base64,aGk=
└┬─┘ └────┬────┘ └─┬──┘ └┬─┘
 │        │        │      └ Base64 ma'lumot ("hi")
 │        │        └ kodlash usuli
 │        └ MIME turi
 └ sxema
```

**Izoh:** Brauzer manzil satrida bu URI ochilganda `hi` matni ko'rinadi — brauzer Base64'ni decode qilib, `text/plain` sifatida ko'rsatadi. Rasm uchun `data:image/png;base64,...` bo'lardi va `<img src>` ichida ishlatish mumkin (tashqi so'rovsiz inline rasm).

---

## 10. Round-trip funksiya

```js
function encode(str) {
  return Buffer.from(str, 'utf8').toString('base64');
}
function decode(b64) {
  return Buffer.from(b64, 'base64').toString('utf8');
}

for (const s of ['Salom dunyo', 'Assalomu alaykum 🌙', '', 'ўзбек']) {
  console.log(decode(encode(s)) === s); // => true (hammasi)
}
```

**Izoh:** `Buffer` UTF-8'ni to'g'ri eplaydi, shuning uchun o'zbekcha, emoji va bo'sh string ham round-trip'da saqlanadi. Bo'sh string `''` → `''` (Base64'da ham bo'sh). `btoa`/`atob` bilan bu Unicode inputlar buzilardi — `Buffer` farqi shunda.

---

← [Encoding bo'limiga qaytish](../../encoding/README.md)
