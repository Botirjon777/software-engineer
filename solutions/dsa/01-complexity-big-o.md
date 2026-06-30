# Complexity va Big-O — Yechimlar

Har masala uchun to'liq ishlaydigan kod, complexity tahlili va o'zbekcha izoh. Kod TypeScript/JavaScript'da.

---

## 1. Funksiyalar uchun time complexity aniqlash

```js
function singleLoop(n) {       // O(n)
  for (let i = 0; i < n; i++) {}
}

function nestedLoop(n) {       // O(n²)
  for (let i = 0; i < n; i++)
    for (let j = 0; j < n; j++) {}
}

function halvingLoop(n) {      // O(log n)
  for (let i = n; i > 0; i = Math.floor(i / 2)) {}
}
```

**Izoh:**
- `singleLoop` — tsikl `n` marta aylanadi, demak `O(n)`.
- `nestedLoop` — har tashqi qadam uchun ichki tsikl `n` marta → `n·n = O(n²)`.
- `halvingLoop` — `i` har qadamda 2 ga bo'linadi, `n` dan 1 gacha `log₂ n` qadam → `O(log n)`.

---

## 2. Set bilan dublikat aniqlash — O(n) vs O(n²)

```js
// O(n) time, O(n) space
function hasDuplicate(arr) {
  const seen = new Set();
  for (const x of arr) {
    if (seen.has(x)) return true; // Set lookup O(1) average
    seen.add(x);
  }
  return false;
}

// O(n²) time, O(1) space — naive
function hasDuplicateNaive(arr) {
  for (let i = 0; i < arr.length; i++)
    for (let j = i + 1; j < arr.length; j++)
      if (arr[i] === arr[j]) return true;
  return false;
}
```

**Complexity:** `hasDuplicate` — Time **O(n)**, Space **O(n)**. `hasDuplicateNaive` — Time **O(n²)**, Space **O(1)**.

**Izoh:** Bu klassik **time-space trade-off**. `Set` har elementni bir marta ko'rib, `O(1)` average lookup beradi, shu sababli umumiy `O(n)`. Naive variant qo'shimcha xotira ishlatmaydi, lekin har juftlikni tekshirgani uchun `O(n²)`. Katta massivlarda `Set` variant ancha tez.

---

## 3. Space complexity aniqlash

```js
// (a) map — yangi array yaratadi → Space O(n)
function doubleMap(arr) {
  return arr.map((x) => x * 2);
}

// (b) in-place reverse → Space O(1)
function reverseInPlace(arr) {
  let l = 0, r = arr.length - 1;
  while (l < r) {
    [arr[l], arr[r]] = [arr[r], arr[l]];
    l++; r--;
  }
  return arr;
}

// (c) recursive sum → Space O(n) (call stack)
function recSum(arr, i = 0) {
  if (i === arr.length) return 0;
  return arr[i] + recSum(arr, i + 1);
}
```

**Complexity:**
- (a) Space **O(n)** — `map` `n` ta yangi element yaratadi.
- (b) Space **O(1)** — faqat ikki indeks o'zgaruvchisi, original array joyida o'zgartiriladi.
- (c) Space **O(n)** — `n` ta recursion freymi call stack'da to'planadi, garchi yangi struktura yaratilmasa ham.

**Izoh:** (c) muhim tuzoq — ko'pchilik uni `O(1)` deb o'ylaydi, lekin call stack ham xotira. Iterativ variant `O(1)` space bo'lardi.

---

## 4. Eng katta ikkita element — O(n), O(1)

```js
function topTwo(arr) {
  let first = -Infinity, second = -Infinity;
  for (const x of arr) {
    if (x > first) {
      second = first;
      first = x;
    } else if (x > second) {
      second = x;
    }
  }
  return [first, second];
}
```

