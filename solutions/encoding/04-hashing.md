# Hashing — Yechimlar

Bu fayl [`encoding/04-hashing.md`](../../encoding/04-hashing.md) dagi "Masalalar" bo'limining yechimlarini izohlari bilan beradi. Avval mustaqil urinib ko'ring, keyin solishtiring.

---

## 1. Deterministic va fixed-size

```js
const crypto = require('crypto');
const sha = s => crypto.createHash('sha256').update(s).digest('hex');

sha('a').length;            // => 64  (256 bit = 64 hex char)
sha('Salom dunyo').length;  // => 64
sha('x'.repeat(100000)).length; // => 64

sha('a') === sha('a');      // => true  (deterministic)
```

**Izoh:** Input uzunligidan qat'i nazar SHA-256 output doim 256 bit = 64 hex char (fixed-size). Bir xil input doim bir xil hash beradi (deterministic) — shuning uchun parolni tekshirishda qayta hash qilib solishtirish ishlaydi.

---

## 2. Avalanche effect

```js
sha('password');
// => '5e884898da28047151d0e56f8dc6292773603d0d6aabbdd62a11ef721d1542d8'
sha('passwor d'); // bitta belgi farq (bo'sh joy)
// => '...butunlay boshqa qiymat...'
```

**Izoh:** Atigi bitta belgi o'zgardi, lekin output'ning taxminan **yarmi (~50%)** biti o'zgaradi — bu **avalanche effect**. Ikki hash'ni yonma-yon qo'ysangiz, hech qanday o'xshashlik yo'q; input bilan output orasidagi bog'liqlik prognoz qilib bo'lmaydigan darajada yashiringan.

---

## 3. MD5 vs SHA-256

```js
crypto.createHash('md5').update('test').digest('hex').length;    // => 32  (128 bit)
crypto.createHash('sha256').update('test').digest('hex').length; // => 64  (256 bit)
```

**Izoh:** MD5 — 128 bit (32 hex char), SHA-256 — 256 bit (64 hex char). MD5 bugun xavfsizlik uchun ishlatilmaydi, chunki uning **collision**lari amalda oson topiladi (ikki har xil fayl bir xil MD5 berishi ko'rsatilgan) — bu imzo va integrity kafolatini buzadi.

---

## 4. Salt namoyishi

```js
// Salt'siz — bir xil parol → bir xil hash:
sha('123456') === sha('123456'); // => true (rainbow table oson)

// Salt bilan:
function hashWithSalt(pass) {
  const salt = crypto.randomBytes(16).toString('hex');
  const hash = crypto.createHash('sha256').update(salt + pass).digest('hex');
  return { salt, hash };
}
const a = hashWithSalt('123456');
const b = hashWithSalt('123456');
a.hash === b.hash; // => false (salt har xil!)
```

**Izoh:** Salt'siz bir xil parol doim bir xil hash beradi — hujumchi rainbow table bilan tez topadi. Tasodifiy salt qo'shilsa, har bir hash noyob bo'ladi (`a.salt != b.salt`), shuning uchun hash'lar ham har xil. Bu rainbow table'ni befoyda qiladi. (Eslatma: real parol uchun baribir SHA-256 emas, bcrypt kerak — keyingi misol.)

---

## 5. bcrypt round-trip

```js
const bcrypt = require('bcrypt');

const hash1 = await bcrypt.hash('mySecret1', 12);
const hash2 = await bcrypt.hash('mySecret1', 12);
hash1 === hash2; // => false  (har safar boshqa salt)

await bcrypt.compare('mySecret1', hash1); // => true  (to'g'ri parol)
await bcrypt.compare('wrongPass', hash1); // => false (noto'g'ri)
```

**Izoh:** Bir xil parol ikki marta hash qilinsa ham hash'lar **har xil**, chunki bcrypt har safar yangi tasodifiy salt yaratadi va uni hash ichiga (`$2b$12$...`) joylaydi. `bcrypt.compare` saqlangan hash'dan salt'ni o'qib, kiritilgan parolni o'sha salt bilan qayta hash qilib solishtiradi — shuning uchun `===` ishlatish shart emas (va noto'g'ri bo'lardi).

---

## 6. HMAC

```js
const hmac = (key, msg) =>
  crypto.createHmac('sha256', key).update(msg).digest('hex');

const sig1 = hmac('secret', 'webhook-payload');
const sig2 = hmac('secret2', 'webhook-payload'); // kalit o'zgardi
const sig3 = hmac('secret', 'webhook-payload!'); // xabar o'zgardi

sig1 === sig2; // => false (kalit ta'sir qiladi)
sig1 === sig3; // => false (xabar ta'sir qiladi)
```

