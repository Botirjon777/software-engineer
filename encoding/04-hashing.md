# Hashing

**Hash** funksiya — ixtiyoriy uzunlikdagi input'ni sobit uzunlikdagi "barmoq izi"ga aylantiruvchi, **bir tomonlama** (one-way) funksiya. Bu bo'limda hash xossalari (deterministic, fast, fixed-size, avalanche, one-way, collision-resistant), hashing'ni encoding va encryption'dan ajratish, cryptographic vs non-cryptographic hash, mashhur algoritmlar (MD5, SHA-1, SHA-256/SHA-3), collision va birthday paradox, real use-case'lar va eng muhimi — **parol hashing** (nega SHA-256 yetarli emas, **salt**, **bcrypt/scrypt/argon2**), **HMAC** va **checksum (CRC32)** ko'rib chiqiladi. Node.js'da `crypto.createHash`, `createHmac` va `bcrypt` misollari bilan.

## Mundarija

- [Hash funksiya nima](#hash-funksiya-nima)
- [Hash xossalari](#hash-xossalari)
- [Hashing vs Encoding vs Encryption](#hashing-vs-encoding-vs-encryption)
- [Cryptographic vs Non-cryptographic hash](#cryptographic-vs-non-cryptographic-hash)
- [Mashhur algoritmlar (MD5, SHA-1, SHA-256, SHA-3)](#mashhur-algoritmlar-md5-sha-1-sha-256-sha-3)
- [Collision va birthday paradox](#collision-va-birthday-paradox)
- [Hash use-case'lar](#hash-use-caselar)
- [Password hashing: nega SHA-256 YOMON](#password-hashing-nega-sha-256-yomon)
- [Salt nima va nega kerak](#salt-nima-va-nega-kerak)
- [bcrypt, scrypt, argon2](#bcrypt-scrypt-argon2)
- [HMAC — hash + secret key](#hmac--hash--secret-key)
- [Checksum (CRC32) vs cryptographic hash](#checksum-crc32-vs-cryptographic-hash)
- [Node: createHash, createHmac, bcrypt](#node-createhash-createhmac-bcrypt)
- [Intervyu savollari (Q&A)](#intervyu-savollari-qa)
- [Masalalar](#masalalar)

---

## Hash funksiya nima

**💡 Tushuncha:** **Hash funksiya** — ixtiyoriy uzunlikdagi input'ni (matn, fayl, baytlar) qabul qilib, **sobit uzunlikdagi** chiqish (hash, digest) qaytaruvchi funksiya. Bu chiqish — ma'lumotning "barmoq izi" (fingerprint).

```text
  Input (ixtiyoriy uzunlik)        Hash (sobit uzunlik)
  "a"               ─────────▶     ca978112ca1bbdca... (SHA-256, 256 bit)
  "Salom dunyo"     ─────────▶     a1d0c6e83f02732... (SHA-256, 256 bit)
  500 MB lik fayl   ─────────▶     5f4dcc3b5aa765d... (SHA-256, 256 bit)
```

E'tibor bering: input 1 belgi yoki 500 MB bo'lishidan qat'i nazar, SHA-256 chiqishi **doim 256 bit** (64 hex char).

```js
const crypto = require('crypto');
crypto.createHash('sha256').update('a').digest('hex');
// => 'ca978112ca1bbdcafac231b39a23dc4da786eff8147c4e72b9807785afee48bb'
```

---

## Hash xossalari

**💡 Tushuncha:** Yaxshi (cryptographic) hash funksiya quyidagi xossalarga ega:

| Xossa | Ma'nosi |
|---|---|
| **Deterministic** | Bir xil input → har doim bir xil hash |
| **Fast** | Tez hisoblanadi (lekin parol uchun bu kamchilik!) |
| **Fixed-size** | Output uzunligi doim bir xil (input qancha bo'lsa ham) |
| **Avalanche effect** | Input'da 1 bit o'zgarsa, output'ning ~yarmi o'zgaradi |
| **One-way / irreversible** | Hash'dan input'ni qaytarib bo'lmaydi |
| **Collision-resistant** | Bir xil hash beruvchi ikki input topish amalda imkonsiz |

**Avalanche effect** misoli:

```js
crypto.createHash('sha256').update('hello').digest('hex');
// => '2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824'
crypto.createHash('sha256').update('hellp').digest('hex'); // bitta harf o'zgardi
// => '7d6e9b... butunlay boshqa'  (deyarli barchasi o'zgardi)
```

**⚠️ Ehtiyot bo'l:** "Fast" — cryptographic hash uchun yaxshi, lekin **parol hashing uchun aynan yomon** xususiyat (quyida ko'ramiz).

---

## Hashing vs Encoding vs Encryption

**💡 Tushuncha:** Uchovini ajratish — intervyuda doimiy savol. Asosiy farq — **qaytariladimi** va **kalit kerakmi**.

| Xossa | Hashing | Encoding | Encryption |
|---|---|---|---|
| Yo'nalish | **one-way** (qaytmaydi) | two-way | two-way |
| Kalit (key) | yo'q (HMAC bundan mustasno) | yo'q | bor |
| Maqsad | barmoq izi / integrity / parol | format moslash | maxfiylik |
| Output hajmi | **sobit** | kattalashadi | ~teng/katta |
| Misol | SHA-256, bcrypt | Base64, hex | AES, RSA |

```text
Encoding:    "Hi" ⇄ "SGk="      (oson qaytadi)
Encryption:  "Hi" ⇄ "x9$@"      (kalit bilan qaytadi)
Hashing:     "Hi" → "3639.." ⇏  (QAYTMAYDI)
```

**⚠️ Ehtiyot bo'l:** "Parolni hash qilib saqladim, keyin uni decrypt qilaman" — **mantiqiy xato**. Hash qaytmaydi. Parolni tekshirish — kiritilgan parolni *qaytadan hash qilib*, saqlangan hash bilan solishtirish orqali bo'ladi.

---

## Cryptographic vs Non-cryptographic hash

**💡 Tushuncha:** Hammasi "hash" deyilsa-da, ikki sinf butunlay boshqa maqsad uchun:

| | Cryptographic | Non-cryptographic |
|---|---|---|
| Maqsad | xavfsizlik, integrity, parol | tezlik, hash table, checksum |
| Collision-resistant | HA (atayin) | YO'Q (faqat tasodifiy taqsimot) |
| Tezlik | sekinroq | juda tez |
| Misol | SHA-256, SHA-3, BLAKE2 | CRC32, MurmurHash, FNV |

**⚠️ Ehtiyot bo'l:** **Hash table**dagi "hash" (masalan JS `Map` ichida yoki MurmurHash) — bu *non-cryptographic*. Uni hech qachon parol yoki xavfsizlik uchun ishlatmang. Aksincha, SHA-256'ni hash table kaliti uchun ishlatish — keraksiz sekinlik.

---

## Mashhur algoritmlar (MD5, SHA-1, SHA-256, SHA-3)

**💡 Tushuncha:** Tarixiy va hozirgi cryptographic hash'lar:

| Algoritm | Output | Holati | Tavsiya |
|---|---|---|---|
| **MD5** | 128 bit | **BUZILGAN** (collision oson topiladi) | Xavfsizlik uchun ISHLATMANG |
| **SHA-1** | 160 bit | **BUZILGAN** (2017, SHAttered hujumi) | ISHLATMANG |
| **SHA-256** (SHA-2) | 256 bit | Ishonchli | HA, integrity uchun |
| **SHA-3** | 256/512 bit | Ishonchli (boshqa dizayn) | HA |

```js
crypto.createHash('md5').update('test').digest('hex');    // tez, lekin BUZILGAN
crypto.createHash('sha256').update('test').digest('hex'); // ishonchli
```

**⚠️ Ehtiyot bo'l:** MD5/SHA-1 ni bugun **faqat** xavfsizlikka aloqasiz checksum (masalan fayl tasodifiy buzilganini tekshirish) uchun ishlatish mumkin. Imzo, sertifikat, parol uchun — qat'iyan yo'q.

---

## Collision va birthday paradox

**💡 Tushuncha:** **Collision** — ikki *har xil* input bir xil hash chiqarishi: `hash(A) == hash(B)`, `A != B`. Hash output cheklangan (masalan 256 bit), input cheksiz bo'lgani uchun collision'lar **mavjud bo'lishi shart** — masala ularni *topish qanchalik qiyin*ligida.

**Birthday paradox:** 23 kishilik xonada ikki kishining tug'ilgan kuni bir xil bo'lish ehtimoli ~50% (365 emas, atigi 23 ta!). Sabab — biz *istalgan juftlik*ni qidiramiz, aniq bir kunni emas.

Hash'ga ta'siri: **n-bitlik** hash'da collision topish uchun taxminan **2^(n/2)** urinish kerak, 2^n emas. Ya'ni SHA-256 (256 bit) uchun collision ~2^128 — bu amalda imkonsiz, lekin "yarmi" ekanini bilish muhim.

**⚠️ Ehtiyot bo'l:** "256 bit hash → collision uchun 2^256 kerak" — **xato**. Birthday paradox sababli ~2^128. Shuning uchun hash uzunligi xavfsizlik darajasidan ikki barobar katta tanlanadi.

---

## Hash use-case'lar

**💡 Tushuncha:** Hash qayerda ishlatiladi:

- **Data integrity / checksum** — yuklab olingan fayl buzilmaganini tekshirish: e'lon qilingan hash bilan solishtirasiz.
- **Deduplication** — bir xil fayllarni hash orqali aniqlash (bir xil hash → ehtimol bir xil fayl).
- **Hash table** — kalitni "vedro" indeksiga moslash (bu *non-cryptographic*, boshqa maqsad).
- **Digital signature** — hujjatni avval hash qilib, so'ng hash'ni imzolash (butun hujjatni emas).
- **Password storage** — parolni hash qilib saqlash (lekin maxsus, sekin hash bilan — quyida).
- **Git** — har commit/fayl SHA-1/SHA-256 hash bilan identifikatsiya qilinadi.

```js
// Fayl integrity tekshiruvi (soddalashtirilgan)
const fileBytes = require('fs').readFileSync('archive.zip');
crypto.createHash('sha256').update(fileBytes).digest('hex');
// => e'lon qilingan hash bilan solishtiriladi
```

---

## Password hashing: nega SHA-256 YOMON

**💡 Tushuncha:** Parolni `SHA-256` bilan hash qilib saqlash **xavfsiz emas**. Sabab — SHA-256 *atayin tez*. Hujumchi bazani o'g'irlasa:

- **Brute-force** — zamonaviy GPU sekundiga **milliardlab** SHA-256 hisoblaydi → ko'p parolni tez topadi.
- **Rainbow table** — oldindan hisoblangan `parol → hash` jadvali; baza hash'ini qidirib parolni topadi.

```text
Parol bazasi o'g'irlandi:
  user1: 5e88489...  (SHA-256("123456"))
  user2: 5e88489...  (yana SHA-256("123456")!)   ← bir xil hash!
```

**⚠️ Ehtiyot bo'l:** Tez hash (SHA-256, MD5) parol uchun yomon. Kerak — *sekin, atayin qimmat* (slow, adaptive) hash: bcrypt/scrypt/argon2.

---

## Salt nima va nega kerak

**💡 Tushuncha:** **Salt** — har parol uchun yaratiladigan tasodifiy qiymat; u parolga qo'shilib hash qilinadi va hash bilan birga saqlanadi. Maqsadi:

- Bir xil parol → **har xil hash** (chunki salt har xil). Yuqoridagi `user1`/`user2` muammosi yo'qoladi.
- **Rainbow table'ni befoyda qiladi** — har parol uchun alohida jadval kerak bo'lib qoladi.

```text
hash = H(salt + parol)
user1: salt=ab12, hash=H("ab12" + "123456") = 9f3c...
user2: salt=cd34, hash=H("cd34" + "123456") = 71e0...   ← endi har xil!
```

**⚠️ Ehtiyot bo'l:** Salt **maxfiy emas** — hash bilan birga saqlanadi va shunday bo'lishi kerak. Uning vazifasi maxfiylik emas, balki har hash'ni *noyob* qilish. (Maxfiy qo'shimcha kerak bo'lsa — bu "pepper", alohida tushuncha.)

---

## bcrypt, scrypt, argon2

**💡 Tushuncha:** Parol uchun maxsus, **sekin va adaptiv** hash funksiyalar. Ular ichida salt va **work factor** (cost / iterations) bor — apparat tezlashgan sari raqamni oshirib, hujumni qimmat ushlab turish mumkin.

| Funksiya | Asosiy resurs | Eslatma |
|---|---|---|
| **bcrypt** | CPU vaqti | Eng keng tarqalgan, `cost` (rounds) parametri |
| **scrypt** | CPU + **xotira** | Xotiraga och — GPU/ASIC hujumiga chidamliroq |
| **argon2** | CPU + xotira + parallellik | Zamonaviy tavsiya (argon2id) |

```js
const bcrypt = require('bcrypt');
const hash = await bcrypt.hash('parol123', 12); // cost=12, salt avtomatik ichida
await bcrypt.compare('parol123', hash);         // => true (tekshirish)
```

**💡 Tushuncha:** bcrypt hash o'z ichiga algoritm, cost va salt'ni *o'zi* saqlaydi (`$2b$12$...`), shuning uchun salt'ni alohida saqlash shart emas.

**⚠️ Ehtiyot bo'l:** **Work factor** (cost) ni yetarlicha baland qo'ying (masalan bcrypt cost ≥ 12). Juda past bo'lsa — sekinlikning foydasi yo'qoladi.

---

## HMAC — hash + secret key

**💡 Tushuncha:** **HMAC** (Hash-based Message Authentication Code) — hash + **maxfiy kalit**. U xabarning ham buzilmaganini (integrity), ham haqiqiy yuboruvchidan kelganini (authenticity) tasdiqlaydi. Faqat kalitni biluvchi to'g'ri HMAC yarata/tekshira oladi.

```js
const sig = crypto.createHmac('sha256', 'secret-key')
  .update('message')
  .digest('hex');
// => kalitsiz bu qiymatni qayta yasab bo'lmaydi
```

**Nega oddiy `hash(key + msg)` emas?** Bunday "naive" konstruksiya **length-extension hujumi**ga zaif (SHA-256 kabi Merkle–Damgård hash'larda). Hujumchi kalitni bilmasdan ham `key + msg + qo'shimcha` uchun yangi valid hash yasashi mumkin. HMAC ikki bosqichli ichki/tashqi padding (`ipad`/`opad`) bilan buni bartaraf qiladi.

**⚠️ Ehtiyot bo'l:** Webhook imzosini tekshirganda HMAC'larni **doimiy vaqtli** (`crypto.timingSafeEqual`) solishtiring — oddiy `===` timing attack'ga zaif.

---

## Checksum (CRC32) vs cryptographic hash

**💡 Tushuncha:** **Checksum** (masalan **CRC32**) — *tasodifiy* buzilishni (uzatishdagi xato, disk nosozligi) aniqlash uchun tez, non-cryptographic kod. U **atayin hujumdan himoya qilmaydi**.

| | CRC32 (checksum) | SHA-256 (crypto hash) |
|---|---|---|
| Maqsad | tasodifiy xatoni aniqlash | xavfsizlik / integrity |
| Tezlik | juda tez | sekinroq |
| Atayin soxtalashtirish | oson | amalda imkonsiz |
| Output | 32 bit | 256 bit |

**⚠️ Ehtiyot bo'l:** CRC32'ni xavfsizlik uchun ishlatmang — hujumchi xohlagan CRC32'ga moslab ma'lumotni o'zgartira oladi. U faqat "fayl yo'lda buzilib qolmadimi" uchun.

---

## Node: createHash, createHmac, bcrypt

**💡 Tushuncha:** Node.js'da hashing'ning standart vositasi — o'rnatilgan `crypto` moduli; parol uchun esa `bcrypt` (yoki `argon2`) paketi.

```js
const crypto = require('crypto');

// 1) Oddiy hash (integrity / checksum uchun)
crypto.createHash('sha256').update('Salom').digest('hex');
// => '...64 ta hex belgi...'

// 2) HMAC (message authentication uchun)
crypto.createHmac('sha256', 'secret').update('Salom').digest('hex');

// 3) Webhook imzosini xavfsiz solishtirish
const a = Buffer.from(receivedSig, 'hex');
const b = Buffer.from(expectedSig, 'hex');
const ok = a.length === b.length && crypto.timingSafeEqual(a, b);
```

```js
// 4) Parol hashing (bcrypt)
const bcrypt = require('bcrypt');

async function register(password) {
  return bcrypt.hash(password, 12);          // saqlanadigan hash
}
async function login(password, storedHash) {
  return bcrypt.compare(password, storedHash); // true/false
}
```

| Vazifa | To'g'ri vosita |
|---|---|
| Fayl integrity / checksum | `crypto.createHash('sha256')` |
| Message authentication | `crypto.createHmac('sha256', key)` |
| Parol saqlash | `bcrypt` / `argon2` (SHA-256 emas!) |
| Hash solishtirish (xavfsiz) | `crypto.timingSafeEqual` |

---

## Intervyu savollari (Q&A)

### ❓ Hash funksiya nima?

**✅ Javob:** Ixtiyoriy uzunlikdagi input'ni sobit uzunlikdagi chiqishga (digest/barmoq izi) aylantiruvchi bir tomonlama funksiya. Input 1 bayt yoki 1 GB bo'lsin — SHA-256 chiqishi doim 256 bit.

### ❓ Hash funksiyaning asosiy xossalarini ayting.

**✅ Javob:** Deterministic, fast, fixed-size, avalanche effect, one-way (irreversible), collision-resistant. Cryptographic hash uchun bularning hammasi muhim.

### ❓ Avalanche effect nima?

**✅ Javob:** Input'da arzimas (1 bit) o'zgarish output'ning taxminan yarmini o'zgartirishi. Bu input bilan output orasidagi bog'liqlikni yashiradi va prognoz qilishni imkonsiz qiladi.

### ❓ Hashing va encryption farqi nima?

**✅ Javob:** Hashing — one-way, kalitsiz, qaytmaydi (barmoq izi/parol uchun). Encryption — two-way, kalit bilan, maxfiylik uchun qaytariladi. Parolni "hash qilib keyin decrypt qilaman" — mantiqiy xato.

### ❓ Cryptographic va non-cryptographic hash farqi?

**✅ Javob:** Cryptographic (SHA-256) — collision-resistant, xavfsizlik uchun, sekinroq. Non-cryptographic (CRC32, MurmurHash) — tez, hash table/checksum uchun, collision-resistant emas. Ularni aralashtirib ishlatmang.

### ❓ Qaysi hash algoritmlari buzilgan?

**✅ Javob:** MD5 va SHA-1 — collision hujumlari amalda mumkin, xavfsizlik uchun ishlatmaslik kerak. SHA-256 (SHA-2) va SHA-3 hozircha ishonchli.

### ❓ Collision nima?

**✅ Javob:** Ikki har xil input bir xil hash chiqarishi (`hash(A)==hash(B)`, `A!=B`). Output cheklangani uchun collision mavjud bo'lishi shart; yaxshi hash'da uni *topish* amalda imkonsiz.

### ❓ Birthday paradox hash'ga qanday aloqador?

**✅ Javob:** n-bitlik hash'da collision topish ~2^(n/2) urinish talab qiladi (2^n emas), chunki istalgan juftlik qidiriladi. Shu sababli SHA-256 collision'i ~2^128.

### ❓ Nega parolni SHA-256 bilan saqlash yomon?

**✅ Javob:** SHA-256 juda tez — GPU sekundiga milliardlab hisoblaydi, brute-force va rainbow table osonlashadi. Parol uchun atayin sekin (bcrypt/scrypt/argon2) hash kerak.

### ❓ Salt nima va nega kerak?

**✅ Javob:** Har parol uchun tasodifiy qiymat; parolga qo'shilib hash qilinadi va hash bilan saqlanadi. Bir xil parollarni har xil hash qiladi va rainbow table'ni befoyda qiladi. Salt maxfiy emas.

### ❓ Salt'ni qayerda saqlash kerak — maxfiymi?

**✅ Javob:** Maxfiy emas; hash bilan birga saqlanadi (bcrypt uni hash ichiga qo'yadi). Vazifasi maxfiylik emas, har hash'ni noyob qilish. Maxfiy qo'shimcha kerak bo'lsa — bu "pepper", alohida tushuncha.

### ❓ bcrypt/scrypt/argon2 nega SHA-256'dan yaxshi (parol uchun)?

**✅ Javob:** Ular atayin sekin va adaptiv — work factor (cost) bilan qiyinlikni oshirish mumkin. scrypt/argon2 yana xotiraga och, GPU/ASIC hujumiga chidamliroq. argon2id — zamonaviy tavsiya.

### ❓ Work factor (cost) nima?

**✅ Javob:** Hash hisoblash qanchalik qimmat ekanini belgilovchi parametr (bcrypt rounds, argon2 iterations/memory). Apparat tezlashgani sari uni oshirib, hujumchi xarajatini baland ushlab turiladi.

### ❓ HMAC nima va nega oddiy hash(key+msg) emas?

**✅ Javob:** HMAC — hash + maxfiy kalit, message authentication uchun. Naive `hash(key+msg)` length-extension hujumiga zaif (Merkle–Damgård hash'larda) — hujumchi kalitsiz yangi valid hash yasashi mumkin. HMAC ikki bosqichli padding bilan buni bartaraf qiladi.

### ❓ Checksum (CRC32) va cryptographic hash farqi?

**✅ Javob:** CRC32 — tasodifiy buzilishni aniqlash uchun tez, non-cryptographic; atayin soxtalashtirishdan himoya qilmaydi. SHA-256 — xavfsizlik/integrity uchun, soxtalashtirish amalda imkonsiz. CRC32'ni xavfsizlik uchun ishlatmang.

### ❓ Hash'larni solishtirishda nimaga ehtiyot bo'lish kerak?

**✅ Javob:** Maxfiy hash/HMAC'larni `crypto.timingSafeEqual` bilan doimiy vaqtda solishtiring; oddiy `===` belgi-belgi tugaydi va timing attack'ga ma'lumot beradi.

---

## Masalalar

> Yechimlar: [solutions/encoding/04-hashing.md](../solutions/encoding/04-hashing.md)

1. **Deterministic va fixed-size.** `"a"`, `"Salom dunyo"` va katta matnni SHA-256 bilan hash qiling. Uchala natijaning uzunligini (hex char soni) solishtiring va bir xil input doim bir xil hash berishini ko'rsating.

2. **Avalanche effect.** `"password"` va `"passwor d"` (yoki bitta harfi farq qiladigan) ni SHA-256 bilan hash qiling. Ikki hash'ni yonma-yon qo'yib, qancha belgisi farq qilishini taxminan baholang.

3. **MD5 vs SHA-256.** Bir xil matnni MD5 va SHA-256 bilan hash qiling, output uzunligini solishtiring. MD5 nega bugun xavfsizlik uchun ishlatilmasligini 1-2 jumlada izohlang.

4. **Salt namoyishi.** Salt'siz: `"123456"` ni ikki marta SHA-256 qiling — natija bir xilmi? So'ng har safar tasodifiy salt qo'shib (`H(salt+parol)`) ikki marta hash qiling — endi natija har xilmi? Nima uchun?

5. **bcrypt round-trip.** `bcrypt` bilan `"mySecret1"` parolini hash qiling, so'ng `bcrypt.compare` bilan to'g'ri va noto'g'ri parolni tekshiring. Bir xil parolni ikki marta hash qilsangiz, hash'lar bir xil chiqadimi? Nega?

6. **HMAC.** `crypto.createHmac` bilan `"secret"` kalit va `"webhook-payload"` xabari uchun imzo yarating. Kalitni o'zgartirsangiz imzo o'zgaradimi? Xabarni o'zgartirsangizchi?

7. **Xavfsiz solishtirish.** Ikki hex imzoni `===` bilan va `crypto.timingSafeEqual` bilan solishtiruvchi funksiya yozing. Nega webhook tekshiruvida `timingSafeEqual` afzal ekanini tushuntiring.

8. **Checksum vs hash.** Bir matn uchun (kutubxona yoki o'z implementatsiyangiz bilan) CRC32 va SHA-256 hisoblang. Qaysi biri tezroq va nega CRC32 xavfsizlik uchun yaramasligini ayting.

9. **Parol tekshiruv oqimi.** Soddalashtirilgan `register(user, pass)` va `login(user, pass)` funksiyalarini yozing: register bcrypt hash'ni "bazaga" saqlasin, login esa `bcrypt.compare` qilsin. Nega hech qayerda asl parol saqlanmasligi kerakligini izohlang.

10. **Birthday hisobi.** Nega 256-bitli hash uchun collision topishga ~2^128 (2^256 emas) urinish kerakligini birthday paradox orqali o'z so'zlaringiz bilan tushuntiring. 23 kishi misolini ham keltiring.

← [Encoding bo'limiga qaytish](./README.md)
