# Character Encoding — Yechimlar

Har bir masala uchun to'liq yechim va o'zbekcha izoh.

---

## 1. Belgi info (decimal + hex)

```js
function info(ch) {
  const code = ch.codePointAt(0);
  return {
    decimal: code,
    hex: '0x' + code.toString(16).toUpperCase(),
  };
}

info('A'); // => { decimal: 65, hex: '0x41' }
info('z'); // => { decimal: 122, hex: '0x7A' }
```

**Izoh:** `codePointAt(0)` belgining Unicode code point'ini decimal son sifatida qaytaradi. `toString(16)` uni hex string'ga aylantiradi. ASCII belgilar uchun `charCodeAt` ham ishlaydi, lekin `codePointAt` emoji uchun ham to'g'ri (surrogate'ni birlashtiradi).

---

## 2. isAscii (faqat printable 32–126)

```js
function isAscii(str) {
  for (const ch of str) {
    const code = ch.codePointAt(0);
    if (code < 32 || code > 126) return false;
  }
  return true;
}

isAscii('Hello!');  // => true
isAscii('Salom 😀'); // => false (emoji ASCII emas)
isAscii('a\tb');    // => false (tab = 9, printable emas)
```

**Izoh:** Har bir belgining code point'i 32–126 oralig'ida bo'lishini tekshiramiz. `for...of` ishlatamiz, chunki u code point bo'yicha yuradi va emoji kabi belgilarni butun holicha ko'radi.

---

## 3. toUpper (ASCII arifmetikasi bilan)

```js
function toUpper(str) {
  let result = '';
  for (const ch of str) {
    const code = ch.charCodeAt(0);
    // faqat 'a'(97)–'z'(122) oralig'idagilarni o'zgartiramiz:
    if (code >= 97 && code <= 122) {
      result += String.fromCharCode(code - 32);
    } else {
      result += ch;
    }
  }
  return result;
}

toUpper('salom123'); // => 'SALOM123'
```

**Izoh:** ASCII'da `'a'` (97) va `'A'` (65) farqi aniq 32. Demak kichik harfdan 32 ni ayirsak — katta harf. Faqat kichik lotin harflarini (97–122) tekshiramiz, qolganlarini (raqam, belgi) o'zgarishsiz qoldiramiz.

---

## 4. charCount (code point soni)

```js
function charCount(str) {
  return [...str].length;
}

charCount('a😀b');   // => 3
charCount('hello'); // => 5
'a😀b'.length;       // => 4 (noto'g'ri — surrogate pair sababli)
```

**Izoh:** `str.length` UTF-16 code unit sonini beradi, shuning uchun emoji 2 deb sanaladi. Spread (`[...str]`) string iterator'idan foydalanadi — u code point bo'yicha yuradi, surrogate pair'ni bitta element deb oladi. `Array.from(str).length` ham bir xil ishlaydi.

---

## 5. encodeUtf8 (code point → UTF-8 byte[])

```js
function encodeUtf8(cp) {
  if (cp <= 0x7F) {
    return [cp];
  } else if (cp <= 0x7FF) {
    return [
      0xC0 | (cp >> 6),
      0x80 | (cp & 0x3F),
    ];
  } else if (cp <= 0xFFFF) {
    return [
      0xE0 | (cp >> 12),
      0x80 | ((cp >> 6) & 0x3F),
      0x80 | (cp & 0x3F),
    ];
  } else {
    return [
      0xF0 | (cp >> 18),
      0x80 | ((cp >> 12) & 0x3F),
      0x80 | ((cp >> 6) & 0x3F),
      0x80 | (cp & 0x3F),
    ];
  }
}

encodeUtf8(0x41).map(b => b.toString(16));    // => ['41']            ('A')
encodeUtf8(0x20AC).map(b => b.toString(16));  // => ['e2','82','ac']  ('€')
encodeUtf8(0x1F600).map(b => b.toString(16)); // => ['f0','9f','98','80'] ('😀')

// tekshirish:
[...new TextEncoder().encode('😀')]; // => [240, 159, 152, 128] = f0 9f 98 80
```

**Izoh:** UTF-8 qoidasiga ko'ra code point diapazoniga qarab byte soni tanlanadi. Leading byte prefiks bit'lari (`0xC0`=`110…`, `0xE0`=`1110…`, `0xF0`=`11110…`) bilan, davom byte'lari `0x80`=`10…` bilan boshlanadi. `>> n` bilan kerakli bit guruhini olamiz, `& 0x3F` (=`00111111`) bilan pastki 6 bitni ajratamiz.

---

## 6. stripBom

```js
function stripBom(str) {
  if (str.charCodeAt(0) === 0xFEFF) {
    return str.slice(1);
  }
  return str;
}

stripBom('﻿hello'); // => 'hello'
stripBom('hello');       // => 'hello' (o'zgarmaydi)
```

**Izoh:** BOM belgisi U+FEFF (decimal 65279). Birinchi belgini tekshirib, agar BOM bo'lsa `slice(1)` bilan olib tashlaymiz. Bu JSON.parse yoki CSV o'qishdan oldin foydali.

---

## 7. countCodePoints (UTF-8 byte[] dan)

```js
function countCodePoints(bytes) {
  let count = 0;
  for (const b of bytes) {
    // continuation byte'lar 10xxxxxx (0x80–0xBF) — ularni sanamaymiz:
    if ((b & 0xC0) !== 0x80) {
      count++;
    }
  }
  return count;
}

const bytes = new TextEncoder().encode('a😀b'); // [97, 240,159,152,128, 98]
countCodePoints(bytes); // => 3
```

**Izoh:** UTF-8'da har bir belgi bitta leading byte va 0–3 ta continuation byte'dan iborat. Continuation byte'lar `10xxxxxx` pattern'iga ega (yuqori 2 bit = `10`). `b & 0xC0` (yuqori 2 bitni ajratish) `0x80` ga teng bo'lsa — bu continuation byte, sanamaymiz. Faqat leading byte'larni sanaymiz = belgilar soni.

---

## 8. Mojibake simulyatsiyasi

```js
function simulateMojibake(str) {
  // 1. UTF-8 da to'g'ri encode:
  const bytes = new TextEncoder().encode(str);
  // 2. har bir byte = 1 belgi deb (Latin-1) noto'g'ri decode:
  return new TextDecoder('latin1').decode(bytes);
}

simulateMojibake('Привет'); // => 'Ð\x9FÑ€Ð¸Ð²ÐµÑ‚' (mojibake)
simulateMojibake('café');   // => 'cafÃ©'
```

**Izoh:** `Привет` UTF-8'da har bir kirill harfi 2 byte bilan kodlanadi. Latin-1 esa har bir byte'ni alohida belgi deb o'qiydi, shuning uchun 1 belgi → 2 buzilgan belgi bo'ladi. Bu real mojibake'ning aniq sababi — encode va decode encoding'lari mos kelmaydi.

---

## 9. toSurrogatePair

```js
function toSurrogatePair(cp) {
  if (cp <= 0xFFFF) {
    throw new Error('BMP belgilar surrogate pair talab qilmaydi');
  }
  const offset = cp - 0x10000;
  const high = 0xD800 + (offset >> 10);        // yuqori 10 bit
  const low  = 0xDC00 + (offset & 0x3FF);      // pastki 10 bit
  return {
    high: '0x' + high.toString(16).toUpperCase(),
    low:  '0x' + low.toString(16).toUpperCase(),
  };
}

toSurrogatePair(0x1F600); // => { high: '0xD83D', low: '0xDE00' }

// tekshirish:
'😀'.charCodeAt(0).toString(16); // => 'd83d'
'😀'.charCodeAt(1).toString(16); // => 'de00'
```

**Izoh:** U+FFFF dan katta code point'dan 0x10000 ayiriladi (20-bitli offset chiqadi). Yuqori 10 bit high surrogate base'ga (`0xD800`), pastki 10 bit (`& 0x3FF` = pastki 10 bit) low surrogate base'ga (`0xDC00`) qo'shiladi. Bu UTF-16'ning surrogate pair formulasi.

---

## 10. truncateBytes (UTF-8, belgini buzmasdan)

```js
function truncateBytes(str, maxBytes) {
  const encoder = new TextEncoder();
  let result = '';
  let bytesUsed = 0;
  for (const ch of str) {              // code point bo'yicha yuramiz
    const chBytes = encoder.encode(ch).length;
    if (bytesUsed + chBytes > maxBytes) break;
    result += ch;
    bytesUsed += chBytes;
  }
  return result;
}

truncateBytes('Salom😀', 6); // => 'Salom'  (😀 = 4 byte, sig'maydi)
truncateBytes('Salom😀', 9); // => 'Salom😀' (5 + 4 = 9 byte)
truncateBytes('café', 4);    // => 'caf'    (é = 2 byte, 3+2=5 > 4)
```

**Izoh:** `for...of` bilan har bir to'liq belgini (code point) ko'rib chiqamiz. Har bir belgining UTF-8'dagi byte uzunligini hisoblab, agar qo'shsak limit'dan oshib ketsa — to'xtaymiz. Shu sababli multi-byte belgi (emoji, `é`) hech qachon yarmidan kesilmaydi.

---

← [Encoding bo'limiga qaytish](../../encoding/README.md)
