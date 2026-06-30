# Streams — Masalalar Yechimlari

Bu fayl [`backend/05-streams.md`](../../backend/05-streams.md) dagi "Masalalar" bo'limining yechimlarini o'z ichiga oladi. Har bir yechim izohlangan va asosiy nuqtalar tushuntirilgan.

## Mundarija

- [1. Counter Readable](#1-counter-readable)
- [2. Uppercase Transform](#2-uppercase-transform)
- [3. Line counter](#3-line-counter)
- [4. Gzip qilib nusxalash](#4-gzip-qilib-nusxalash)
- [5. CSV → obyekt Transform](#5-csv--obyekt-transform)
- [6. Backpressure'ni qo'lda boshqarish](#6-backpressureni-qolda-boshqarish)
- [7. Throttle Transform](#7-throttle-transform)
- [8. JSON satr filtri](#8-json-satr-filtri)

---

## 1. Counter Readable

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
      this.push(null);                          // EOF — tugadi
      return;
    }
    this.push(Buffer.from(`${this.current}\n`, 'utf8'));
    this.current++;
  }
}

new CounterStream(5).pipe(process.stdout);      // 1\n2\n3\n4\n5\n
```

**Asosiy nuqtalar:** `_read()` ni Node o'zi bufer bo'shaganda chaqiradi — uni qo'lda chaqirmaymiz. `this.push(null)` "boshqa data yo'q" signalini bildiradi va `'end'` event'ini keltirib chiqaradi. Har `_read` da bittadan bo'lak push qilamiz.

Generator orqali soddaroq alternativa:

```js
const { Readable } = require('node:stream');
function counter(max) {
  return Readable.from((function* () {
    for (let i = 1; i <= max; i++) yield `${i}\n`;
  })());
}
counter(5).pipe(process.stdout);
```

---

## 2. Uppercase Transform

```js
const fs = require('node:fs');
const { Transform } = require('node:stream');
const { pipeline } = require('node:stream/promises');

class UppercaseTransform extends Transform {
  _transform(chunk, encoding, callback) {
    // callback(null, data) === this.push(data); callback()
    callback(null, chunk.toString().toUpperCase());
  }
}

async function run() {
  await pipeline(
    fs.createReadStream('in.txt'),
    new UppercaseTransform(),
    process.stdout
  );
}
run().catch(console.error);
```

**Asosiy nuqtalar:** `callback(null, data)` — `this.push(data); callback()` ning qisqartmasi. `pipeline()` xatolarni markazlashtiradi va stream'larni tozalaydi. Diqqat: ko'p baytli belgilar bo'lak chegarasida buzilmasligi uchun real kodda `setEncoding('utf8')` yoki `StringDecoder` qo'shish mumkin.

---

## 3. Line counter

```js
const fs = require('node:fs');
const readline = require('node:readline');

async function countLines(path) {
  const rl = readline.createInterface({
    input: fs.createReadStream(path),
    crlfDelay: Infinity,                // \r\n ni bitta qator deb hisoblaydi
  });
  let count = 0;
  for await (const _line of rl) {
    count++;
  }
  return count;
}

countLines('big.log').then((n) => console.log('Qatorlar:', n));
```

**Asosiy nuqtalar:** `readline` butun faylni RAM ga yuklamaydi — u stream ustida ishlab, satrma-satr beradi. RAM iste'moli faylning eng uzun satri darajasida doimiy qoladi. `for await...of` backpressure'ni avtomatik boshqaradi.

`for await` bilan qo'lda variant (newline'larni sanab):

```js
async function countLines2(path) {
  const rs = fs.createReadStream(path, { encoding: 'utf8' });
  let count = 0;
  for await (const chunk of rs) {
    for (const ch of chunk) if (ch === '\n') count++;
  }
  return count;
}
```

---

## 4. Gzip qilib nusxalash

```js
const fs = require('node:fs');
const zlib = require('node:zlib');
const { pipeline } = require('node:stream/promises');

async function gzipFile(path) {
  try {
    await pipeline(
      fs.createReadStream(path),
      zlib.createGzip(),
      fs.createWriteStream(`${path}.gz`)
    );
    console.log(`Siqildi: ${path}.gz`);
  } catch (err) {
    console.error('Gzip xatosi:', err);
    throw err;
  }
}

gzipFile('app.log');
```

**Asosiy nuqtalar:** `zlib.createGzip()` — Transform stream. `pipeline()` uchta bosqichni ulaydi, backpressure'ni boshqaradi va istalgan bosqich xatosini `try/catch` ga tashlaydi. 10 GB fayl ham doimiy RAM da siqiladi. Siqish darajasini `zlib.createGzip({ level: zlib.constants.Z_BEST_COMPRESSION })` bilan sozlash mumkin.

---

## 5. CSV → obyekt Transform

```js
const fs = require('node:fs');
const { Transform, Writable } = require('node:stream');
const { pipeline } = require('node:stream/promises');

class CsvParser extends Transform {
  constructor(opts) {
    super({ ...opts, readableObjectMode: true }); // chiqish — obyekt
    this.buf = '';
    this.headers = null;
  }
  _transform(chunk, enc, cb) {
    this.buf += chunk.toString();
    const lines = this.buf.split('\n');
    this.buf = lines.pop();                 // to'liqsiz oxirgi satr qoladi
    for (const line of lines) this._emitRow(line);
    cb();
  }
  _flush(cb) {
    if (this.buf.trim()) this._emitRow(this.buf); // qolgan oxirgi satr
    cb();
  }
  _emitRow(line) {
    const cells = line.split(',').map((c) => c.trim());
    if (!this.headers) { this.headers = cells; return; }
    const row = {};
    this.headers.forEach((h, i) => { row[h] = cells[i]; });
    this.push(row);
  }
}

async function run() {
  await pipeline(
    fs.createReadStream('users.csv'),
    new CsvParser(),
    new Writable({
      objectMode: true,
      write(row, enc, cb) { console.log(row); cb(); },
    })
  );
}
run().catch(console.error);
```

**Asosiy nuqtalar:** `readableObjectMode: true` chiqishni obyekt rejimiga o'tkazadi (kirish hali ham Buffer). To'liqsiz oxirgi satrni `this.buf` da saqlab, faqat to'liq satrlarni qayta ishlaymiz; `_flush` da fayl oxiridagi qolgan satrni chiqaramiz — busiz oxirgi qator yo'qoladi. Eslatma: bu sodda parser qo'shtirnoq ichidagi vergulni (`"a,b"`) hisobga olmaydi — production'da `csv-parse` ishlating.

---

## 6. Backpressure'ni qo'lda boshqarish

```js
function copyWithBackpressure(readable, writable) {
  return new Promise((resolve, reject) => {
    readable.on('data', (chunk) => {
      const ok = writable.write(chunk);
      if (!ok) {
        readable.pause();                       // bufer to'ldi → to'xtat
        writable.once('drain', () => readable.resume()); // bo'shadi → davom
      }
    });
    readable.on('end', () => writable.end());
    readable.on('error', reject);
    writable.on('error', reject);
    writable.on('finish', resolve);
  });
}

// Tekshirish
const fs = require('node:fs');
copyWithBackpressure(
  fs.createReadStream('in.txt'),
  fs.createWriteStream('out.txt')
).then(() => console.log('Tayyor'));
```

**Asosiy nuqtalar:** Bu aslida `pipe()` ichida nima sodir bo'lishini ko'rsatadi. `write()` `false` qaytarsa — manbani `pause()` qilamiz, `'drain'` kutamiz, keyin `resume()`. `end` da `writable.end()` chaqiramiz va `finish` da resolve. Har ikkala stream'ning `'error'` ini ushlash muhim — busiz xato jarayonni crash qiladi.

---

## 7. Throttle Transform

```js
const { Transform } = require('node:stream');

class ThrottleTransform extends Transform {
  constructor(ms, opts) {
    super(opts);
    this.ms = ms;
  }
  _transform(chunk, enc, callback) {
    // callback'ni kechiktirib chaqiramiz → keyingi bo'lak ms dan keyin keladi
    setTimeout(() => callback(null, chunk), this.ms);
  }
}

// Tekshirish: har bo'lak 500 ms oraliq bilan chiqadi
const { Readable } = require('node:stream');
Readable.from(['a\n', 'b\n', 'c\n'])
  .pipe(new ThrottleTransform(500))
  .pipe(process.stdout);
```

**Asosiy nuqtalar:** `callback` ni `setTimeout` ichida chaqirish stream'ni sekinlatadi — Node `_transform` tugaguncha (callback chaqirilguncha) keyingi bo'lakni bermaydi. Bu tabiiy ravishda backpressure orqali manbani ham sekinlatadi. Cheksiz buffering yo'q.

---

## 8. JSON satr filtri

```js
const fs = require('node:fs');
const readline = require('node:readline');
const { Readable, Transform, Writable } = require('node:stream');
const { pipeline } = require('node:stream/promises');

// readline'ni async iterable manba sifatida ishlatib, filtrlaymiz
async function filterNdjson(inputPath, outputPath, predicate) {
  const rl = readline.createInterface({
    input: fs.createReadStream(inputPath),
    crlfDelay: Infinity,
  });

  // readline → obyekt Readable
  const source = Readable.from((async function* () {
    for await (const line of rl) {
      if (!line.trim()) continue;
      yield JSON.parse(line);             // parse
    }
  })());

  const filter = new Transform({
    objectMode: true,
    transform(obj, enc, cb) {
      if (predicate(obj)) this.push(obj); // shartga mos bo'lsa o'tkazamiz
      cb();
    },
  });

  const sink = new Writable({
    objectMode: true,
    write(obj, enc, cb) {
      this.out.write(JSON.stringify(obj) + '\n'); // serialize → yozish
      cb();
    },
  });
  sink.out = fs.createWriteStream(outputPath);
  sink.on('finish', () => sink.out.end());

  await pipeline(source, filter, sink);
}

// Tekshirish: faqat active === true bo'lganlarni saqlaymiz
filterNdjson('users.ndjson', 'active.ndjson', (u) => u.active === true)
  .then(() => console.log('Filtrlandi'))
  .catch(console.error);
```

**Asosiy nuqtalar:** Pipeline bosqichlari: o'qish (readline) → parse (generator `JSON.parse`) → filter (object-mode Transform) → serialize + yozish (Writable). Hammasi stream bo'lgani uchun millionlab qatorli NDJSON ham doimiy RAM da ishlanadi. `pipeline()` xato va tozalashni boshqaradi. Eslatma: `predicate` — filtrlash sharti funksiyasi, masalan `(u) => u.active === true`.

---

← [Backend bo'limiga qaytish](../../backend/README.md)
