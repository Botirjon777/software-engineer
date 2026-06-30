# Streams va Buffers

Stream — bu Node.js'da ma'lumotni **bo'lak-bo'lak (chunk)** qayta ishlash abstraksiyasi. Bu mavzu senior darajadagi suhbatlarda eng kuchli farqlovchi mavzulardan biri: kim stream'larni, backpressure'ni va `pipeline()`'ni tushunadi — u kim katta hajmli ma'lumot bilan ishlay olishini bildiradi. Bu hujjat nega stream kerakligidan boshlab, custom Transform yozish va real misollargacha (gzip, CSV, HTTP streaming) chuqur boradi.

## Mundarija

- [Nega stream kerak (xotira va vaqt samaradorligi)](#nega-stream-kerak-xotira-va-vaqt-samaradorligi)
- [Buffer — stream'ning quruvchi g'ishti](#buffer--streamning-quruvchi-gishti)
- [4 ta stream turi](#4-ta-stream-turi)
- [Flowing vs Paused rejim](#flowing-vs-paused-rejim)
- ['data' / 'end' / 'readable' bilan o'qish](#data--end--readable-bilan-oqish)
- [pipe() va pipeline()](#pipe-va-pipeline)
- [Backpressure — nima, nega muhim](#backpressure--nima-nega-muhim)
- [highWaterMark va buffering](#highwatermark-va-buffering)
- [Custom Readable yaratish](#custom-readable-yaratish)
- [Custom Writable yaratish](#custom-writable-yaratish)
- [Custom Transform yaratish](#custom-transform-yaratish)
- [Object mode](#object-mode)
- [Stream'da error handling](#streamda-error-handling)
- [Async iterator (for await...of)](#async-iterator-for-awaitof)
- [Real misollar](#real-misollar)
- [Intervyu Q&A](#intervyu-qa)
- [Masalalar](#masalalar)

---

## Nega stream kerak (xotira va vaqt samaradorligi)

**💡 Tushuncha:** Tasavvur qiling, 2 GB lik log faylni o'qib, mijozga jo'natmoqchisiz. Agar `fs.readFile()` ishlatsangiz, Node butun 2 GB ni **RAM ga** yuklaydi, keyin yuboradi. 50 ta mijoz bir vaqtda so'rasa — 100 GB RAM kerak bo'ladi va server qulaydi. Stream esa faylni kichik bo'laklarga (default 64 KB) bo'lib, har bo'lakni o'qib-yuborib, keyingisiga o'tadi. Xotira iste'moli **doimiy** bo'lib qoladi (fayl hajmidan qat'i nazar).

```js
const fs = require('node:fs');
const http = require('node:http');

// ❌ Yomon: butun fayl RAM ga yuklanadi
http.createServer((req, res) => {
  fs.readFile('big.log', (err, data) => {
    res.end(data); // 2 GB RAM band bo'ladi
  });
});

// ✅ Yaxshi: bo'lak-bo'lak oqadi, RAM doimiy
http.createServer((req, res) => {
  fs.createReadStream('big.log').pipe(res);
});
```

Ikki asosiy yutuq:

- **Xotira samaradorligi:** butun ma'lumotni bir vaqtda RAM da ushlab turish shart emas.
- **Vaqt samaradorligi:** birinchi bo'lak kelishi bilanoq qayta ishlashni boshlaysiz — butun ma'lumot yuklanishini kutmaysiz (past **latency**, "time to first byte").

**⚠️ Ehtiyot bo'l:** Kichik fayllar (bir necha KB) uchun stream ortiqcha murakkablik beradi — oddiy `readFile` yetarli. Stream'ning afzalligi **katta yoki noma'lum hajmli** ma'lumotda (fayllar, tarmoq, real-time data) namoyon bo'ladi.

---

## Buffer — stream'ning quruvchi g'ishti

**💡 Tushuncha:** `Buffer` — bu V8 heap'idan tashqarida joylashgan, o'zgarmas hajmdagi **raw binary** ma'lumot bloki. Stream'lar (object mode'dan tashqari) ma'lumotni Buffer bo'laklari sifatida uzatadi. Buffer string emas: u baytlar ketma-ketligi, va string ga aylantirish uchun **encoding** (`utf8`, `ascii`, `hex`, `base64`) kerak.

```js
const buf = Buffer.from('Salom', 'utf8');
console.log(buf);             // <Buffer 53 61 6c 6f 6d>
console.log(buf.length);      // 5 (bayt, belgi emas!)
console.log(buf.toString('utf8')); // 'Salom'

// Diqqat: ko'p baytli belgi bo'laklarga bo'linib qolishi mumkin
const part = Buffer.from('🚀'); // 4 bayt
console.log(part.length);     // 4
```

**⚠️ Ehtiyot bo'l:** Stream bo'laklari ko'p baytli UTF-8 belgini **o'rtasidan** kesib qo'yishi mumkin (masalan emoji yarmi bir bo'lakda, yarmi keyingisida). `chunk.toString()` ni har bo'lakka alohida qo'llasangiz, buzilgan belgilar paydo bo'ladi. To'g'ri yechim — `string_decoder` modulining `StringDecoder` klassi yoki stream'ga `setEncoding('utf8')` o'rnatish, ular chegaradagi belgilarni to'g'ri yig'adi.

---

## 4 ta stream turi

**💡 Tushuncha:** Node'da to'rtta asosiy stream turi bor. Hammasi `EventEmitter` dan meros oladi.

| Tur | Yo'nalish | Misol |
|---|---|---|
| **Readable** | ma'lumot **o'qiladi** (manba) | `fs.createReadStream`, HTTP `req`, `process.stdin` |
| **Writable** | ma'lumot **yoziladi** (manzil) | `fs.createWriteStream`, HTTP `res`, `process.stdout` |
| **Duplex** | ham o'qish, ham yozish (ikki mustaqil kanal) | TCP `socket` |
| **Transform** | Duplex, lekin chiqish kirishdan **hosil qilinadi** | `zlib.createGzip`, `crypto.createCipheriv` |

```js
const { Readable, Writable, Duplex, Transform } = require('node:stream');
```

- **Duplex**: o'qish va yozish kanallari **bog'liq emas** (TCP socket — siz yozasiz, qarama-qarshi tomon javob yuboradi).
- **Transform**: Duplex ning maxsus turi — siz yozgan narsa **o'zgartirilib** o'qish tomonidan chiqadi (gzip: yozasiz raw, o'qiysiz siqilgan).

**⚠️ Ehtiyot bo'l:** Transform — bu Duplex'ning bola klassi. Suhbatda "Transform Duplex'mi?" deb so'rashsa — ha, lekin uning kirish va chiqishi bog'langan (input → transform → output), oddiy Duplex'da esa ular mustaqil.

---

## Flowing vs Paused rejim

**💡 Tushuncha:** Readable stream ikki rejimda ishlaydi:

- **Paused (default):** ma'lumot stream ichida turadi; siz uni `.read()` chaqirib **tortib olasiz** (pull).
- **Flowing:** ma'lumot avtomatik **oqib keladi** va `'data'` event orqali sizga **itariladi** (push), siz uni qabul qilishga ulgurishingiz kerak.

Stream paused'da boshlanadi va quyidagilar uni **flowing** ga o'tkazadi:

- `.on('data', ...)` listener qo'shish,
- `.pipe(dest)` chaqirish,
- `.resume()` chaqirish.

```js
const rs = fs.createReadStream('file.txt');

// Paused → Flowing: 'data' listener qo'shdik
rs.on('data', (chunk) => {
  console.log('bo\'lak:', chunk.length);
});

// Flowing → Paused
rs.pause();
// Paused → Flowing
rs.resume();
```

**⚠️ Ehtiyot bo'l:** Flowing rejimda agar ma'lumotni iste'mol qilmasangiz (sekin yozsangiz yoki umuman handle qilmasangiz), ma'lumot **yo'qoladi** yoki RAM da yig'ilib ketadi. Aynan shu sabab `pipe()`/`pipeline()` afzal — ular flowing rejimni backpressure bilan avtomatik boshqaradi.

---

## 'data' / 'end' / 'readable' bilan o'qish

**💡 Tushuncha:** Readable'dan qo'lda o'qishning ikki uslubi bor:

**1) `'data'` + `'end'` (flowing, push):**

```js
const rs = fs.createReadStream('file.txt', { encoding: 'utf8' });
let content = '';

rs.on('data', (chunk) => { content += chunk; });   // har bo'lak
rs.on('end', () => { console.log('Tugadi:', content.length); }); // boshqa bo'lak yo'q
rs.on('error', (err) => { console.error(err); });
```

**2) `'readable'` + `.read()` (paused, pull):**

```js
const rs = fs.createReadStream('file.txt');

rs.on('readable', () => {
  let chunk;
  // null qaytguncha o'qiymiz — bu "hozircha boshqa data yo'q" degani
  while ((chunk = rs.read()) !== null) {
    console.log('o\'qildi:', chunk.length);
  }
});
rs.on('end', () => console.log('Tugadi'));
```

| Event | Qachon ishlaydi | Rejim |
|---|---|---|
| `'data'` | yangi bo'lak tayyor (avto push) | flowing |
| `'readable'` | buferda o'qishga ma'lumot bor | paused |
| `'end'` | boshqa o'qiladigan ma'lumot yo'q | har ikkisi |
| `'close'` | stream va resurslar yopildi | har ikkisi |
| `'error'` | xato yuz berdi | har ikkisi |

**⚠️ Ehtiyot bo'l:** Bir stream'ga ham `'data'` listener, ham `'readable'` listener qo'ymang — ular bir-biriga xalaqit beradi (`'readable'` qo'shilishi flowing rejimni o'chiradi). Bittasini tanlang. Zamonaviy kodda esa eng yaxshisi — `for await...of` (pastda).

---

## pipe() va pipeline()

**💡 Tushuncha:** `pipe()` Readable'dan Writable'ga ma'lumotni avtomatik uzatadi va **backpressure'ni o'zi boshqaradi**. Bu eng ko'p ishlatiladigan stream operatsiyasi.

```js
// Readable → Writable
fs.createReadStream('in.txt').pipe(fs.createWriteStream('out.txt'));

// Zanjir: o'qish → gzip → yozish
fs.createReadStream('in.txt')
  .pipe(zlib.createGzip())     // Transform
  .pipe(fs.createWriteStream('in.txt.gz'));
```

**Muammo:** `pipe()` xatolarni avtomatik tarqatmaydi va **resurslarni tozalamaydi**. Agar zanjirning o'rtasida xato bo'lsa, oldingi stream'lar ochiq qolib, fayl deskriptorlari **oqib ketadi** (resource leak).

**✅ Yechim — `pipeline()`:** xatolarni to'g'ri tarqatadi, barcha stream'larni tozalaydi va tugaganda callback/promise beradi.

```js
const { pipeline } = require('node:stream/promises');

async function gzipFile() {
  await pipeline(
    fs.createReadStream('in.txt'),
    zlib.createGzip(),
    fs.createWriteStream('in.txt.gz')
  );
  console.log('Tayyor'); // faqat hammasi muvaffaqiyatli tugasa
}
// Istalgan bosqichdagi xato bu yerda catch qilinadi
gzipFile().catch(console.error);
```

**⚠️ Ehtiyot bo'l:** Production'da **doim `pipeline()`** ishlating, `.pipe()` zanjirini emas. `.pipe()` da xato bo'lganda destination avtomatik yopilmaydi va leak yuzaga keladi. `pipeline()` esa har qanday holatda barcha stream'larni `destroy` qiladi.

---

## Backpressure — nima, nega muhim

**💡 Tushuncha:** **Backpressure** — bu manba (Readable) ma'lumotni manzil (Writable) qayta ishlab ulgurganidan **tezroq** ishlab chiqarganda yuzaga keladigan vaziyat. Misol: fayldan SSD tezligida o'qiysiz, lekin sekin tarmoqqa yozasiz. Agar tezlikni moslamasang, o'qilgan, lekin hali yozilmagan ma'lumot **RAM da yig'ilib boradi** va oxir-oqibat xotira tugaydi (OOM crash).

`.write()` metodi `false` qaytarsa — bu "ichki bufer to'ldi, sekinlash" signali. To'g'ri kodda siz manbani `pause()` qilishingiz, Writable `'drain'` event chiqargach `resume()` qilishingiz kerak.

```js
// Qo'lda backpressure boshqaruvi (pipe nima qilishini ko'rsatadi)
readable.on('data', (chunk) => {
  const ok = writable.write(chunk);
  if (!ok) {
    readable.pause();                       // bufer to'ldi → to'xtat
    writable.once('drain', () => readable.resume()); // bo'shadi → davom et
  }
});
```

**`pipe()` / `pipeline()` buni avtomatik qiladi:**

1. Readable'dan o'qiydi, Writable'ga yozadi.
2. `write()` `false` qaytarsa — Readable'ni avtomatik `pause()` qiladi.
3. Writable `'drain'` chiqargach — avtomatik `resume()` qiladi.

Natijada xotira iste'moli `highWaterMark` chegarasi atrofida doimiy qoladi.

**⚠️ Ehtiyot bo'l:** Eng tarqalgan xato — `.write()` ning qaytargan qiymatini **e'tiborsiz qoldirish** loopda. Quyidagi kod tez manbada RAM ni portlatadi:

```js
// ❌ Backpressure'ni e'tiborsiz qoldirdi — RAM portlaydi
for (const row of millionRows) {
  writable.write(row); // false qaytishini tekshirmaydi
}
```

Buning o'rniga `pipeline()`, async iterator yoki `'drain'` ni kuting.

---

## highWaterMark va buffering

**💡 Tushuncha:** Har bir stream ichki **bufer** ga ega. `highWaterMark` — bu buferning **chegarasi**: Readable uchun bu darajaga yetganda stream manbadan o'qishni to'xtatadi; Writable uchun bu darajaga yetganda `.write()` `false` qaytaradi (backpressure signali). Bu **qattiq limit emas**, balki "yetarli yig'ildi, sekinlash" ostonasi.

- Byte-rejim stream'lar uchun default: **64 KB** (65536 bayt).
- Object mode uchun default: **16 obyekt**.

```js
// Kichikroq bufer — tez-tez, kichik bo'laklar; kam RAM, ko'proq overhead
const rs = fs.createReadStream('big.bin', { highWaterMark: 16 * 1024 }); // 16 KB

// Kattaroq bufer — kam o'qish, ko'proq RAM, kam syscall
const rs2 = fs.createReadStream('big.bin', { highWaterMark: 1024 * 1024 }); // 1 MB
```

**⚠️ Ehtiyot bo'l:** `highWaterMark` ni juda katta qilish (masalan 256 MB) backpressure'ni amalda o'chiradi — stream juda ko'p RAM yig'adi. Juda kichik qilish esa syscall overhead'ini oshiradi. Default 64 KB ko'pchilik holatlar uchun yaxshi tanlangan; o'lchamasdan o'zgartirmang.

---

## Custom Readable yaratish

**💡 Tushuncha:** O'z manbangizdan (DB cursor, API, generator) ma'lumot beradigan stream yaratish uchun `Readable` dan meros olib `_read()` ni implement qilasiz yoki `Readable.from()` ishlatasiz. `_read()` ichida `this.push(chunk)` bilan ma'lumot berasiz; `this.push(null)` esa "tugadi" signalini bildiradi.

```js
const { Readable } = require('node:stream');

class CounterStream extends Readable {
  constructor(max, options) {
    super(options);
    this.current = 1;
    this.max = max;
  }
  _read() {
    if (this.current > this.max) {
      this.push(null);            // EOF — boshqa data yo'q
    } else {
      const buf = Buffer.from(`${this.current}\n`, 'utf8');
      this.push(buf);             // bo'lakni buferga qo'shamiz
      this.current++;
    }
  }
}

new CounterStream(5).pipe(process.stdout); // 1\n2\n3\n4\n5\n
```

Eng oson usul — iterator/generator'dan stream yasash:

```js
async function* generate() {
  for (let i = 1; i <= 5; i++) yield `${i}\n`;
}
Readable.from(generate()).pipe(process.stdout);
```

**⚠️ Ehtiyot bo'l:** `_read()` ni o'zingiz **chaqirmang** — uni Node ichki mexanizmi bufer bo'shaganda chaqiradi. `this.push()` `false` qaytarsa, bufer to'lgan — yangi `push` qilishni `_read` keyingi chaqirilguncha kuting. Aks holda backpressure buziladi.

---

## Custom Writable yaratish

**💡 Tushuncha:** Ma'lumotni o'z manzilingizga (DB, tashqi API, log) yozadigan stream uchun `Writable` dan meros olib `_write(chunk, encoding, callback)` ni implement qilasiz. `callback()` ni chaqirib "bu bo'lak qayta ishlandi, keyingisini ber" deysiz — bu **backpressure'ning asosi**.

```js
const { Writable } = require('node:stream');

class DbWriteStream extends Writable {
  constructor(options) {
    super(options);
  }
  _write(chunk, encoding, callback) {
    saveToDb(chunk.toString())
      .then(() => callback())      // muvaffaqiyat → keyingi bo'lak
      .catch((err) => callback(err)); // xato → stream'ga xato beradi
  }
}

fs.createReadStream('data.txt').pipe(new DbWriteStream());
```

Ko'p bo'lakni birvarakayiga optimallashtirish uchun `_writev(chunks, callback)` ham bor.

**⚠️ Ehtiyot bo'l:** `callback()` ni **albatta** chaqiring — agar unutsangiz, stream `_write` ni boshqa chaqirmaydi va butun pipeline **muzlab qoladi** (osilib qoladi). Xato bo'lsa `callback(err)` bilan uzating, `throw` qilmang — `throw` stream error handling'idan chetda qoladi.

---

## Custom Transform yaratish

**💡 Tushuncha:** Transform — kirish ma'lumotni o'zgartirib chiqaradigan stream. `Transform` dan meros olib `_transform(chunk, encoding, callback)` ni implement qilasiz: `this.push(natija)` bilan chiqarasiz, `callback()` bilan keyingi bo'lakka ruxsat berasiz. Ixtiyoriy `_flush(callback)` esa stream tugaganda qolgan ma'lumotni chiqarish uchun.

```js
const { Transform } = require('node:stream');

// Har bo'lakni katta harfga aylantiruvchi Transform
class UppercaseTransform extends Transform {
  _transform(chunk, encoding, callback) {
    const upper = chunk.toString().toUpperCase();
    this.push(upper);   // o'zgartirilgan natijani chiqaramiz
    callback();         // keyingi bo'lakka tayyor
  }
}

fs.createReadStream('in.txt')
  .pipe(new UppercaseTransform())
  .pipe(process.stdout);
```

`_flush` misoli — agar ichki holat (qolgan satr) bo'lsa:

```js
class LineSplitter extends Transform {
  constructor(opts) { super(opts); this.buf = ''; }
  _transform(chunk, enc, cb) {
    this.buf += chunk.toString();
    const lines = this.buf.split('\n');
    this.buf = lines.pop();             // oxirgi (to'liqsiz) qism qoladi
    for (const line of lines) this.push(line + '\n');
    cb();
  }
  _flush(cb) {                          // stream tugadi
    if (this.buf) this.push(this.buf);  // qolgan oxirgi qism
    cb();
  }
}
```

**⚠️ Ehtiyot bo'l:** `_transform` ichida `callback()` ni har doim chaqiring (xohlasa xato bilan). `callback(null, data)` — bu `this.push(data); callback()` ning qisqartmasi. `_flush` faqat **bir marta**, stream oxirida ishlaydi — qisman yig'ilgan ma'lumotni (yarim satr, oxirgi CSV qatori) shu yerda chiqaring, aks holda u yo'qoladi.

---

## Object mode

**💡 Tushuncha:** Default stream'lar faqat `Buffer` yoki `string` uzatadi. **Object mode** (`{ objectMode: true }`) yoqilganda esa stream **istalgan JS obyektini** bo'lak sifatida uzata oladi. Bu DB qatorlari, JSON obyektlari, parse qilingan CSV satrlari bilan ishlashda juda foydali.

```js
const { Transform } = require('node:stream');

const toJson = new Transform({
  objectMode: true,              // bo'laklar — obyektlar
  transform(obj, enc, cb) {
    cb(null, { ...obj, processed: true }); // obyekt chiqaramiz
  },
});

Readable.from([{ id: 1 }, { id: 2 }]) // Readable.from default objectMode
  .pipe(toJson)
  .on('data', (o) => console.log(o)); // { id: 1, processed: true }, ...
```

Object mode'da `highWaterMark` **obyektlar soni** bilan o'lchanadi (default 16), bayt bilan emas.

**⚠️ Ehtiyot bo'l:** Object mode stream'ni to'g'ridan-to'g'ri faylga yoki socketga `pipe` qila olmaysiz — fayl Buffer/string kutadi, obyekt emas. Object mode pipeline oxirida ma'lumotni `JSON.stringify` yoki serialize qiluvchi Transform qo'yish kerak.

---

## Stream'da error handling

**💡 Tushuncha:** Stream'lar `EventEmitter` bo'lgani uchun xatolar `'error'` event orqali keladi, `throw` qilinmaydi. Agar `'error'` listener bo'lmasa — Node butun jarayonni **crash** qiladi (uncaught exception). Va eng muhim: `.pipe()` xatolarni zanjir bo'ylab **tarqatmaydi**.

```js
// ❌ pipe da xato: faqat readable'ning xatosi catch qilinadi, writable leak bo'ladi
fs.createReadStream('in.txt')
  .on('error', console.error)
  .pipe(fs.createWriteStream('out.txt')); // bu yerdagi xato ushlanmaydi

// ✅ pipeline: barcha bosqich xatolari bitta joyda, resurslar tozalanadi
const { pipeline } = require('node:stream/promises');
try {
  await pipeline(
    fs.createReadStream('in.txt'),
    fs.createWriteStream('out.txt')
  );
} catch (err) {
  console.error('Pipeline xatosi:', err); // har qanday bosqich xatosi
}
```

**⚠️ Ehtiyot bo'l:** Har bir stream'ga `'error'` listener qo'ymasangiz, bitta xato butun serverni o'ldiradi. `.pipe()` zanjiri ishlatsangiz — **har bir** stream'ga alohida `'error'` qo'yish kerak, bu unutilishi oson. Shu sabab `pipeline()` afzal: u xatolarni markazlashtiradi va `destroy()` ni o'zi chaqiradi.

---

## Async iterator (for await...of)

**💡 Tushuncha:** Zamonaviy Node'da Readable stream **async iterable** — uni `for await...of` bilan o'qish mumkin. Bu eng toza usul: backpressure avtomatik (loop tanasi sekin ishlasa, stream kutadi), xato esa oddiy `try/catch` bilan ushlanadi. Hech qanday `'data'`/`'end'`/`'error'` listener kerak emas.

```js
async function readLines() {
  const rs = fs.createReadStream('big.log', { encoding: 'utf8' });
  try {
    for await (const chunk of rs) {   // har bo'lak; loop sekin bo'lsa stream pauza
      process(chunk);
    }
    console.log('Tugadi');
  } catch (err) {
    console.error('O\'qish xatosi:', err);
  }
}
```

`readline` bilan satrma-satr o'qish — juda keng tarqalgan pattern:

```js
const readline = require('node:readline');
const rl = readline.createInterface({
  input: fs.createReadStream('big.log'),
  crlfDelay: Infinity,
});
for await (const line of rl) {
  console.log('Satr:', line);
}
```

**⚠️ Ehtiyot bo'l:** `for await...of` loopdan `break` qilsangiz yoki xato otilsa, stream avtomatik `destroy` qilinadi — bu yaxshi. Lekin agar stream'ni `for await` da iste'mol qilsangiz, bir vaqtning o'zida `'data'` listener qo'shmang: ikkalasi bir manbadan o'qiy olmaydi, ma'lumot yo'qoladi.

---

## Real misollar

**1) Katta faylni o'zgartirib nusxalash (xotira-samarali):**

```js
const { pipeline } = require('node:stream/promises');
await pipeline(
  fs.createReadStream('10gb.log'),
  new UppercaseTransform(),
  fs.createWriteStream('10gb.upper.log')
); // 10 GB fayl, lekin RAM ~ bir necha 64 KB
```

**2) HTTP streaming (faylni javob sifatida oqizish):**

```js
http.createServer((req, res) => {
  res.writeHead(200, { 'Content-Type': 'video/mp4' });
  // pipe backpressure'ni boshqaradi: mijoz sekin bo'lsa, o'qish sekinlaydi
  fs.createReadStream('movie.mp4').pipe(res);
}).listen(3000);
```

**3) Gzip transform (siqib yozish va siqib jo'natish):**

```js
const zlib = require('node:zlib');

// Faylni siqib saqlash
await pipeline(
  fs.createReadStream('app.log'),
  zlib.createGzip(),                 // Transform stream
  fs.createWriteStream('app.log.gz')
);

// HTTP javobni siqib jo'natish
http.createServer((req, res) => {
  res.writeHead(200, { 'Content-Encoding': 'gzip' });
  pipeline(
    fs.createReadStream('big.json'),
    zlib.createGzip(),
    res,
    (err) => err && console.error(err)
  );
});
```

**4) CSV ni qatorma-qator stream qilib parse qilish:**

```js
const { Transform } = require('node:stream');

// Sodda CSV → obyekt Transform (object mode chiqish)
class CsvParser extends Transform {
  constructor(opts) {
    super({ ...opts, readableObjectMode: true });
    this.buf = '';
    this.headers = null;
  }
  _transform(chunk, enc, cb) {
    this.buf += chunk.toString();
    const lines = this.buf.split('\n');
    this.buf = lines.pop();           // to'liqsiz oxirgi satr
    for (const line of lines) {
      const cells = line.split(',');
      if (!this.headers) { this.headers = cells; continue; }
      const row = {};
      this.headers.forEach((h, i) => { row[h] = cells[i]; });
      this.push(row);                 // obyekt sifatida chiqaramiz
    }
    cb();
  }
}

await pipeline(
  fs.createReadStream('users.csv'),
  new CsvParser(),
  new Writable({
    objectMode: true,
    write(row, enc, cb) { console.log(row); cb(); },
  })
); // millionlab qatorli CSV ham doimiy RAM da
```

**⚠️ Ehtiyot bo'l:** Production'da CSV uchun `csv-parse`, siqishda esa kerakli darajani (`zlib.constants.Z_BEST_SPEED` vs `Z_BEST_COMPRESSION`) tanlang. Yuqoridagi CSV parser sodda — u qo'shtirnoq ichidagi vergullarni (`"a,b",c`) hisobga olmaydi; real loyihada tayyor kutubxona ishlating.

---

## Intervyu Q&A

### ❓ Stream nima va nega kerak?

**✅ Javob:** Stream — ma'lumotni **bo'lak-bo'lak** qayta ishlash abstraksiyasi. Asosiy foyda ikkita: (1) **xotira samaradorligi** — butun ma'lumotni RAM ga yuklamaysiz, doimiy hajmdagi bufer bilan ishlaysiz, shuning uchun gigabaytli fayllarni ham qayta ishlay olasiz; (2) **vaqt samaradorligi** — birinchi bo'lak kelishi bilan ish boshlanadi, butun yuklanishni kutmaysiz (past latency). Katta yoki noma'lum hajmli ma'lumotda (fayl, tarmoq, real-time) stream zarur.

### ❓ Stream'ning 4 turini ayting.

**✅ Javob:** **Readable** (manba, o'qish — `fs.createReadStream`, HTTP `req`), **Writable** (manzil, yozish — `fs.createWriteStream`, HTTP `res`), **Duplex** (mustaqil o'qish va yozish kanallari — TCP socket), va **Transform** (Duplex'ning maxsus turi, chiqish kirishdan hosil qilinadi — `zlib.createGzip`). Hammasi `EventEmitter` dan meros oladi.

### ❓ Flowing va paused rejim farqi nima?

**✅ Javob:** Paused (default) — ma'lumot stream ichida turadi, siz `.read()` bilan **tortib olasiz** (pull). Flowing — ma'lumot avtomatik oqib, `'data'` event orqali sizga **itariladi** (push). `'data'` listener qo'shish, `.pipe()` yoki `.resume()` stream'ni flowing'ga o'tkazadi; `.pause()` qaytaradi. Flowing'da iste'mol qilmasangiz ma'lumot yo'qoladi yoki RAM da yig'iladi.

### ❓ `.pipe()` va `pipeline()` farqi?

**✅ Javob:** Ikkalasi ham Readable'dan Writable'ga ma'lumot uzatadi va backpressure'ni boshqaradi. Lekin `.pipe()` **xatolarni tarqatmaydi** va xato bo'lganda stream'larni tozalamaydi — fayl deskriptorlari leak bo'ladi. `pipeline()` (yoki `stream/promises` dagi promise versiyasi) xatolarni markazlashtiradi, barcha stream'larni `destroy` qiladi va tugaganda callback/promise beradi. Production'da doim `pipeline()` ishlating.

### ❓ Backpressure nima va nega muhim?

**✅ Javob:** Backpressure — manba ma'lumotni manzil qayta ishlab ulgurganidan tezroq ishlab chiqarganda yuzaga keladigan vaziyat (masalan tez SSD'dan o'qib, sekin tarmoqqa yozish). Agar boshqarilmasa, o'qilgan-yu yozilmagan ma'lumot RAM da yig'ilib, xotira tugaydi (OOM). `.write()` `false` qaytarsa — "sekinlash" signali; manbani pauza qilib, `'drain'` event'ni kutish kerak. `pipe()`/`pipeline()` buni avtomatik bajaradi.

### ❓ `pipe()` backpressure'ni qanday hal qiladi?

**✅ Javob:** `pipe()` Readable'dan o'qib Writable'ga yozadi; agar `writable.write()` `false` qaytarsa (bufer to'ldi), u Readable'ni avtomatik `pause()` qiladi. Writable buferi bo'shab `'drain'` event chiqargach, `resume()` qiladi. Natijada xotira iste'moli `highWaterMark` atrofida doimiy qoladi va tezliklar moslashadi.

### ❓ `highWaterMark` nima?

**✅ Javob:** Stream ichki buferining ostonasi. Readable bu darajaga yetsa manbadan o'qishni to'xtatadi; Writable bu darajaga yetsa `.write()` `false` qaytaradi (backpressure signali). Bayt-rejimda default 64 KB, object mode'da 16 obyekt. Bu qattiq limit emas, "yetarli yig'ildi" ostonasi. Juda katta qilish backpressure'ni o'chiradi, juda kichik qilish syscall overhead'ini oshiradi.

### ❓ Custom Readable qanday yaratiladi?

**✅ Javob:** `Readable` dan meros olib `_read()` metodini implement qilasiz: ichida `this.push(chunk)` bilan ma'lumot berasiz, `this.push(null)` bilan tugashni bildirasiz. `_read()` ni o'zingiz chaqirmaysiz — Node bufer bo'shaganda chaqiradi. Eng oson usul — `Readable.from(iterable)` yoki async generator'dan stream yasash.

### ❓ Custom Writable'da `callback()` nima uchun kerak?

**✅ Javob:** `_write(chunk, encoding, callback)` da `callback()` "bu bo'lak qayta ishlandi, keyingisini ber" degani — bu backpressure'ning asosi. Agar `callback()` ni chaqirmasangiz, stream `_write` ni boshqa chaqirmaydi va butun pipeline muzlab qoladi. Xato bo'lsa `callback(err)` bilan uzating, `throw` qilmang.

### ❓ Object mode nima?

**✅ Javob:** `{ objectMode: true }` yoqilganda stream Buffer/string o'rniga **istalgan JS obyektini** bo'lak sifatida uzatadi. DB qatorlari, parse qilingan CSV/JSON obyektlari bilan ishlashda foydali. `highWaterMark` obyektlar soni bilan o'lchanadi (default 16). Object mode stream'ni to'g'ridan-to'g'ri faylga pipe qilib bo'lmaydi — oxirida serialize qiluvchi bosqich kerak.

### ❓ Stream'da xatoni qanday boshqarasiz?

**✅ Javob:** Xatolar `'error'` event orqali keladi, `throw` qilinmaydi. `'error'` listener bo'lmasa jarayon crash bo'ladi. `.pipe()` xatolarni tarqatmaydi, shuning uchun har stream'ga alohida `'error'` kerak — bu unutilishi oson. To'g'ri yechim — `pipeline()` ishlatish: u xatolarni bitta joyda ushlaydi va barcha stream'larni `destroy` qiladi.

### ❓ `for await...of` bilan stream o'qishning afzalligi nima?

**✅ Javob:** Readable async iterable bo'lgani uchun `for await...of` bilan o'qish mumkin: backpressure avtomatik (loop tanasi sekin bo'lsa stream kutadi), xato oddiy `try/catch` bilan ushlanadi, listener'lar kerak emas. `break` yoki xatoda stream avtomatik `destroy` qilinadi. Eng toza zamonaviy usul.

### ❓ Transform va Duplex farqi nima?

**✅ Javob:** Duplex — mustaqil o'qish va yozish kanallariga ega stream (TCP socket: yozasiz va alohida o'qiysiz, ular bog'liq emas). Transform — Duplex'ning maxsus turi, bunda chiqish **kirishdan hosil qilinadi** (yozasiz raw → gzip → o'qiysiz siqilgan). Transform'da `_transform()` kirishni o'zgartirib chiqaradi.

### ❓ Buffer string bilan ishlashda qanday muammo bor?

**✅ Javob:** Stream bo'laklari ko'p baytli UTF-8 belgini (emoji, kirill) o'rtasidan kesib qo'yishi mumkin — yarmi bir bo'lakda, yarmi keyingisida. Har bo'lakka `chunk.toString()` qo'llasangiz, buzilgan belgilar chiqadi. Yechim — `setEncoding('utf8')` yoki `StringDecoder`, ular chegaradagi belgilarni to'g'ri yig'adi.

### ❓ Nima uchun katta faylni `readFile` bilan o'qish xavfli?

**✅ Javob:** `readFile` butun faylni RAM ga yuklaydi. 2 GB fayl + 50 ta parallel so'rov = 100 GB RAM, server qulaydi. Stream esa faylni 64 KB bo'laklarga bo'lib oqizadi, RAM doimiy qoladi. Shuning uchun katta yoki ko'p mijozli holatda `createReadStream().pipe()` yoki `pipeline()` ishlatiladi.

---

## Masalalar

> Yechimlar: [solutions/backend/05-streams.md](../solutions/backend/05-streams.md)

1. **Counter Readable.** `Readable` dan meros olib, 1 dan `n` gacha sonlarni har birini alohida bo'lak qilib chiqaradigan custom Readable stream yozing. `this.push(null)` bilan to'g'ri yakunlang va `process.stdout` ga pipe qilib tekshiring.

2. **Uppercase Transform.** Kelgan har bir bo'lakni katta harfga aylantiruvchi custom `Transform` yozing. Uni `fs.createReadStream` va `process.stdout` orasiga `pipeline()` bilan ulang.

3. **Line counter.** Berilgan katta matn faylidagi qatorlar sonini stream orqali (butun faylni RAM ga yuklamasdan) sanaydigan funksiya yozing. `for await...of` yoki `readline` ishlating.

4. **Gzip qilib nusxalash.** Berilgan fayl yo'lini olib, uni gzip bilan siqib `<fayl>.gz` ga yozadigan async funksiya yozing. `stream/promises` dagi `pipeline()` va `zlib.createGzip()` ishlating, xatoni `try/catch` bilan ushlang.

5. **CSV → obyekt Transform.** Object mode chiqishli `Transform` yozing: birinchi qatorni header sifatida oladi, qolgan har bir qatorni `{ header: value, ... }` obyektiga aylantirib chiqaradi. To'liqsiz oxirgi qatorni `_flush` da to'g'ri chiqaring.

6. **Backpressure'ni qo'lda boshqarish.** `pipe()` ishlatmasdan, `'data'`, `pause()`, `'drain'` va `resume()` yordamida Readable'dan Writable'ga backpressure'ni hisobga olgan holda ma'lumot uzatuvchi funksiya yozing.

7. **Throttle Transform.** Har bir bo'lakni chiqarishdan oldin `ms` millisekund kutadigan (tezlikni cheklovchi) `Transform` yozing. `_transform` ichida `setTimeout` dan keyin `callback()` chaqiring.

8. **JSON satr filtri.** NDJSON (har qatorda bitta JSON obyekt) faylini o'qib, faqat berilgan shartga mos obyektlarni (masalan `active === true`) chiqaradigan stream pipeline'ini yozing: o'qish → split → parse → filter → yozish.

← [Backend bo'limiga qaytish](./README.md)
