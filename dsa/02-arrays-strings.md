# Arrays va Strings

Array va string — intervyuda eng ko'p uchraydigan ma'lumot tuzilmalari. Bu yerda array'ning xotira tuzilishi, JS metodlari complexity'si, string immutability, va eng muhim pattern'lar — **two pointers**, **sliding window**, **prefix sum**, **in-place** o'zgartirish va string manipulyatsiyasini shablon kodlar bilan ko'rib chiqamiz. Junior va mid darajadagi savollarning katta qismi shu mavzudan keladi.

## Mundarija

- [Array tuzilishi va xotira](#array-tuzilishi-va-xotira)
- [JS array metodlari complexity](#js-array-metodlari-complexity)
- [String immutability JS'da](#string-immutability-jsda)
- [Pattern: Two Pointers](#pattern-two-pointers)
- [Pattern: Sliding Window](#pattern-sliding-window)
- [Pattern: Prefix Sum](#pattern-prefix-sum)
- [In-place o'zgartirish](#in-place-ozgartirish)
- [String manipulyatsiya](#string-manipulyatsiya)
- [Array vs Linked List trade-off](#array-vs-linked-list-trade-off)
- [Savol-javoblar](#savol-javoblar)
- [Masalalar](#masalalar)

---

## Array tuzilishi va xotira

Array — xotirada **ketma-ket** (contiguous) joylashgan elementlar to'plami. Har element bir xil hajmda bo'lgani uchun `i`-element manzilini `boshlanish + i·element_hajmi` formulasi bilan **bevosita** topish mumkin — shu sababli index orqali kirish `O(1)`.

**💡 Tushuncha:** Index orqali `arr[i]` ga kirish `O(1)` — bu array'ning eng kuchli tomoni. Sababi: element manzili matematik hisoblanadi, qidirish kerak emas.

**Dynamic array (amortized).** JS array'lari dinamik — oldindan o'lcham belgilanmaydi. Ichki bufer to'lganida ikki barobar kattalashtirilib elementlar ko'chiriladi (`O(n)`), lekin bu kamdan-kam. Shu sababli `push` **amortized O(1)** (jami `n` ta push `O(n)`).

```js
const arr = []; // dinamik o'sadi
arr.push(1);    // amortized O(1)
arr.push(2);    // ba'zan resize bo'ladi, lekin o'rtacha O(1)
console.log(arr[0]); // O(1) — bevosita index access
```

**⚠️ Ehtiyot bo'l:** JS array'lari aslida "haqiqiy" array emas — ular obyekt asosida, "sparse" (siyrak) bo'lishi mumkin (`arr[100] = 1` bo'sh joylar yaratadi). Lekin oddiy ketma-ket ishlatishda yuqoridagi complexity'lar amal qiladi.

## JS array metodlari complexity

| Metod | Complexity | Izoh |
|-------|-----------|------|
| `arr[i]` (access) | `O(1)` | Index orqali bevosita |
| `push(x)` | `O(1)` amortized | Oxiriga qo'shadi |
| `pop()` | `O(1)` | Oxiridan oladi |
| `shift()` | `O(n)` | Boshidan oladi — qolganlar **suriladi** |
| `unshift(x)` | `O(n)` | Boshiga qo'shadi — qolganlar suriladi |
| `splice(i, d, ...)` | `O(n)` | Surish/ko'chirish kerak |
| `slice(i, j)` | `O(n)` | Yangi nusxa yaratadi |
| `indexOf` / `includes` | `O(n)` | Chiziqli qidiruv |
| `concat` | `O(n + m)` | Yangi array |
| `sort` | `O(n log n)` | Timsort |
| `map` / `filter` / `forEach` | `O(n)` | Har element ustida |
| `find` / `some` / `every` | `O(n)` | Worst case |

**⚠️ Ehtiyot bo'l:** `shift` va `unshift` — `O(n)`! Tsikl ichida `arr.shift()` ishlatish `O(n²)` ga olib keladi. Boshidan tez olish kerak bo'lsa, indeks (pointer) ishlatish yoki teskari tomondan `pop` qilish, yoki queue uchun maxsus struktura afzal.

```js
// YOMON: O(n²) — har shift O(n)
while (arr.length) process(arr.shift());

// YAXSHI: O(n) — pointer bilan
for (let i = 0; i < arr.length; i++) process(arr[i]);
```

## String immutability JS'da

JS'da string'lar **immutable** (o'zgarmas). String'ning bitta belgisini o'zgartirib bo'lmaydi — har "o'zgartirish" **yangi string** yaratadi.

```js
let s = "hello";
s[0] = "H";         // ishlamaydi! string o'zgarmaydi
console.log(s);     // "hello"

s = "H" + s.slice(1); // yangi string yaratadi → "Hello"
```

**💡 Tushuncha:** String'ni belgi-belgi qurish (`result += char` tsikl ichida) har safar yangi string yaratgani uchun `O(n²)` bo'lishi mumkin. To'g'risi — belgilarni **array**'ga yig'ib, oxirida `arr.join("")` qilish (`O(n)`).

```js
// YOMON: O(n²) — har += yangi string
let result = "";
for (const ch of input) result += ch.toUpperCase();

// YAXSHI: O(n) — array buffer
const parts = [];
for (const ch of input) parts.push(ch.toUpperCase());
const result2 = parts.join("");
```

**⚠️ Ehtiyot bo'l:** String manipulyatsiyada `split("")`, `join`, `slice` ko'pincha `O(n)` xotira yaratadi. Two pointers bilan ko'pincha qo'shimcha xotirasiz (`O(1)`) ishlash mumkin (lekin string immutable bo'lgani uchun array'ga aylantirish kerak bo'lishi mumkin).

## Pattern: Two Pointers

Ikki pointer (indeks) bir vaqtning o'zida massiv/string bo'ylab harakatlanadi. Ko'pincha `O(n²)` brute force'ni `O(n)` ga tushiradi. Asosiy turlari: **qarama-qarshi** (chetdan markazga) va **bir yo'nalishli** (slow/fast).

**Shablon — qarama-qarshi pointerlar:**

```js
function twoPointersOpposite(arr) {
  let left = 0, right = arr.length - 1;
  while (left < right) {
    // ... left va right bilan ish
    if (/* shartga ko'ra */ true) left++;
    else right--;
  }
}
```

**Misol 1 — Two Sum (saralangan massivda):** `O(n)` time, `O(1)` space.

```js
function twoSumSorted(arr, target) {
  let l = 0, r = arr.length - 1;
  while (l < r) {
    const sum = arr[l] + arr[r];
    if (sum === target) return [l, r];
    if (sum < target) l++;   // kichik → chapni surish
    else r--;                // katta → o'ngni surish
  }
  return [-1, -1];
}
```

**Misol 2 — Reverse (in-place):** `O(n)` time, `O(1)` space.

```js
function reverse(arr) {
  let l = 0, r = arr.length - 1;
  while (l < r) {
    [arr[l], arr[r]] = [arr[r], arr[l]];
    l++; r--;
  }
  return arr;
}
```

**Misol 3 — Palindrome tekshirish:** `O(n)` time, `O(1)` space.

```js
function isPalindrome(s) {
  let l = 0, r = s.length - 1;
  while (l < r) {
    if (s[l] !== s[r]) return false;
    l++; r--;
  }
  return true;
}
```

**💡 Tushuncha:** Two pointers ishlashi uchun ko'pincha massiv **saralangan** bo'lishi kerak (yoki muammo strukturasi monotonlikka imkon berishi kerak). Saralangan bo'lsa, yig'indi kichikligi/kattaligiga qarab qaysi pointerni surishni aniq bilamiz.

## Pattern: Sliding Window

Massiv/string bo'ylab **uzluksiz oyna** (subarray/substring) siljiydi. Brute force `O(n²)` o'rniga `O(n)`. Ikki tur: **fixed** (o'lcham doimiy) va **variable** (oyna kengayadi/qisqaradi).

**Shablon — variable window:**

```js
function variableWindow(arr) {
  let left = 0, best = 0;
  // window holatini saqlovchi struktura (sum, Set, Map ...)
  for (let right = 0; right < arr.length; right++) {
    // 1. right elementni oynaga qo'sh
    while (/* oyna shartni buzdi */ false) {
      // 2. chapdan qisqart
      left++;
    }
    // 3. natijani yangila
    best = Math.max(best, right - left + 1);
  }
  return best;
}
```

**Misol 1 — Fixed window: hajmi k bo'lgan eng katta subarray yig'indisi:** `O(n)` time, `O(1)` space.

```js
function maxSubarraySum(arr, k) {
  let windowSum = 0;
  for (let i = 0; i < k; i++) windowSum += arr[i]; // birinchi oyna
  let max = windowSum;
  for (let i = k; i < arr.length; i++) {
    windowSum += arr[i] - arr[i - k]; // yangi kir, eski chiq
    max = Math.max(max, windowSum);
  }
  return max;
}
```

**Misol 2 — Variable window: takrorlanmaydigan eng uzun substring:** `O(n)` time, `O(min(n, alphabet))` space.

```js
function longestUniqueSubstring(s) {
  const seen = new Set();
  let left = 0, best = 0;
  for (let right = 0; right < s.length; right++) {
    while (seen.has(s[right])) {  // takror bor — chapdan qisqart
      seen.delete(s[left]);
      left++;
    }
    seen.add(s[right]);
    best = Math.max(best, right - left + 1);
  }
  return best;
}
```

**💡 Tushuncha:** Fixed window'da har qadamda bitta element kiritib bittasini chiqaramiz (`O(1)` yangilanish). Variable window'da o'ng pointer hamisha oldinga, chap pointer faqat kerak bo'lganda oldinga suriladi — har element ko'pi bilan ikki marta ko'riladi, demak `O(n)`.

## Pattern: Prefix Sum

Oldindan **yig'indilar massivi** tuzib, har qanday oraliq yig'indisini `O(1)` da olamiz. Ko'p marta oraliq so'rovlar bo'lganda foydali.

```js
function buildPrefix(arr) {
  const prefix = [0]; // prefix[0] = 0
  for (let i = 0; i < arr.length; i++)
    prefix.push(prefix[i] + arr[i]);
  return prefix; // O(n) qurish
}

// [i, j] oralig'i yig'indisi (j inclusive) — O(1)
function rangeSum(prefix, i, j) {
  return prefix[j + 1] - prefix[i];
}
```

**Misol — Subarray sum = k bo'lganlar soni:** `O(n)` time, `O(n)` space (hash map bilan).

```js
function subarraySumCount(arr, k) {
  const seen = new Map([[0, 1]]); // prefix → necha marta uchragan
  let sum = 0, count = 0;
  for (const x of arr) {
    sum += x;
    if (seen.has(sum - k)) count += seen.get(sum - k);
    seen.set(sum, (seen.get(sum) || 0) + 1);
  }
  return count;
}
```

**💡 Tushuncha:** `[i..j]` yig'indisi = `prefix[j+1] - prefix[i]`. Prefix sum'ni hash map bilan birlashtirsak, "yig'indisi k ga teng subarray" muammosini `O(n)` da yechamiz — bu intervyuda kuchli usul.

## In-place o'zgartirish

Massivni **qo'shimcha xotirasiz** (`O(1)` space) joyida o'zgartirish. Ko'pincha pointer texnikasi bilan.

**Misol — Nollarni oxiriga surish (Move Zeroes), tartibni saqlab:** `O(n)` time, `O(1)` space.

```js
function moveZeroes(arr) {
  let insert = 0; // noldan boshqa elementlar joyi
  for (let i = 0; i < arr.length; i++) {
    if (arr[i] !== 0) {
      [arr[insert], arr[i]] = [arr[i], arr[insert]];
      insert++;
    }
  }
  return arr;
}
```

**Misol — Saralangan massivdan dublikatlarni o'chirish:** `O(n)` time, `O(1)` space.

```js
function removeDuplicates(arr) {
  if (arr.length === 0) return 0;
  let write = 1;
  for (let read = 1; read < arr.length; read++) {
    if (arr[read] !== arr[write - 1]) {
      arr[write++] = arr[read];
    }
  }
  return write; // yangi uzunlik
}
```

**⚠️ Ehtiyot bo'l:** In-place yechimlarda "write" va "read" pointerlarini aralashtirib yubormaslik kerak. Write pointer faqat saqlanadigan element joylashtirilganda oldinga suriladi.

## String manipulyatsiya

**Anagram tekshirish:** ikki string bir xil harflardan tuzilganmi. `O(n)` time, `O(1)` space (alifbo cheklangan).

```js
function isAnagram(a, b) {
  if (a.length !== b.length) return false;
  const count = {};
  for (const ch of a) count[ch] = (count[ch] || 0) + 1;
  for (const ch of b) {
    if (!count[ch]) return false;
    count[ch]--;
  }
  return true;
}
```

**Substring qidirish (oddiy):** `O(n·m)` worst case (n — matn, m — pattern).

```js
function indexOfSubstring(text, pattern) {
  for (let i = 0; i + pattern.length <= text.length; i++) {
    let j = 0;
    while (j < pattern.length && text[i + j] === pattern[j]) j++;
    if (j === pattern.length) return i;
  }
  return -1;
}
```

**💡 Tushuncha:** Anagram, palindrom, character-count muammolarida tez-tez **frequency map** (yoki 26 elementli array agar faqat lotin harflari bo'lsa) ishlatiladi. Bu `O(n)` time va konstanta space beradi.

## Array vs Linked List trade-off

| Amal | Array | Linked List |
|------|-------|-------------|
| Index orqali kirish | `O(1)` | `O(n)` |
| Qidirish (search) | `O(n)` | `O(n)` |
| Boshiga qo'shish/o'chirish | `O(n)` | `O(1)` |
| Oxiriga qo'shish | `O(1)`* amortized | `O(1)` (tail bilan) |
| O'rtaga qo'shish (pozitsiya ma'lum) | `O(n)` | `O(1)`** |
| Xotira | Ketma-ket, kam overhead | Har tugun uchun pointer overhead |
| Cache-friendly | Ha (lokalitet) | Yo'q |

\* resize bo'lmasa; \*\* tugunga reference bo'lsa.

**💡 Tushuncha:** Array — tez tasodifiy kirish (`O(1)` index) va cache lokalitet kerak bo'lganda. Linked list — boshiga/o'rtaga tez-tez qo'shish/o'chirish kerak bo'lganda (pozitsiya ma'lum bo'lsa). Amalda JS'da array ko'p hollarda yetarli va cache lokalitet tufayli tezroq ishlaydi.

**⚠️ Ehtiyot bo'l:** "Linked list boshiga qo'shish `O(1)`" to'g'ri, lekin **n-elementni topib** keyin qo'shish baribir `O(n)` (topish uchun). Array index'i ma'lum bo'lsa kirish `O(1)`, lekin qo'shish/o'chirishda surish `O(n)`.

---

## Savol-javoblar

### ❓ Nega `arr[i]` ga kirish O(1)?

**✅ Javob:** Array elementlari xotirada ketma-ket (contiguous) va bir xil hajmda joylashgani uchun `i`-element manzili `boshlanish + i·element_hajmi` formulasi bilan bevosita hisoblanadi. Hech narsa qidirilmaydi, faqat matematik adres hisobi — shuning uchun doimiy vaqt.

### ❓ `push` O(1) bo'lsa, nega "amortized" deyiladi?

**✅ Javob:** Dynamic array bufer to'lganida ikki barobar kattalashtirilib eski elementlar ko'chiriladi — bu alohida `O(n)`. Lekin bu kamdan-kam bo'lgani uchun `n` ta push'ning jami narxi `O(n)`, demak bitta push **o'rtacha (amortized)** `O(1)`. Alohida push esa ba'zan `O(n)` bo'lishi mumkin.

### ❓ Nega `shift()` va `unshift()` O(n)?

**✅ Javob:** Ular massivning **boshida** ishlaydi. Boshidan element olinsa yoki qo'shilsa, qolgan barcha elementlarning indekslari o'zgaradi — ya'ni hammasi bir pozitsiyaga suriladi. Bu `n` ta ko'chirish, demak `O(n)`. Tsikl ichida `shift` ishlatish `O(n²)` ga olib keladi.

### ❓ JS'da string nega immutable, bu nima muammo tug'diradi?

**✅ Javob:** JS string'lari o'zgarmas — bitta belgini o'zgartirib bo'lmaydi, har "o'zgartirish" yangi string yaratadi. Muammo: tsikl ichida `result += char` har safar yangi string yaratgani uchun `O(n²)`. Yechim — belgilarni array'ga yig'ib, oxirida `join("")` qilish (`O(n)`).

### ❓ Two pointers patterni qachon ishlatiladi?

**✅ Javob:** Ko'pincha **saralangan** massiv/string'da juftliklar, palindrom, reverse, yoki yig'indi qidirishda. Ikki pointer (chetdan markazga yoki slow/fast) bir vaqtning o'zida harakatlanib, `O(n²)` brute force'ni `O(n)` ga tushiradi. Saralangan bo'lsa, yig'indi kichik/kattaligiga qarab qaysi pointerni surishni aniq bilamiz.

### ❓ Two Sum'da hash map va two pointers orasidagi farq?

**✅ Javob:** **Saralanmagan** massivda hash map ishlatamiz: har elementni ko'rib, `target - x` ni map'da qidiramiz — `O(n)` time, `O(n)` space. **Saralangan** massivda two pointers ishlatamiz: `O(n)` time, `O(1)` space. Saralangan bo'lsa two pointers afzal (xotira tejaydi); aks holda hash map (sortlash `O(n log n)` qo'shadi).

### ❓ Fixed va variable sliding window orasidagi farq?

**✅ Javob:** **Fixed** — oyna o'lchami doimiy (`k`); har qadamda bitta element kirib bittasi chiqadi (`O(1)` yangilanish). **Variable** — oyna shartga qarab kengayadi/qisqaradi (masalan "takrorlanmaydigan eng uzun substring"); o'ng pointer doim oldinga, chap pointer faqat shart buzilganda suriladi. Ikkalasi ham `O(n)`.

### ❓ Sliding window nega O(n), nested loop emas?

**✅ Javob:** `while` ichida bo'lsa-da, **chap pointer hech qachon orqaga qaytmaydi** — u faqat oldinga suriladi. Har element o'ng pointer bilan bir marta, chap pointer bilan ko'pi bilan bir marta ko'riladi → jami `O(2n) = O(n)`. Bu amortized tahlil.

### ❓ Prefix sum nima va qachon foydali?

**✅ Javob:** Oldindan yig'indilar massivini tuzib, har qanday `[i..j]` oraliq yig'indisini `prefix[j+1] - prefix[i]` orqali `O(1)` da olamiz. Qurish `O(n)`, har so'rov `O(1)`. **Ko'p marta** oraliq so'rovlari bo'lganda yoki "subarray sum = k" tipidagi muammolarda (hash map bilan) juda foydali.

### ❓ In-place algoritm nima va nega muhim?

**✅ Javob:** Input'ni qo'shimcha massiv yaratmasdan **joyida** o'zgartiradi — `O(1)` qo'shimcha xotira. Muhim, chunki katta ma'lumotlarda xotirani tejaydi. Misol: move zeroes, remove duplicates, reverse. Ko'pincha read/write pointer texnikasi bilan amalga oshiriladi.

### ❓ Anagram tekshirishning eng samarali usuli?

**✅ Javob:** Frequency map (yoki 26 elementli array agar faqat lotin harflari): birinchi string harflarini sanaymiz, ikkinchisida kamaytiramiz — agar manfiyga tushsa yoki uzunliklar farq qilsa, anagram emas. `O(n)` time, `O(1)` space (alifbo cheklangan). Sortlab taqqoslash ham mumkin (`O(n log n)`), lekin frequency map tezroq.

### ❓ String'ni belgi-belgi qurishda nima muammo?

**✅ Javob:** Immutability tufayli `result += char` har safar yangi string yaratadi — `O(n)` ko'chirish, tsiklda jami `O(n²)`. Yechim: belgilarni array'ga `push` qilib (`O(1)` amortized), oxirida `join("")` (`O(n)`). Umumiy `O(n)`.

### ❓ Array va linked list orasidagi asosiy trade-off?

**✅ Javob:** Array — `O(1)` index access va cache lokalitet, lekin boshiga/o'rtaga qo'shish `O(n)`. Linked list — boshiga qo'shish/o'chirish `O(1)`, lekin index access `O(n)` va pointer overhead. Tasodifiy kirish ko'p bo'lsa array; tez-tez boshiga/o'rtaga (pozitsiya ma'lum) qo'shish bo'lsa linked list.

### ❓ `slice` va `splice` orasidagi farq nima (complexity bilan)?

**✅ Javob:** `slice(i, j)` — **yangi** array qaytaradi (nusxa), originalni o'zgartirmaydi, `O(n)`. `splice(i, d, ...items)` — originalni **joyida** o'zgartiradi (o'chiradi/qo'shadi), o'chirilganlarni qaytaradi, `O(n)` (surish kerak). Nomi o'xshash, lekin biri immutable nusxa, biri mutatsiya.

---

## Masalalar

> Yechimlar: [`solutions/dsa/02-arrays-strings.md`](../solutions/dsa/02-arrays-strings.md)

1. **Oson.** Two Sum (saralanmagan massiv): yig'indisi `target` bo'lgan ikki indeksni hash map bilan `O(n)` da toping.
2. **Oson.** Valid Palindrome: faqat harf/raqamlarni hisobga olib, registr farqsiz palindromni two pointers bilan tekshiring.
3. **Oson.** Valid Anagram: ikki string anagram ekanini frequency map bilan aniqlang.
4. **Oson.** Reverse String: string'ni (array sifatida) in-place teskari aylantiring (`O(1)` space).
5. **O'rta.** Maximum Average Subarray: hajmi `k` bo'lgan subarray'ning eng katta o'rtacha yig'indisini fixed sliding window bilan toping.
6. **O'rta.** Longest Substring Without Repeating Characters: takrorlanmaydigan eng uzun substring uzunligini variable sliding window bilan toping.
7. **O'rta.** Move Zeroes: barcha nollarni massiv oxiriga, qolgan tartibni saqlab, in-place suring.
8. **O'rta.** Product of Array Except Self: har indeks uchun o'zidan boshqa hamma elementlar ko'paytmasini bo'lishsiz (`O(n)`) toping (prefix/suffix).
9. **O'rta.** Subarray Sum Equals K: yig'indisi `k` ga teng subarray'lar sonini prefix sum + hash map bilan `O(n)` da sanang.
10. **O'rta.** Group Anagrams: string'lar massivini anagram guruhlariga ajrating.
11. **Qiyin.** Minimum Window Substring: `t` ning barcha harflarini o'z ichiga olgan `s` ning eng qisqa substring'ini sliding window bilan toping.
12. **Qiyin.** Trapping Rain Water: balandliklar massivida yomg'irdan keyin tutilgan suv hajmini two pointers bilan `O(n)` time, `O(1)` space da hisoblang.

← [DSA bo'limiga qaytish](./README.md)