**Izoh:** HMAC ham kalitga, ham xabarga bog'liq — ikkalasidan biri o'zgarsa imzo butunlay o'zgaradi (avalanche). Kalitni bilmagan odam to'g'ri imzo yasay olmaydi; shuning uchun HMAC ham integrity (xabar buzilmadimi), ham authenticity (haqiqiy yuboruvchidanmi) ni tasdiqlaydi.

---

## 7. Xavfsiz solishtirish

```js
function compareEq(a, b) {
  return a === b; // tez chiqib ketadi — timing leak
}

function compareSafe(aHex, bHex) {
  const a = Buffer.from(aHex, 'hex');
  const b = Buffer.from(bHex, 'hex');
  if (a.length !== b.length) return false;
  return crypto.timingSafeEqual(a, b);
}
```

**Izoh:** `===` ikki string'ni belgi-belgi solishtiradi va birinchi farqda **darhol** to'xtaydi — to'g'ri prefiks uzunroq tekshiriladi, ya'ni javob vaqti hujumchiga imzoni belgi-belgi taxmin qilishga ma'lumot beradi (**timing attack**). `crypto.timingSafeEqual` har doim **doimiy vaqtda** ishlaydi (uzunliklar teng bo'lsa), shu sirni chiqarmaydi. Webhook/token tekshiruvida shu kerak.

---

## 8. Checksum vs hash

```js
const crc = require('zlib').crc32
  ? require('zlib').crc32('hello')        // Node 22+
  : 0;                                     // yoki tashqi 'crc' paketi
const sha = crypto.createHash('sha256').update('hello').digest('hex');
```

**Izoh:** CRC32 — 32 bit, juda tez, *tasodifiy* buzilishni (uzatish xatosi, disk nuqsoni) aniqlash uchun mo'ljallangan. U **xavfsizlik uchun yaramaydi**: hujumchi xohlagan CRC32 qiymatiga moslab ma'lumotni atayin o'zgartira oladi — CRC'da collision yasash arzon. SHA-256 esa collision-resistant, atayin soxtalashtirish amalda imkonsiz, lekin sekinroq.

---

## 9. Parol tekshiruv oqimi

```js
const bcrypt = require('bcrypt');
const db = new Map(); // soddalashtirilgan "baza"

async function register(user, pass) {
  const hash = await bcrypt.hash(pass, 12);
  db.set(user, hash);        // FAQAT hash saqlanadi, asl parol emas
}

async function login(user, pass) {
  const hash = db.get(user);
  if (!hash) return false;
  return bcrypt.compare(pass, hash);
}
```

**Izoh:** Asl parol hech qayerda saqlanmaydi — faqat bcrypt hash. Sabab: baza o'g'irlansa ham, hujumchi hash'dan parolni qaytara olmaydi (one-way), bcrypt esa sekin va salt bilan bo'lgani uchun brute-force/rainbow table befoyda. Tekshirish faqat `bcrypt.compare` orqali: kiritilgan parolni qayta hash qilib solishtirish.

---

## 10. Birthday hisobi

**Tushuntirish:** Birthday paradox — 23 kishilik xonada ikki kishining tug'ilgan kuni mos kelish ehtimoli ~50%. Sabab: biz *aniq bir kun*ni emas, *istalgan juftlik*ni qidiramiz; juftliklar soni `C(23,2)=253` ta — shuning uchun 365 emas, 23 kishida yarim ehtimol.

Hash'ga: n-bitlik hash'da 2^n ta mumkin bo'lgan qiymat bor. Collision (istalgan ikki input bir xil hash) topish uchun barcha 2^n ni emas, taxminan **2^(n/2)** ta input ko'rish kifoya — chunki har yangi input mavjudlari bilan juftlik hosil qiladi.

```text
256-bitli hash:
  aniq bir hash'ni topish (preimage):  ~2^256 urinish
  istalgan collision (birthday):        ~2^128 urinish  ← ancha kam
```

**Izoh:** Shu sababli "256 bit → 2^256" deyish xato; amaliy collision chidamliligi ~2^128 (lekin bu ham hozircha imkonsiz). Hash uzunligi kerakli xavfsizlik darajasidan ikki barobar tanlanishi shundan.

---

← [Encoding bo'limiga qaytish](../../encoding/README.md)
