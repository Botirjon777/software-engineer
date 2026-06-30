# Complexity va Big-O

Algoritm qanchalik "yaxshi" ekanini sekundlar bilan emas, balki **input hajmi o'sganda qancha resurs talab qilishini** o'lchaymiz. Bu bo'lim Big-O notatsiyasi, time va space complexity, complexity sinflari va kodga qarab complexity'ni qanday hisoblashni o'rgatadi. Intervyuning deyarli har bir savolida sendan "buning complexity'si qancha?" deb so'rashadi — shu sababli bu mavzu fundament.

## Mundarija

- [Nega complexity tahlili kerak](#nega-complexity-tahlili-kerak)
- [Time vs Space complexity](#time-vs-space-complexity)
- [Asimptotik notatsiya: O, Ω, Θ](#asimptotik-notatsiya-o-Ω-Θ)
- [Best / Average / Worst case](#best--average--worst-case)
- [Complexity sinflari](#complexity-sinflari)
- [Complexity'ni qanday hisoblash](#complexityni-qanday-hisoblash)
- [Recursion va recurrence](#recursion-va-recurrence)
- [Amortized complexity](#amortized-complexity)
- [Space complexity va call stack](#space-complexity-va-call-stack)
- [Amaliy misollar](#amaliy-misollar)
- [Taqqoslash jadvali](#taqqoslash-jadvali)
- [Savol-javoblar](#savol-javoblar)
- [Masalalar](#masalalar)

---

## Nega complexity tahlili kerak

Bir algoritmning ikki implementatsiyasini taqqoslash uchun ularni bitta kompyuterda ishga tushirib vaqtni o'lchasak bo'lardi. Lekin bu ishonchsiz: vaqt protsessor, til, kompilyator, hatto boshqa ishlayotgan dasturlarga bog'liq. Bizga **mashinaga bog'liq bo'lmagan** o'lchov kerak.

**💡 Tushuncha:** Big-O — bu input hajmi `n` cheksizlikka intilganda algoritm bajaradigan **amallar sonining o'sish tezligi**. U aniq vaqtni emas, balki "n ikki barobar oshsa, ish necha barobar oshadi?" degan savolga javob beradi.

Masalan:

```js
// A: har bir elementni bir marta ko'radi
function sumA(arr) {
  let total = 0;
  for (const x of arr) total += x; // n ta amal
  return total;
}

// B: har juftlikni ko'radi
function sumPairsB(arr) {
  let total = 0;
  for (let i = 0; i < arr.length; i++)
    for (let j = 0; j < arr.length; j++)
      total += arr[i] * arr[j]; // n*n amal
  return total;
}
```

`n = 1000` bo'lsa, A ~1000 amal, B ~1,000,000 amal bajaradi. Mashina qanchalik tez bo'lishidan qat'i nazar, B kattalashganda yomonlashadi. Big-O aynan shu farqni ushlaydi: A = `O(n)`, B = `O(n²)`.

## Time vs Space complexity

- **Time complexity** — algoritm bajaradigan amallar soni `n` ga qarab qanday o'sadi.
- **Space complexity** — algoritm input'dan tashqari qancha **qo'shimcha xotira** ishlatadi (auxiliary space). Bunga call stack ham kiradi.

**⚠️ Ehtiyot bo'l:** Space complexity'da odatda input'ning o'zini hisobga olmaymiz (u baribir kerak), faqat **qo'shimcha** xotirani hisoblaymiz. `O(1)` space — input hajmiga bog'liq bo'lmagan doimiy xotira.

```js
// Time O(n), Space O(1) — faqat bir nechta o'zgaruvchi
function max(arr) {
  let m = -Infinity;
  for (const x of arr) if (x > m) m = x;
  return m;
}

// Time O(n), Space O(n) — yangi array yaratadi
function doubled(arr) {
  return arr.map((x) => x * 2); // n ta yangi element
}
```

Ko'pincha **time-space trade-off** bo'ladi: tezlikni oshirish uchun ko'proq xotira sarflaymiz (masalan hash map orqali kesh).

## Asimptotik notatsiya: O, Ω, Θ

Asimptotik notatsiya funksiyalarning o'sish tezligini chegaralaydi:

- **Big-O (O)** — **yuqori chegara** ("dan ko'p bo'lmaydi"). `f(n) = O(g(n))` degani: yetarlicha katta `n` uchun `f(n) ≤ c·g(n)`. Bu worst case'ni ifodalaydi.
- **Big-Omega (Ω)** — **quyi chegara** ("dan kam bo'lmaydi"). `f(n) = Ω(g(n))`: `f(n) ≥ c·g(n)`. Best case'ni ifodalaydi.
- **Big-Theta (Θ)** — **aniq chegara** (yuqori ham, quyi ham bir xil). `f(n) = Θ(g(n))`: `c₁·g(n) ≤ f(n) ≤ c₂·g(n)`.

**💡 Tushuncha:** Amalda intervyuda deyarli har doim **Big-O** ishlatiladi, chunki bizni eng yomon holatdagi kafolat qiziqtiradi. Lekin "linear search Θ(n) emas, O(n)" deyish to'g'riroq: linear search best case'da Ω(1) (birinchi elementda topsa), shuning uchun Θ(n) emas. O(n) esa to'g'ri (yuqori chegara).

Formal misol: `f(n) = 3n² + 5n + 2`.
- `f(n) = O(n²)` ✅ (yuqori chegara)
- `f(n) = Ω(n²)` ✅ (quyi chegara)
- `f(n) = Θ(n²)` ✅ (ikkalasi ham)
- `f(n) = O(n³)` ✅ ham to'g'ri (sust, lekin haqiqat) — O faqat "dan oshmaydi" deydi.

## Best / Average / Worst case

Bitta algoritmning complexity'si input'ning **ko'rinishiga** qarab farq qilishi mumkin:

```js
function linearSearch(arr, target) {
  for (let i = 0; i < arr.length; i++) {
    if (arr[i] === target) return i;
  }
  return -1;
}
```

- **Best case** — `Ω(1)`: target birinchi pozitsiyada.
- **Average case** — `Θ(n)`: o'rtacha `n/2` ta tekshiruv.
- **Worst case** — `O(n)`: target oxirida yoki yo'q.

**💡 Tushuncha:** Intervyuda "complexity" deganda odatda **worst case (Big-O)** nazarda tutiladi. Lekin Quicksort kabi algoritmlarda average (`O(n log n)`) va worst (`O(n²)`) farqini bilish muhim.

## Complexity sinflari

Eng sekindan tezgacha tartib: `O(1) < O(log n) < O(n) < O(n log n) < O(n²) < O(2ⁿ) < O(n!)`.

### O(1) — Constant
Input hajmiga bog'liq emas. Doim bir xil miqdorda ish.

```js
function first(arr) {
  return arr[0]; // index orqali — O(1)
}
```

### O(log n) — Logarithmic
Har qadamda muammo **yarmiga** qisqaradi. Binary search klassik misol.

```js
// Saralangan arrayda binary search — O(log n)
function binarySearch(arr, target) {
  let lo = 0, hi = arr.length - 1;
  while (lo <= hi) {
    const mid = (lo + hi) >> 1;
    if (arr[mid] === target) return mid;
    if (arr[mid] < target) lo = mid + 1;
    else hi = mid - 1;
  }
  return -1;
}
```

`n = 1,000,000` bo'lsa ham faqat ~20 ta qadam (log₂ 10⁶ ≈ 20).

### O(n) — Linear
Har elementni bir marta ko'radi.

```js
function contains(arr, x) {
  for (const v of arr) if (v === x) return true; // O(n)
  return false;
}
```

### O(n log n) — Linearithmic
Samarali sortlash algoritmlari (Merge sort, Heap sort, Timsort). "n ta elementni log n marta" tipidagi ish.

```js
function sortCopy(arr) {
  return [...arr].sort((a, b) => a - b); // O(n log n)
}
```

### O(n²) — Quadratic
Ko'pincha **nested loop** (ichma-ich tsikl) bilan. Har juftlikni ko'radi.

```js
function hasDuplicateNaive(arr) {
  for (let i = 0; i < arr.length; i++)
    for (let j = i + 1; j < arr.length; j++)
      if (arr[i] === arr[j]) return true; // O(n²)
  return false;
}
```

### O(2ⁿ) — Exponential
Har qadamda variantlar **ikki barobar**. Naive recursive Fibonacci, subset'larni generatsiya qilish.

```js
function fib(n) {
  if (n < 2) return n;
  return fib(n - 1) + fib(n - 2); // O(2ⁿ)
}
```

### O(n!) — Factorial
Barcha permutatsiyalar. Travelling salesman brute force.

```js
function permutations(arr) {
  if (arr.length <= 1) return [arr];
  const result = [];
  for (let i = 0; i < arr.length; i++) {
    const rest = [...arr.slice(0, i), ...arr.slice(i + 1)];
    for (const p of permutations(rest)) result.push([arr[i], ...p]);
  }
  return result; // n! ta natija
}
```

**⚠️ Ehtiyot bo'l:** `O(2ⁿ)` va `O(n!)` faqat juda kichik `n` uchun amaliy (n ≤ 20 atrofida). Intervyuda exponential yechim bersang, "buni DP/memoization bilan optimallashtirsa bo'ladimi?" degan savolga tayyor bo'l.

## Complexity'ni qanday hisoblash

Bir necha oddiy qoida:

**1. Ketma-ket bloklar — qo'shiladi, lekin dominant qoladi.**
```js
function f(arr) {
  for (const x of arr) console.log(x);        // O(n)
  for (const x of arr)                          // O(n²)
    for (const y of arr) console.log(x, y);
}
// O(n) + O(n²) = O(n²) — dominant term qoladi
```

**2. Nested loop — ko'paytiriladi.**
```js
for (let i = 0; i < n; i++)      // n marta
  for (let j = 0; j < m; j++)    // m marta
    work();                      // O(n * m)
```

**3. Konstantalar va sust terminlar tashlanadi.**
- `O(2n)` → `O(n)`
- `O(3n² + 100n + 5)` → `O(n²)`
- `O(n/2)` → `O(n)`

**💡 Tushuncha:** Big-O'da **dominant term** (eng tez o'sadigan) qoladi va **konstanta koeffitsientlar tashlanadi**. Sababi: `n → ∞` da `n²` har qanday `n` ga nisbatan cheksiz katta bo'ladi, koeffitsient esa o'sish *tezligini* o'zgartirmaydi.

**4. Yarimlanuvchi tsikl — O(log n).**
```js
for (let i = n; i > 0; i = Math.floor(i / 2)) work(); // O(log n)
```

**5. Ikki har xil input — alohida o'zgaruvchi.**
```js
function g(a, b) {
  for (const x of a) work(); // O(a)
  for (const y of b) work(); // O(b)
} // O(a + b) — "O(n)" deb soddalashtirma!
```

## Recursion va recurrence

Recursive funksiya complexity'si **recurrence relation** (rekurrent munosabat) orqali hisoblanadi: `T(n) = (necha marta chaqiriladi) · T(qisqargan hajm) + (har chaqiruvdagi ish)`.

```js
// Merge sort: massivni 2 ga bo'lib, har birini sortlab, birlashtiradi
// T(n) = 2·T(n/2) + O(n)  →  O(n log n)
```

Tez-tez uchraydigan recurrence'lar:

| Recurrence | Natija | Misol |
|------------|--------|-------|
| `T(n) = T(n/2) + O(1)` | `O(log n)` | Binary search |
| `T(n) = 2·T(n/2) + O(n)` | `O(n log n)` | Merge sort |
| `T(n) = T(n-1) + O(1)` | `O(n)` | Linear recursion |
| `T(n) = T(n-1) + O(n)` | `O(n²)` | Selection sort recursive |
| `T(n) = 2·T(n-1) + O(1)` | `O(2ⁿ)` | Naive Fibonacci |

**💡 Tushuncha:** Tez baholash uchun **recursion tree** chizish foydali: har darajada qancha ish bajariladi va necha daraja bor — shularni ko'paytir. Master Theorem `T(n) = a·T(n/b) + O(nᵈ)` shaklidagi recurrence'larni tez yechadi.

## Amortized complexity

Ba'zan bitta amal ba'zan qimmat, ba'zan arzon bo'ladi. **Amortized** complexity — ko'p amalning **o'rtacha** narxi.

Klassik misol — **dynamic array push** (JS `Array.push`):

```js
const arr = [];
for (let i = 0; i < n; i++) arr.push(i); // har push O(1) amortized
```

Array to'lganida ichki bufer **ikki barobar** kattalashtiriladi va eski elementlar ko'chiriladi (bu `O(n)`). Lekin bu kamdan-kam bo'ladi. `n` ta push'ni jami ko'chirish narxi `n + n/2 + n/4 + ... ≈ 2n = O(n)`, demak **bitta push o'rtacha `O(1)`** — buni amortized O(1) deymiz.

**⚠️ Ehtiyot bo'l:** Amortized O(1) — bu "har doim O(1)" degani EMAS. Bitta alohida push `O(n)` bo'lishi mumkin (resize paytida). Lekin ketma-ket ko'p amalning o'rtachasi O(1). Real-time tizimlarda bu farq muhim bo'lishi mumkin.

## Space complexity va call stack

Space complexity faqat aniq yaratilgan struktura emas — **recursion stack** ham xotira oladi.

```js
// Time O(n), Space O(n) — n ta freym stack'da
function sumRec(arr, i = 0) {
  if (i === arr.length) return 0;
  return arr[i] + sumRec(arr, i + 1); // har chaqiruv stack'da qoladi
}

// Time O(n), Space O(1) — tail/iterative, stack o'smaydi
function sumIter(arr) {
  let s = 0;
  for (const x of arr) s += x;
  return s;
}
```

**💡 Tushuncha:** Recursion chuqurligi `d` bo'lsa, call stack `O(d)` xotira oladi. Merge sort space'i `O(n)` (vaqtinchalik massivlar), lekin recursion chuqurligi `O(log n)`. Binary search recursive variantida space `O(log n)`, iterativda `O(1)`.

## Amaliy misollar

Quyidagi kodlarning complexity'sini ayt (javoblar pastda):

```js
// (1)
function a(n) {
  for (let i = 0; i < n; i++)
    for (let j = i; j < n; j++) work();
}

// (2)
function b(n) {
  for (let i = 1; i < n; i *= 2) work();
}

// (3)
function c(matrix) { // matrix n x m
  for (const row of matrix)
    for (const cell of row) work();
}

// (4)
function d(n) {
  for (let i = 0; i < n; i++)
    for (let j = 0; j < 100; j++) work();
}
```

**✅ Javoblar:**
- (1) `O(n²)` — ichki tsikl `n + (n-1) + ... + 1 = n(n+1)/2` marta ishlaydi, bu `O(n²)`.
- (2) `O(log n)` — `i` har qadamda ikki barobar oshadi.
- (3) `O(n·m)` — barcha kataklar (jami element soni). Kvadrat matritsa bo'lsa `O(n²)`.
- (4) `O(n)` — ichki tsikl konstanta (100) marta, demak `O(100n) = O(n)`.

## Taqqoslash jadvali

`n = 1000` uchun amallar soni va o'sish xarakteri:

| Big-O | Nomi | n=10 | n=1000 | n=1M | Tipik misol |
|-------|------|------|--------|------|-------------|
| `O(1)` | Constant | 1 | 1 | 1 | Array index, hash lookup |
| `O(log n)` | Logarithmic | ~3 | ~10 | ~20 | Binary search |
| `O(n)` | Linear | 10 | 1000 | 10⁶ | Array bo'ylab tsikl |
| `O(n log n)` | Linearithmic | ~33 | ~10⁴ | ~2·10⁷ | Merge/Quick sort |
| `O(n²)` | Quadratic | 100 | 10⁶ | 10¹² | Nested loop, bubble sort |
| `O(2ⁿ)` | Exponential | 1024 | 10³⁰¹ | ∞ | Naive Fibonacci, subsets |
| `O(n!)` | Factorial | 3.6M | ∞ | ∞ | Permutations, TSP brute force |

**💡 Tushuncha:** Intervyuda input chegarasiga qarab kerakli complexity'ni taxmin qilish mumkin: `n ≤ 10⁸` → `O(n)`/`O(n log n)`; `n ≤ 10⁴` → `O(n²)` ham o'tadi; `n ≤ 20` → `O(2ⁿ)`/`O(n!)` ehtimoli bor.

---

## Savol-javoblar

### ❓ Big-O nima va nima uchun aniq vaqt o'rniga ishlatiladi?

**✅ Javob:** Big-O — input hajmi `n` o'sganda algoritm amallar sonining **o'sish tezligini** ifodalovchi asimptotik yuqori chegara. Aniq vaqt mashina, til va tashqi omillarga bog'liq bo'lgani uchun ishonchsiz; Big-O esa mashinaga bog'liq emas va algoritmlarni adolatli taqqoslash imkonini beradi.

### ❓ O, Ω va Θ orasidagi farq nima?

**✅ Javob:** `O` — yuqori chegara (dan ko'p bo'lmaydi, worst case), `Ω` — quyi chegara (dan kam bo'lmaydi, best case), `Θ` — aniq chegara (yuqori va quyi bir xil). Amalda Big-O eng ko'p ishlatiladi, chunki bizni eng yomon holatdagi kafolat qiziqtiradi.

### ❓ Nega Big-O'da konstantalar va sust terminlar tashlanadi?

**✅ Javob:** Big-O o'sish tezligini ifodalaydi, aniq amallar sonini emas. `n → ∞` da dominant term (`n²`) boshqalarini (`n`, konstanta) ustun keladi, koeffitsient esa o'sish *tezligini* o'zgartirmaydi. Shu sababli `3n² + 5n + 2 → O(n²)`.

### ❓ Time va space complexity orasidagi trade-off nima?

**✅ Javob:** Ko'pincha algoritmni tezlashtirish uchun qo'shimcha xotira sarflaymiz (masalan, takroriy hisoblashlarni kesh/hash map'da saqlash) yoki aksincha xotirani tejash uchun ko'proq hisob-kitob qilamiz. Klassik misol — memoization: time'ni `O(2ⁿ)` dan `O(n)` ga tushiradi, lekin `O(n)` space sarflaydi.

### ❓ Worst, average va best case orasidagi farq nima? Qaysi biri muhimroq?

**✅ Javob:** Best — eng qulay input (Ω), average — odatdagi/o'rtacha input (Θ), worst — eng yomon input (O). Intervyuda va amalda odatda **worst case** muhim, chunki u kafolatlangan yuqori chegarani beradi. Ammo Quicksort kabi algoritmlarda average (`O(n log n)`) va worst (`O(n²)`) farqini bilish zarur.

### ❓ Nested loop har doim O(n²) bo'ladimi?

**✅ Javob:** Yo'q. Faqat ikkala tsikl ham `n` ga proporsional bo'lsa `O(n²)`. Agar ichki tsikl konstanta marta ishlasa — `O(n)`; ikki har xil input bo'lsa — `O(n·m)`; ichki tsikl yarimlanib borsa — `O(n log n)` bo'lishi mumkin. Tsikllarning chegaralariga qarab hisoblash kerak.

### ❓ Amortized complexity nima? Dynamic array push misolida tushuntir.

**✅ Javob:** Amortized — ketma-ket ko'p amalning **o'rtacha** narxi. JS `Array.push` odatda `O(1)`, lekin ichki bufer to'lganida u ikki barobar kattalashtirilib elementlar ko'chiriladi (`O(n)`). Bu kamdan-kam bo'lgani uchun `n` ta push'ning jami narxi `O(n)`, demak bitta push **amortized O(1)**. Bu "har doim O(1)" degani emas — alohida push `O(n)` bo'lishi mumkin.

### ❓ Recursive funksiya complexity'sini qanday hisoblaysan?

**✅ Javob:** Recurrence relation tuzaman: `T(n) = (chaqiruvlar soni)·T(qisqargan hajm) + (har chaqiruvdagi ish)`. Masalan merge sort: `T(n) = 2T(n/2) + O(n) → O(n log n)`. Tez baholash uchun recursion tree chizib, har darajadagi ishni daraja soniga ko'paytiraman, yoki Master Theorem'dan foydalanaman.

### ❓ Space complexity'da call stack qanday rol o'ynaydi?

**✅ Javob:** Har bir recursion chaqiruvi call stack'da freym egallaydi. Recursion chuqurligi `d` bo'lsa, space `O(d)`. Masalan, `O(n)` chuqurlikdagi linear recursion `O(n)` space oladi — hatto qo'shimcha struktura yaratmasa ham. Iterativ variantga aylantirsak ko'pincha space'ni `O(1)` ga tushirish mumkin.

### ❓ O(log n) qaerdan kelib chiqadi?

**✅ Javob:** Har qadamda muammo hajmi **konstanta marta** (odatda yarmiga) qisqaradi. `n` ni 1 ga yetkazguncha necha marta 2 ga bo'lish kerak — bu `log₂ n`. Shuning uchun binary search, balanced BST qidiruvi, yarimlanuvchi tsikllar `O(log n)`.

### ❓ `O(n + m)` ni `O(n)` deb soddalashtirsa bo'ladimi?

**✅ Javob:** Yo'q, agar `n` va `m` mustaqil inputlar bo'lsa. `O(n + m)` to'g'ri ifoda — masalan ikki alohida massiv bo'ylab tsikl. Ularni bitta `n` deb soddalashtirish noto'g'ri, chunki biri ikkinchisidan ancha katta bo'lishi mumkin. Faqat `m`-`n` bilan bog'liq (masalan `m ≤ n`) bo'lsa soddalashtirish mumkin.

### ❓ Input chegarasiga qarab kerakli complexity'ni qanday taxmin qilasan?

**✅ Javob:** Taxminan: `n ≤ 20` → `O(2ⁿ)`/`O(n!)` o'tadi; `n ≤ 500` → `O(n³)`; `n ≤ 5000` → `O(n²)`; `n ≤ 10⁶` → `O(n)`/`O(n log n)`; juda katta → `O(log n)`/`O(1)`. Bu intervyuda kerakli yondashuvni tanlashga yordam beradi (taxminan 10⁸ amal/sekund deb hisoblanadi).

### ❓ `O(2ⁿ)` algoritmni qanday optimallashtirish mumkin?

**✅ Javob:** Ko'pincha takroriy subproblem'lar bor — ularni **memoization** yoki **dynamic programming** bilan keshlab, exponential'ni polinomial (`O(n)` yoki `O(n²)`) ga tushirish mumkin. Naive Fibonacci `O(2ⁿ)`, memoized esa `O(n)`. Ba'zan pruning (backtracking'da keraksiz shoxlarni kesish) yoki yaxshiroq matematik formula yordam beradi.

### ❓ JS'da `arr.includes()` yoki `arr.indexOf()` complexity'si qancha?

**✅ Javob:** `O(n)` — massiv bo'ylab chiziqli qidiruv. Agar tez-tez "bor/yo'q" tekshiruvi kerak bo'lsa, `Set` (`O(1)` average lookup) yoki `Map` ishlatish kerak. Bu intervyuda tez-tez ishlatiladigan optimizatsiya: `O(n²)` yechimni `Set` orqali `O(n)` ga tushirish.

---

## Masalalar

> Yechimlar: [`solutions/dsa/01-complexity-big-o.md`](../solutions/dsa/01-complexity-big-o.md)

1. **Oson.** Berilgan funksiyalar ro'yxatidagi har biri uchun time complexity'ni aniqlang (kod intervyuerdan beriladi): single loop, nested loop, halving loop.
2. **Oson.** `Set` yordamida massivda dublikat bor-yo'qligini `O(n)` time'da aniqlang va naive `O(n²)` yechim bilan taqqoslang.
3. **Oson.** Berilgan kodning space complexity'sini aniqlang: (a) `arr.map`, (b) in-place reverse, (c) recursive sum.
4. **O'rta.** Massivdagi **eng katta ikkita** elementni faqat bitta o'tishda (`O(n)` time, `O(1)` space) toping.
5. **O'rta.** Naive Fibonacci (`O(2ⁿ)`) ni memoization bilan `O(n)` ga optimallashtiring va ikkala variant uchun time/space complexity'ni yozing.
6. **O'rta.** Berilgan recurrence `T(n) = 2T(n/2) + O(n)` ni recursion tree orqali yeching va natijani asoslang (yozma tahlil + tekshiruvchi kod).
7. **O'rta.** `n` ta `push` amalining amortized O(1) ekanini empirik tasdiqlovchi o'lchov yozing (push'lar soni va ko'chirishlar sonini sanang).
8. **O'rta.** Saralangan massivda binary search yozing va uning `O(log n)` ekanini qadamlar sonini sanab ko'rsating.
9. **Qiyin.** Berilgan ikki vloyenniy (nested) tsiklli kod uchun aniq amallar sonini formula bilan (`n(n+1)/2`) hisoblang va Big-O ga keltiring.
10. **Qiyin.** Bir massivda ketma-ket `O(n)` + `O(n log n)` + `O(n²)` bloklari bor funksiyaning umumiy complexity'sini aniqlang va nega dominant term qolishini tushuntiring.

← [DSA bo'limiga qaytish](./README.md)
