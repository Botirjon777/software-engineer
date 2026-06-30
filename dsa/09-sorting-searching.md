# Sorting va Searching

Saralash (sorting) va qidirish (searching) — DSA'ning poydevori. Suhbatlarda ular ikki sababdan muhim: birinchidan, klassik algoritmlar (merge sort, quick sort, binary search) sizning algoritmik fikrlashingizni ko'rsatadi; ikkinchidan, real masalalarning yarmidan ko'pi "avval saralab ol, keyin qidir" yoki "javob ustida binary search" pattern'iga keladi. Bu hujjat har bir algoritmni qanday ishlashidan tortib, complexity, stability, in-place tushunchalari, `Array.sort()` ning ichki tuzog'i va binary search'ning ishonchli shablonlarigacha boradi.

## Mundarija

- [Sorting nima va nega muhim](#sorting-nima-va-nega-muhim)
- [Bubble Sort — O(n²)](#bubble-sort--on)
- [Selection Sort — O(n²)](#selection-sort--on)
- [Insertion Sort — O(n²)](#insertion-sort--on)
- [Merge Sort — O(n log n)](#merge-sort--on-log-n)
- [Quick Sort — O(n log n)](#quick-sort--on-log-n)
- [Heap Sort — O(n log n)](#heap-sort--on-log-n)
- [Stable vs Unstable sort](#stable-vs-unstable-sort)
- [In-place vs Out-of-place](#in-place-vs-out-of-place)
- [Merge Sort vs Quick Sort taqqoslash](#merge-sort-vs-quick-sort-taqqoslash)
- [Array.sort() ichki ishlashi va comparator tuzog'i](#arraysort-ichki-ishlashi-va-comparator-tuzogi)
- [Counting va Radix Sort (non-comparison)](#counting-va-radix-sort-non-comparison)
- [Binary Search — shablon va edge case'lar](#binary-search--shablon-va-edge-caselar)
- [Binary Search on Answer pattern](#binary-search-on-answer-pattern)
- [Search in Rotated Array](#search-in-rotated-array)
- [Intervyu Q&A](#intervyu-qa)
- [Masalalar](#masalalar)

---

## Sorting nima va nega muhim

**💡 Tushuncha:** Sorting — elementlarni ma'lum tartibda (odatda o'sish bo'yicha) joylashtirish. Bu shunchaki "chiroyli ko'rinish" emas: saralangan ma'lumotda binary search O(log n) da ishlaydi, dublikatlar yonma-yon turadi, ikkita massivni birlashtirish oson bo'ladi. Ko'p masala "avval saralang" bilan O(n²) dan O(n log n) ga tushadi.

Algoritmlarni ikki sinfga ajratamiz:

| Sinf | Algoritmlar | Vaqt (o'rtacha) |
|---|---|---|
| **Comparison-based** | bubble, selection, insertion, merge, quick, heap | eng yaxshisi O(n log n) |
| **Non-comparison** | counting, radix, bucket | O(n) yoki O(n+k) |

**⚠️ Ehtiyot bo'l:** Comparison-based saralashning **nazariy chegarasi O(n log n)** — undan tezroq qilib bo'lmaydi (har solishtirish 1 bit ma'lumot beradi, n! permutatsiyani ajratish uchun log(n!) ≈ n log n solishtirish kerak). O(n) ga faqat elementlar haqida qo'shimcha bilim (masalan, butun son va chegarali diapazon) bo'lsa erishiladi.

---

## Bubble Sort — O(n²)

**💡 Tushuncha:** Qo'shni juftlarni solishtirib, noto'g'ri tartibda bo'lsa almashtiramiz. Har "o'tish"da (pass) eng katta element "pufakcha" kabi oxiriga suriladi. `swapped` bayrog'i bilan optimallashtirilsa, allaqachon saralangan massivda O(n) bo'ladi.

```js
function bubbleSort(arr) {
  const a = [...arr];
  for (let i = 0; i < a.length - 1; i++) {
    let swapped = false;
    for (let j = 0; j < a.length - 1 - i; j++) {
      if (a[j] > a[j + 1]) {
        [a[j], a[j + 1]] = [a[j + 1], a[j]];   // swap
        swapped = true;
      }
    }
    if (!swapped) break;                        // allaqachon saralangan
  }
  return a;
}
```

- **Vaqt:** O(n²) o'rtacha/worst, O(n) eng yaxshi (saralangan).
- **Xotira:** O(1) (in-place).
- **Stable:** ha.

**⚠️ Ehtiyot bo'l:** Bubble sort amaliyotda deyarli ishlatilmaydi — u faqat o'qitish uchun. Suhbatda "bubble sort qo'llang" desa, optimallashtirilgan (`swapped` bayrog'li) versiyani bilganingizni ko'rsating.

---

## Selection Sort — O(n²)

**💡 Tushuncha:** Har qadamda qolgan saralanmagan qismdan **eng kichik** elementni topib, uni hozirgi pozitsiyaga olib qo'yamiz. Bubble'dan farqi — almashtirishlar (swap) soni minimal: ko'pi bilan n ta.

```js
function selectionSort(arr) {
  const a = [...arr];
  for (let i = 0; i < a.length - 1; i++) {
    let min = i;
    for (let j = i + 1; j < a.length; j++) {
      if (a[j] < a[min]) min = j;
    }
    if (min !== i) [a[i], a[min]] = [a[min], a[i]];
  }
  return a;
}
```

- **Vaqt:** O(n²) har doim (hatto saralangan massivda ham).
- **Xotira:** O(1).
- **Stable:** yo'q (swap teng elementlar tartibini buzishi mumkin).

**⚠️ Ehtiyot bo'l:** Selection sort har doim O(n²) — `swapped` optimallashtirishi yo'q, chunki minimumni topish uchun baribir butun qismni ko'rib chiqasiz. Lekin swap kam bo'lgani uchun yozish qimmat bo'lgan xotirada (masalan flash) foydali.

---

## Insertion Sort — O(n²)

**💡 Tushuncha:** Qarta o'yinidagi kabi: har bir yangi elementni allaqachon saralangan chap qismga **to'g'ri joyiga** kiritamiz. Kichik yoki deyarli saralangan massivlarda juda tez — shu sababli ko'p tilning ichki sort'i kichik bo'laklar uchun insertion sort'ga o'tadi.

```js
function insertionSort(arr) {
  const a = [...arr];
  for (let i = 1; i < a.length; i++) {
    const key = a[i];
    let j = i - 1;
    while (j >= 0 && a[j] > key) {
      a[j + 1] = a[j];                          // o'ngga surish
      j--;
    }
    a[j + 1] = key;
  }
  return a;
}
```

- **Vaqt:** O(n²) worst/o'rtacha, **O(n) eng yaxshi** (deyarli saralangan).
- **Xotira:** O(1).
- **Stable:** ha.

**💡 Tushuncha:** Insertion sort — **adaptive** (deyarli saralangan ma'lumotda tez), **stable**, **online** (elementlar oqib kelganda ham saralay oladi). Shuning uchun u O(n²) bo'lsa-da, kichik massivlarda merge/quick sort'dan tez.

---

## Merge Sort — O(n log n)

**💡 Tushuncha:** **Divide and conquer**: massivni ikkiga bo'lamiz, har yarmini rekursiv saralaymiz, keyin ikki saralangan yarmni bitta saralangan massivga **birlashtiramiz (merge)**. Birlashtirish ikki pointer bilan O(n) da bo'ladi.

```js
function mergeSort(arr) {
  if (arr.length <= 1) return arr;              // base case
  const mid = arr.length >> 1;
  const left = mergeSort(arr.slice(0, mid));
  const right = mergeSort(arr.slice(mid));
  return merge(left, right);
}

function merge(left, right) {
  const result = [];
  let i = 0, j = 0;
  while (i < left.length && j < right.length) {
    if (left[i] <= right[j]) result.push(left[i++]);  // <= → stable
    else result.push(right[j++]);
  }
  while (i < left.length) result.push(left[i++]);
  while (j < right.length) result.push(right[j++]);
  return result;
}
```

- **Vaqt:** O(n log n) har doim (worst ham!).
- **Xotira:** O(n) (qo'shimcha massiv).
- **Stable:** ha (`<=` ishlatilsa).

**⚠️ Ehtiyot bo'l:** Merge'da `left[i] <= right[j]` (qat'iy `<` emas) shart — bu teng elementlarda **chap** (avvalgi) elementni oldin oladi, ya'ni **stability** saqlanadi. Agar `<` qo'ysangiz stability buziladi.

---

## Quick Sort — O(n log n)

**💡 Tushuncha:** **Divide and conquer**, lekin merge'ning teskarisi: avval **partition** qilamiz (pivot tanlanib, undan kichiklar chapga, kattalar o'ngga joylanadi), keyin ikki qismni rekursiv saralaymiz. Birlashtirish kerak emas — partition o'zi joyiga qo'yadi. In-place ishlaydi.

```js
function quickSort(arr, lo = 0, hi = arr.length - 1) {
  if (lo < hi) {
    const p = partition(arr, lo, hi);
    quickSort(arr, lo, p - 1);
    quickSort(arr, p + 1, hi);
  }
  return arr;
}

function partition(arr, lo, hi) {
  const pivot = arr[hi];                         // oxirgi element pivot
  let i = lo - 1;
  for (let j = lo; j < hi; j++) {
    if (arr[j] < pivot) {
      i++;
      [arr[i], arr[j]] = [arr[j], arr[i]];
    }
  }
  [arr[i + 1], arr[hi]] = [arr[hi], arr[i + 1]]; // pivotni joyiga
  return i + 1;
}
```

- **Vaqt:** O(n log n) o'rtacha, **O(n²) worst** (yomon pivot, masalan allaqachon saralangan massivda oxirgi elementni pivot olsangiz).
- **Xotira:** O(log n) (rekursiya stack), in-place.
- **Stable:** yo'q.

**⚠️ Ehtiyot bo'l:** Quick sort'ning O(n²) worst case'i pivot tanlashga bog'liq. **Randomized pivot** yoki **median-of-three** strategiyasi bilan worst case'ni amalda chetlab o'tasiz. Suhbatda buni eslatish katta plus.

---

## Heap Sort — O(n log n)

**💡 Tushuncha:** Massivdan **max-heap** quramiz (eng katta element ildizda), keyin ildizni oxirgi element bilan almashtirib, heap hajmini kamaytirib, qayta heapify qilamiz. Natijada massiv o'sish bo'yicha saralanadi. In-place, qo'shimcha xotira yo'q.

```js
function heapSort(arr) {
  const n = arr.length;
  // 1) max-heap quramiz (oxiridan boshlab)
  for (let i = (n >> 1) - 1; i >= 0; i--) heapify(arr, n, i);
  // 2) ildizni oxirga olib, heapni kichraytiramiz
  for (let i = n - 1; i > 0; i--) {
    [arr[0], arr[i]] = [arr[i], arr[0]];
    heapify(arr, i, 0);
  }
  return arr;
}

function heapify(arr, n, i) {
  let largest = i, l = 2 * i + 1, r = 2 * i + 2;
  if (l < n && arr[l] > arr[largest]) largest = l;
  if (r < n && arr[r] > arr[largest]) largest = r;
  if (largest !== i) {
    [arr[i], arr[largest]] = [arr[largest], arr[i]];
    heapify(arr, n, largest);
  }
}
```

- **Vaqt:** O(n log n) har doim.
- **Xotira:** O(1) (in-place).
- **Stable:** yo'q.

**💡 Tushuncha:** Heap sort'ning afzalligi — **O(n log n) worst case + O(1) xotira**. Merge sort O(n) xotira ishlatadi, quick sort O(n²) worst case'ga ega. Lekin heap sort cache-unfriendly (xotirada sakrab ishlaydi), shuning uchun amalda quick sort'dan sekinroq.

---

## Stable vs Unstable sort

**💡 Tushuncha:** **Stable** sort teng kalitli elementlarning **dastlabki nisbiy tartibini** saqlaydi. Bu ko'p maydonli saralashda muhim: avval ism bo'yicha, keyin yosh bo'yicha saralasangiz, teng yoshlilar ism tartibida qoladi.

```js
// Stable: [{name:'A',age:30},{name:'B',age:30}] → age bo'yicha
// saralaganda A hali ham B'dan oldin turadi.
const people = [
  { name: 'A', age: 30 },
  { name: 'B', age: 25 },
  { name: 'C', age: 30 },
];
people.sort((x, y) => x.age - y.age);
// [B(25), A(30), C(30)] — A, C tartibi saqlanadi (stable)
```

| Stable | Unstable |
|---|---|
| merge, insertion, bubble, counting | quick, heap, selection |

**⚠️ Ehtiyot bo'l:** ES2019'dan boshlab `Array.prototype.sort()` **stable** bo'lishi spesifikatsiyada kafolatlangan. Eski muhitlar (eski V8, ba'zi engine'lar) unstable bo'lishi mumkin edi — eski kodga tayanmang.

---

## In-place vs Out-of-place

**💡 Tushuncha:** **In-place** algoritm qo'shimcha xotira deyarli ishlatmaydi (O(1) yoki O(log n) auxiliary) — kirish massivining o'zida ishlaydi. **Out-of-place** esa qo'shimcha massiv yaratadi (O(n)).

- **In-place:** bubble, selection, insertion, quick (O(log n) stack), heap.
- **Out-of-place:** merge sort (O(n)), counting/radix.

**⚠️ Ehtiyot bo'l:** Quick sort "in-place" deyiladi, lekin rekursiya O(log n) stack xotirasini oladi (worst case O(n)). "In-place" — bu O(n) qo'shimcha massiv yo'q degani, lekin stack ham xotira ekanini unutmang.

---

## Merge Sort vs Quick Sort taqqoslash

**💡 Tushuncha:** Bu suhbatlardagi eng mashhur taqqoslash. Ikkalasi ham divide-and-conquer, ikkalasi ham o'rtacha O(n log n), lekin trade-off'lari farq qiladi.

| Mezon | Merge Sort | Quick Sort |
|---|---|---|
| O'rtacha vaqt | O(n log n) | O(n log n) |
| **Worst case vaqt** | **O(n log n)** | **O(n²)** |
| **Xotira** | **O(n)** | **O(log n)** (in-place) |
| Stable | ✅ ha | ❌ yo'q |
| Amaliy tezlik | sekinroq (xotira/copy) | **tezroq** (cache-friendly) |
| Linked list uchun | ✅ ideal (O(1) xotira) | ❌ noqulay |
| Tashqi saralash (katta fayl) | ✅ ideal | ❌ |

**✅ Qachon qaysi:**
- **Quick sort** — umumiy holatda tezroq, in-place kerak bo'lsa (massiv, RAM).
- **Merge sort** — kafolatlangan O(n log n) kerak bo'lsa, stability kerak bo'lsa, linked list yoki diskdagi katta fayl bo'lsa.

**⚠️ Ehtiyot bo'l:** "Quick sort har doim tezroq" — noto'g'ri. Worst case'da (yomon pivot) O(n²) ga tushadi. Real-time tizimda kafolatlangan O(n log n) kerak bo'lsa, heap sort yoki merge sort tanlanadi.

---

## Array.sort() ichki ishlashi va comparator tuzog'i

**💡 Tushuncha:** JS'ning `Array.prototype.sort()` engine'ga bog'liq. V8 (Chrome/Node) **TimSort** ishlatadi — merge sort + insertion sort gibridi: kichik bo'laklarni insertion sort bilan, kattalarini merge bilan saralaydi. TimSort **stable** va deyarli saralangan ma'lumotda O(n) ga yaqin.

**⚠️ Ehtiyot bo'l — eng mashhur tuzoq:** Comparator'siz `sort()` elementlarni **string** sifatida solishtiradi!

```js
[10, 2, 1, 100].sort();          // [1, 10, 100, 2]  ❌ string tartibi!
[10, 2, 1, 100].sort((a, b) => a - b); // [1, 2, 10, 100] ✅
```

To'g'ri comparator qoidasi:

```js
arr.sort((a, b) => {
  // < 0  → a oldin keladi
  // > 0  → b oldin keladi
  // === 0 → tartib o'zgarmaydi
  return a - b;   // o'sish bo'yicha
});
arr.sort((a, b) => b - a);        // kamayish bo'yicha
```

**⚠️ Ehtiyot bo'l:** `a - b` faqat **sonlar** uchun ishlaydi. Stringlar uchun `localeCompare`, BigInt yoki katta sonlar uchun `a < b ? -1 : a > b ? 1 : 0` ishlating — `a - b` overflow yoki `NaN` berishi mumkin:

```js
// Stringlar:
words.sort((a, b) => a.localeCompare(b));
// Obyektlar:
users.sort((a, b) => a.age - b.age || a.name.localeCompare(b.name));
```

**⚠️ Ehtiyot bo'l:** `sort()` massivni **joyida** (in-place, mutates) o'zgartiradi va o'zini qaytaradi. Asl massivni saqlash uchun avval nusxa oling: `[...arr].sort(...)` yoki ES2023'da `arr.toSorted(...)`.

---

## Counting va Radix Sort (non-comparison)

**💡 Tushuncha:** Bu algoritmlar elementlarni **solishtirmaydi** — buning o'rniga ularning qiymatlarini "indeks" sifatida ishlatadi. Shu tufayli O(n log n) chegarasini chetlab, O(n) ga erishadi. Lekin faqat butun son yoki sanab bo'ladigan kalitlarda ishlaydi.

**Counting Sort** — har qiymat necha marta uchraganini sanab, keyin tartib bilan chiqaradi:

```js
function countingSort(arr) {
  const max = Math.max(...arr), min = Math.min(...arr);
  const count = new Array(max - min + 1).fill(0);
  for (const x of arr) count[x - min]++;        // sanash
  const result = [];
  for (let v = 0; v < count.length; v++)
    while (count[v]-- > 0) result.push(v + min); // chiqarish
  return result;
}
// Vaqt: O(n + k), k = qiymatlar diapazoni. Xotira: O(k).
```

**Radix Sort** — sonlarni **raqam-raqam** (eng kichik xonadan boshlab) counting sort bilan saralaydi:

```js
function radixSort(arr) {
  const max = Math.max(...arr);
  for (let exp = 1; Math.floor(max / exp) > 0; exp *= 10) {
    const output = new Array(arr.length);
    const count = new Array(10).fill(0);
    for (const x of arr) count[Math.floor(x / exp) % 10]++;
    for (let i = 1; i < 10; i++) count[i] += count[i - 1];
    for (let i = arr.length - 1; i >= 0; i--) {  // teskari → stable
      const d = Math.floor(arr[i] / exp) % 10;
      output[--count[d]] = arr[i];
    }
    arr = output;
  }
  return arr;
}
// Vaqt: O(d·(n+b)), d=xona soni, b=baza (10). Xotira: O(n+b).
```

**⚠️ Ehtiyot bo'l:** Counting sort'ning xotirasi qiymatlar **diapazoniga** (k) bog'liq. Agar diapazon juda katta bo'lsa (masalan [1, 10⁹]), counting sort 10⁹ kataklik massiv talab qiladi — bu yomon. Kichik, chegarali diapazonda (yoshlar, harflar) ideal.

---

## Binary Search — shablon va edge case'lar

**💡 Tushuncha:** **Saralangan** massivda har qadamda qidiruv oralig'ini **yarmiga** kamaytiramiz: o'rta elementni tekshirib, qaysi yarmda davom etishni hal qilamiz. O(log n) — 1 milliard element ichida ~30 qadamda topiladi.

```js
function binarySearch(arr, target) {
  let lo = 0, hi = arr.length - 1;
  while (lo <= hi) {                    // <= muhim!
    const mid = lo + ((hi - lo) >> 1);  // overflow'siz mid
    if (arr[mid] === target) return mid;
    if (arr[mid] < target) lo = mid + 1;
    else hi = mid - 1;
  }
  return -1;                            // topilmadi
}
// Vaqt: O(log n). Xotira: O(1).
```

**⚠️ Ehtiyot bo'l — klassik xatolar:**
1. `lo < hi` yozish (`<=` o'rniga) → bitta elementli oralig'i tekshirilmay qoladi.
2. `mid = (lo + hi) / 2` → katta sonlarda overflow (boshqa tillarda). `lo + ((hi - lo) >> 1)` xavfsiz.
3. `lo`/`hi` ni noto'g'ri yangilash → cheksiz tsikl.

**Lower bound** (target'dan kichik bo'lmagan birinchi indeks) — eng foydali variant:

```js
function lowerBound(arr, target) {
  let lo = 0, hi = arr.length;          // hi = length (mid+1 emas)
  while (lo < hi) {                      // bu yerda < ishlatiladi
    const mid = lo + ((hi - lo) >> 1);
    if (arr[mid] < target) lo = mid + 1;
    else hi = mid;
  }
  return lo;  // target qo'yiladigan pozitsiya
}
```

**💡 Tushuncha:** `lowerBound` bilan "birinchi/oxirgi uchrash", "kiritish pozitsiyasi" kabi masalalar yechiladi. Ikki shablonni (`lo<=hi` topish uchun, `lo<hi` chegara topish uchun) yodda saqlang.

---

## Binary Search on Answer pattern

**💡 Tushuncha:** Eng kuchli pattern. Agar javob **monoton** xususiyatga ega bo'lsa (ya'ni "x ishlasa, x+1 ham ishlaydi"), javobning **o'zi** ustida binary search qilamiz — massivda emas, **javoblar oralig'ida**. "Minimum/maksimum shu shartni qanoatlantiruvchi qiymat" masalalari shu pattern.

```js
// Misol: "Koko bananlarni soatiga k tezlikda yeydi, h soatda
// hammasini yeb bo'lishi uchun minimal k qancha?"
function minEatingSpeed(piles, h) {
  let lo = 1, hi = Math.max(...piles);
  while (lo < hi) {
    const k = lo + ((hi - lo) >> 1);
    const hours = piles.reduce((s, p) => s + Math.ceil(p / k), 0);
    if (hours <= h) hi = k;             // k ishladi → kichikroqni sina
    else lo = k + 1;                    // sekin → tezroq kerak
  }
  return lo;
}
// Vaqt: O(n · log(max)). n = piles, max = eng katta uyum.
```

**💡 Tushuncha:** Pattern'ni tanish: "minimal/maksimal X topingki, shart bajarilsin" + shartning monotonligi. `feasible(x)` funksiyasini yozasiz, keyin uning ustida binary search qilasiz.

**⚠️ Ehtiyot bo'l:** Eng qiyin qism — `lo`/`hi` chegaralarini va `feasible()` ning monotonligini to'g'ri aniqlash. Chegarani har doim "albatta ishlaydigan" va "albatta ishlamaydigan" qiymatlardan tashqarida oling.

---

## Search in Rotated Array

**💡 Tushuncha:** Saralangan massiv noma'lum nuqtada aylantirilgan (masalan `[4,5,6,7,0,1,2]`). Oddiy binary search ishlamaydi, lekin har bir yarim **har doim biri saralangan** — shuni aniqlab, target qaysi yarmda ekanini hal qilamiz. O(log n) saqlanadi.

```js
function searchRotated(arr, target) {
  let lo = 0, hi = arr.length - 1;
  while (lo <= hi) {
    const mid = lo + ((hi - lo) >> 1);
    if (arr[mid] === target) return mid;
    if (arr[lo] <= arr[mid]) {           // chap yarim saralangan
      if (arr[lo] <= target && target < arr[mid]) hi = mid - 1;
      else lo = mid + 1;
    } else {                             // o'ng yarim saralangan
      if (arr[mid] < target && target <= arr[hi]) lo = mid + 1;
      else hi = mid - 1;
    }
  }
  return -1;
}
// Vaqt: O(log n). Xotira: O(1).
```

**⚠️ Ehtiyot bo'l:** `arr[lo] <= arr[mid]` da `=` muhim — `mid === lo` bo'lganda (ikki elementli oraliq) chap yarim "saralangan" deb hisoblanishi kerak. Bu tushib qolsa edge case'larda xato bo'ladi. Dublikatlar bo'lsa (`[1,1,1,0,1]`) worst case O(n) ga tushadi.

---

## Intervyu Q&A

### ❓ Comparison-based sort nega O(n log n) dan tez bo'lolmaydi?

**✅ Javob:** Har bir solishtirish bitta "ha/yo'q" javob — 1 bit ma'lumot beradi. n ta elementning n! ta mumkin tartiblanishini ajratish uchun kamida log₂(n!) ta solishtirish kerak. Stirling formulasi bo'yicha log₂(n!) ≈ n log n. Demak hech qaysi comparison-based algoritm o'rtacha O(n log n) dan tez bo'lmaydi. O(n) ga faqat counting/radix kabi non-comparison algoritmlar (qo'shimcha bilim bilan) erishadi.

### ❓ Stable va unstable sort farqi nima, qachon muhim?

**✅ Javob:** Stable sort teng kalitli elementlarning dastlabki nisbiy tartibini saqlaydi. Bu **ko'p maydonli saralashda** muhim: avval ism, keyin yosh bo'yicha saralasangiz, stable sort teng yoshlilar ichida ism tartibini buzmaydi. Merge, insertion, counting — stable; quick, heap, selection — unstable. JS'da `Array.sort()` ES2019'dan stable.

### ❓ Merge sort va quick sort'ni qachon qaysi birini tanlaysiz?

**✅ Javob:** Quick sort — o'rtacha tezroq, in-place (O(log n) xotira), cache-friendly — umumiy holat uchun. Merge sort — kafolatlangan O(n log n) worst case, stable, linked list yoki diskdagi katta fayl (external sort) uchun ideal, lekin O(n) qo'shimcha xotira oladi. Real-time tizimda kafolatlangan vaqt kerak bo'lsa quick sort emas, merge yoki heap tanlanadi.

### ❓ Quick sort'ning worst case'i qachon va qanday oldini olasiz?

**✅ Javob:** Worst case O(n²) — pivot har doim eng kichik/katta element bo'lganda (masalan, allaqachon saralangan massivda oxirgi elementni pivot olsangiz, partition n-1 va 0 ga bo'linadi). Oldini olish: **randomized pivot** (tasodifiy element), **median-of-three** (boshi/o'rtasi/oxiri medianasi), yoki kichik bo'laklarda insertion sort'ga o'tish. Bular worst case'ni amalda chetlab o'tadi.

### ❓ `[10, 2, 1].sort()` nima qaytaradi va nega?

**✅ Javob:** `[1, 10, 2]` — comparator bermasangiz, `sort()` elementlarni **string** ga aylantirib leksikografik solishtiradi: "10" < "2" chunki "1" < "2". To'g'ri natija uchun `sort((a, b) => a - b)` kerak. Bu JS'ning eng mashhur tuzog'i.

### ❓ Sonlar uchun comparator'da `a - b` nega ba'zan xato beradi?

**✅ Javob:** `a - b` katta sonlarda integer overflow (boshqa tillarda) yoki `NaN`/`Infinity` aralashganda noto'g'ri ishlaydi, BigInt'da esa umuman ishlamaydi. Xavfsiz yo'l: `(a, b) => a < b ? -1 : a > b ? 1 : 0`. Stringlar uchun `localeCompare`. `a - b` faqat oddiy, chegarali butun/float sonlarda ishonchli.

### ❓ Binary search'da `mid = (lo + hi) / 2` nima muammoga olib keladi?

**✅ Javob:** `lo + hi` katta sonlarda integer overflow bo'lishi mumkin (JS'da 2⁵³ dan oshganda aniqlik yo'qoladi, C/Java'da int overflow). Xavfsiz formula: `lo + ((hi - lo) >> 1)` — bu hech qachon overflow bermaydi, chunki `hi - lo` har doim massiv hajmidan kichik.

### ❓ Binary search'da `while (lo <= hi)` va `while (lo < hi)` farqi nima?

**✅ Javob:** `lo <= hi` (hi = length-1) — aniq qiymat topish uchun, oraliq bo'shaganda (lo > hi) to'xtaydi, har bitta elementni tekshiradi. `lo < hi` (hi = length) — chegara topish (lower/upper bound) uchun, lo va hi yaqinlashguncha davom etadi, `mid` ni hech qachon o'tkazib yubormaydi. Qaysi pattern kerakligi masalaga bog'liq — ikkisini ham yodda saqlang.

### ❓ "Binary search on answer" pattern'ini qachon ishlatasiz?

**✅ Javob:** Masala "minimal/maksimal X topingki, shart bajarilsin" ko'rinishida bo'lsa va shart **monoton** bo'lsa (X ishlasa X+1 ham ishlaydi). Massivda emas, **javoblar oralig'ida** binary search qilasiz: `feasible(x)` funksiyasi yozib, uning chegarasini topasiz. Misollar: Koko Eating Bananas, ship within D days, split array largest sum.

### ❓ Rotated sorted array'da qanday binary search qilasiz?

**✅ Javob:** Har qadamda `mid` ni hisoblab, qaysi yarim **saralangan** ekanini aniqlaymiz (`arr[lo] <= arr[mid]` bo'lsa chap yarim saralangan). Saralangan yarimda target diapazonga tushsa o'sha yarmda davom etamiz, aks holda boshqa yarmda. O(log n) saqlanadi. Dublikatlar bo'lsa worst case O(n) ga tushadi (chunki teng elementlarni ajratib bo'lmaydi).

### ❓ Counting sort qachon merge/quick sort'dan yaxshi va qachon yomon?

**✅ Javob:** Yaxshi — elementlar butun son va **chegarali diapazonda** bo'lsa (yoshlar, baholar, harflar): O(n+k) va stable. Yomon — diapazon (k) juda katta bo'lsa: masalan [1, 10⁹] da 10⁹ kataklik massiv kerak, xotira portlaydi. Comparison sort'lar diapazonga bog'liq emas, shuning uchun katta yoki uzluksiz qiymatlarda ular tanlanadi.

### ❓ Heap sort'ning afzalligi nimada, nega amalda kam ishlatiladi?

**✅ Javob:** Afzalligi — **O(n log n) worst case + O(1) xotira** (merge O(n) xotira, quick O(n²) worst). Kamlik sababi — heap xotirada sakrab ishlaydi (poor cache locality), shuning uchun quick sort'dan amalda sekinroq, va u stable emas. Lekin kafolatlangan vaqt + minimal xotira kerak bo'lsa (embedded, real-time) ideal.

### ❓ TimSort nima va nega tez?

**✅ Javob:** TimSort — merge sort va insertion sort gibridi (Python, Java, V8/JS ishlatadi). U ma'lumotdagi tabiiy saralangan bo'laklarni ("runs") topadi, kichiklarini insertion sort bilan saralaydi, keyin merge qiladi. Deyarli saralangan ma'lumotda O(n) ga yaqin, har doim stable, worst case O(n log n). Real ma'lumot ko'pincha qisman saralangan bo'lgani uchun amalda juda tez.

### ❓ JS'da massivni saralab, asl massivni saqlash uchun nima qilasiz?

**✅ Javob:** `Array.sort()` in-place mutatsiya qiladi va asl massivni o'zgartiradi. Asl massivni saqlash uchun avval nusxa oling: `[...arr].sort(cmp)` yoki `arr.slice().sort(cmp)`. ES2023'dan `arr.toSorted(cmp)` yangi saralangan massiv qaytaradi, aslini o'zgartirmaydi — eng toza yo'l.

---

## Masalalar

> Yechimlar: [solutions/dsa/09-sorting-searching.md](../solutions/dsa/09-sorting-searching.md)

1. **Merge Sort implementatsiya.** Berilgan sonlar massivini merge sort bilan saralovchi funksiya yozing. Stable bo'lishini ta'minlang va `merge` qadamini O(n) da bajaring. Complexity'ni izohlang.

2. **Kth Largest Element.** Saralanmagan massivda k-chi eng katta elementni toping. Avval to'liq saralash (O(n log n)) bilan, keyin quickselect (o'rtacha O(n)) bilan yeching.

3. **Sort Colors (Dutch National Flag).** Faqat 0, 1, 2 lardan iborat massivni qo'shimcha massivsiz, bitta o'tishda (O(n), in-place) saralang. Uch pointer (low, mid, high) ishlating.

4. **Merge Intervals.** Oraliqlar massivi `[[1,3],[2,6],[8,10]]` berilgan. Kesishuvchi oraliqlarni birlashtiring. Avval boshlanish bo'yicha saralang, keyin bir o'tishda merge qiling.

5. **Binary Search.** Saralangan massivda target indeksini O(log n) da qaytaring, topilmasa -1. Edge case'larni (bo'sh massiv, bitta element, target chetda) sinab ko'ring.

6. **Find First and Last Position.** Saralangan, dublikatli massivda target'ning birinchi va oxirgi indeksini O(log n) da toping. Lower/upper bound binary search ishlating.

7. **Search in Rotated Sorted Array.** Aylantirilgan saralangan massivda target indeksini O(log n) da toping. Qaysi yarim saralangani aniqlash mantig'ini yozing.

8. **Koko Eating Bananas.** `piles` va `h` soat berilgan. Hamma bananni h soatda yeb bo'lish uchun minimal yeyish tezligini "binary search on answer" bilan toping.

9. **Counting Sort.** Diapazoni [0, 1000] bo'lgan butun sonlar massivini counting sort bilan O(n+k) da saralang. Stable bo'lishini ta'minlang.

10. **Comparator tuzog'i.** Obyektlar massivini avval `age` (o'sish), tenglikda `name` (alfavit) bo'yicha saralovchi to'g'ri comparator yozing. `a - b` tuzog'idan qoching.

11. **Search a 2D Matrix.** Har qatori saralangan va har qator boshi oldingi qator oxiridan katta bo'lgan matritsada target'ni O(log(m·n)) da qidiring (matritsani bitta saralangan massiv deb ko'ring).

12. **Median of Two Sorted Arrays.** Ikkita saralangan massivning umumiy medianasini O(log(min(m, n))) da toping (binary search partition).

← [DSA bo'limiga qaytish](./README.md)