**Complexity:** Time **O(n)** (bitta o'tish), Space **O(1)**.

**Izoh:** Sortlash (`O(n log n)`) yoki ikki marta `Math.max` (`O(2n)`) o'rniga bitta o'tishda ikki o'zgaruvchini yangilab boramiz. `x > first` bo'lsa eski birinchini ikkinchiga suramiz. Bu top-K muammosining sodda holati.

---

## 5. Fibonacci: O(2ⁿ) → O(n) memoization

```js
// Naive — Time O(2ⁿ), Space O(n) (recursion chuqurligi)
function fibNaive(n) {
  if (n < 2) return n;
  return fibNaive(n - 1) + fibNaive(n - 2);
}

// Memoized — Time O(n), Space O(n)
function fibMemo(n, memo = new Map()) {
  if (n < 2) return n;
  if (memo.has(n)) return memo.get(n);
  const result = fibMemo(n - 1, memo) + fibMemo(n - 2, memo);
  memo.set(n, result);
  return result;
}

// Bonus: iterativ — Time O(n), Space O(1)
function fibIter(n) {
  let a = 0, b = 1;
  for (let i = 0; i < n; i++) [a, b] = [b, a + b];
  return a;
}
```

**Complexity:** Naive `O(2ⁿ)`; Memoized `O(n)` time / `O(n)` space; Iterativ `O(n)` time / `O(1)` space.

**Izoh:** Naive variant bir xil `fib(k)` ni qayta-qayta hisoblaydi (eksponensial portlash). Memoization har `fib(k)` ni bir marta hisoblab keshlaydi → `n` ta noyob qiymat → `O(n)`. Iterativ variant esa faqat oxirgi ikki qiymatni saqlab `O(1)` space'ga erishadi.

---

## 6. Recurrence T(n) = 2T(n/2) + O(n) — recursion tree

**Tahlil:** Recursion tree'ni chizamiz:
- 0-daraja: 1 ta tugun, ish `n`.
- 1-daraja: 2 ta tugun, har biri `n/2` → jami `n`.
- 2-daraja: 4 ta tugun, har biri `n/4` → jami `n`.
- ...
- `k`-daraja: `2ᵏ` ta tugun, har biri `n/2ᵏ` → jami `n`.

Har darajada ish `O(n)`. Daraja soni `log₂ n` (hajm 1 ga yetguncha). Demak jami: `n · log₂ n = O(n log n)`.

```js
// Merge sort — aynan shu recurrence: T(n) = 2T(n/2) + O(n) = O(n log n)
function mergeSort(arr) {
  if (arr.length <= 1) return arr;
  const mid = arr.length >> 1;
  const left = mergeSort(arr.slice(0, mid));   // T(n/2)
  const right = mergeSort(arr.slice(mid));     // T(n/2)
  return merge(left, right);                   // O(n)
}

function merge(a, b) {
  const out = [];
  let i = 0, j = 0;
  while (i < a.length && j < b.length)
    out.push(a[i] <= b[j] ? a[i++] : b[j++]);
  while (i < a.length) out.push(a[i++]);
  while (j < b.length) out.push(b[j++]);
  return out;
}
```

**Complexity:** Time **O(n log n)**, Space **O(n)**.

**Izoh:** Har daraja `O(n)` ish (merge), `log n` daraja → `O(n log n)`. Master Theorem bilan ham: `a=2, b=2, d=1`, `bᵈ = 2 = a` → `O(nᵈ log n) = O(n log n)`.

---

## 7. Amortized O(1) push'ni empirik tasdiqlash

```js
// Soddalashtirilgan dynamic array: capacity to'lganda ikki barobar oshadi
class DynamicArray {
  constructor() {
    this.data = new Array(1);
    this.size = 0;
    this.capacity = 1;
    this.copies = 0; // jami ko'chirishlar soni
  }
  push(value) {
    if (this.size === this.capacity) {
      this.capacity *= 2;
      const bigger = new Array(this.capacity);
      for (let i = 0; i < this.size; i++) {
        bigger[i] = this.data[i];
        this.copies++; // har ko'chirishni sanaymiz
      }
      this.data = bigger;
    }
    this.data[this.size++] = value;
  }
}

function measure(n) {
  const arr = new DynamicArray();
  for (let i = 0; i < n; i++) arr.push(i);
  return { pushes: n, copies: arr.copies, ratio: arr.copies / n };
}

console.log(measure(1000));   // { pushes: 1000, copies: ~1023, ratio: ~1.02 }
console.log(measure(100000)); // ratio ~1.0 — jami ko'chirish ≈ n
```

**Complexity:** `n` ta push jami **O(n)** → bitta push **amortized O(1)**.

**Izoh:** Ko'chirishlar `1 + 2 + 4 + ... + n/2 ≈ n` ga teng (geometrik qator). Demak `n` ta push uchun jami ko'chirish chiziqli, har push o'rtacha doimiy. `ratio` ~1-2 atrofida turg'unlashadi — bu amortized O(1) ni tasdiqlaydi.

---

## 8. Binary search va qadamlarni sanash

```js
function binarySearch(arr, target) {
  let lo = 0, hi = arr.length - 1, steps = 0;
  while (lo <= hi) {
    steps++;
    const mid = (lo + hi) >> 1;
    if (arr[mid] === target) return { index: mid, steps };
    if (arr[mid] < target) lo = mid + 1;
    else hi = mid - 1;
  }
  return { index: -1, steps };
}

const arr = Array.from({ length: 1_000_000 }, (_, i) => i);
console.log(binarySearch(arr, 999_999)); // steps ~20
```

**Complexity:** Time **O(log n)**, Space **O(1)**.

**Izoh:** Har qadamda qidiruv oralig'i yarmiga qisqaradi. 10⁶ element uchun `log₂(10⁶) ≈ 20` qadam — `steps` maydoni buni tasdiqlaydi. Linear search bo'lsa 10⁶ qadam kerak bo'lardi.

---

## 9. Nested loop uchun aniq amallar sonini hisoblash

```js
function countOps(n) {
  let ops = 0;
  for (let i = 0; i < n; i++)
    for (let j = i; j < n; j++) ops++; // ichki tsikl i dan boshlanadi
  return ops;
}

// Formula bilan tekshirish
function formula(n) {
  return (n * (n + 1)) / 2;
}

console.log(countOps(5), formula(5)); // 15 15
console.log(countOps(100), formula(100)); // 5050 5050
```

**Tahlil:** Ichki tsikl `i=0` da `n` marta, `i=1` da `n-1` marta, ..., `i=n-1` da 1 marta ishlaydi. Jami: `n + (n-1) + ... + 1 = n(n+1)/2`.

**Complexity:** `n(n+1)/2 = (n² + n)/2 → O(n²)`.

**Izoh:** Aniq amallar soni `n(n+1)/2` bo'lsa-da, Big-O'da dominant term (`n²`) qoladi va `1/2` koeffitsienti tashlanadi → `O(n²)`. Kod va formula bir xil natija beradi.

---

## 10. Ketma-ket bloklar: O(n) + O(n log n) + O(n²)

```js
function combined(arr) {
  const n = arr.length;

  // Blok 1: O(n)
  let sum = 0;
  for (const x of arr) sum += x;

  // Blok 2: O(n log n)
  const sorted = [...arr].sort((a, b) => a - b);

  // Blok 3: O(n²)
  let pairs = 0;
  for (let i = 0; i < n; i++)
    for (let j = i + 1; j < n; j++)
      if (arr[i] + arr[j] === 0) pairs++;

  return { sum, sorted, pairs };
}
```

**Complexity:** `O(n) + O(n log n) + O(n²) = O(n²)`. Space **O(n)** (`sorted` nusxasi).

**Izoh:** Ketma-ket bloklar **qo'shiladi**, lekin Big-O'da faqat **dominant term** qoladi. `n → ∞` da `n²` boshqalardan cheksiz tez o'sadi, shu sababli `O(n)` va `O(n log n)` "yutiladi" va umumiy complexity `O(n²)` bo'ladi. Agar 3-blokni optimallashtirsak (masalan hash map bilan `O(n)`), umumiy complexity `O(n log n)` ga tushadi.

---

← [DSA bo'limiga qaytish](../../dsa/README.md)
