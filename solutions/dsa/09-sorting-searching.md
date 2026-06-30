# Sorting va Searching — Yechimlar

Bu bo'lim `09-sorting-searching.md` dagi 12 ta masalaning to'liq ishlaydigan JS yechimlarini, complexity tahlilini va o'zbekcha izohlarni o'z ichiga oladi.

## 1. Merge Sort implementatsiya

Divide and conquer: massivni ikkiga bo'lib, har yarmini rekursiv saralab, ikki saralangan yarmni O(n) da birlashtiramiz.

```js
function mergeSort(arr) {
  if (arr.length <= 1) return arr; // base case
  const mid = arr.length >> 1;
  const left = mergeSort(arr.slice(0, mid));
  const right = mergeSort(arr.slice(mid));
  return merge(left, right);
}

function merge(left, right) {
  const result = [];
  let i = 0, j = 0;
  while (i < left.length && j < right.length) {
    if (left[i] <= right[j]) result.push(left[i++]); // <= → stable
    else result.push(right[j++]);
  }
  while (i < left.length) result.push(left[i++]);
  while (j < right.length) result.push(right[j++]);
  return result;
}

console.log(mergeSort([5, 2, 8, 1, 9, 3])); // [1, 2, 3, 5, 8, 9]
```

- **Time:** O(n log n) har doim (worst case ham) — log n daraja, har darajada O(n) merge.
- **Space:** O(n) — qo'shimcha massivlar.

**Izoh:** Merge'da `left[i] <= right[j]` (qat'iy `<` emas) teng elementlarda chap (avvalgi) elementni oldin oladi — bu **stability**ni saqlaydi. Agar `<` qo'ysangiz stability buziladi.

---

## 2. Kth Largest Element

k-chi eng katta elementni topamiz. Ikki yo'l: to'liq saralash (O(n log n)) va quickselect (o'rtacha O(n)).

```js
// Variant A: saralash — O(n log n)
function findKthLargestSort(nums, k) {
  return [...nums].sort((a, b) => b - a)[k - 1];
}

// Variant B: quickselect — o'rtacha O(n)
function findKthLargest(nums, k) {
  const target = nums.length - k; // k-chi eng katta = (n-k)-chi eng kichik (0-indexed)
  const a = [...nums];

  function quickselect(lo, hi) {
    const pivot = a[hi];
    let i = lo;
    for (let j = lo; j < hi; j++) {
      if (a[j] < pivot) { [a[i], a[j]] = [a[j], a[i]]; i++; }
    }
    [a[i], a[hi]] = [a[hi], a[i]];
    if (i === target) return a[i];
    if (i < target) return quickselect(i + 1, hi);
    return quickselect(lo, i - 1);
  }

  return quickselect(0, a.length - 1);
}

console.log(findKthLargestSort([3, 2, 1, 5, 6, 4], 2)); // 5
console.log(findKthLargest([3, 2, 1, 5, 6, 4], 2));     // 5
```

- **Saralash:** Time O(n log n), Space O(n).
- **Quickselect:** Time o'rtacha O(n), worst O(n²); Space O(1) (in-place, rekursiya hisobga olmaganda).

**Izoh:** Quickselect — quick sort'ning bir tomonlama versiyasi: faqat target indeksni o'z ichiga olgan qismni rekursiv qayta ishlaymiz. Worst case'dan qochish uchun randomized pivot ishlatish mumkin.

---

## 3. Sort Colors (Dutch National Flag)

Faqat 0, 1, 2 lardan iborat massivni qo'shimcha massivsiz, bitta o'tishda, in-place saralaymiz. Uch pointer.

```js
function sortColors(nums) {
  let low = 0, mid = 0, high = nums.length - 1;
  while (mid <= high) {
    if (nums[mid] === 0) {
      [nums[low], nums[mid]] = [nums[mid], nums[low]];
      low++; mid++;
    } else if (nums[mid] === 1) {
      mid++;
    } else { // === 2
      [nums[mid], nums[high]] = [nums[high], nums[mid]];
      high--; // mid ortmaydi — almashtirib kelgan elementni qayta tekshiramiz
    }
  }
  return nums;
}

console.log(sortColors([2, 0, 2, 1, 1, 0])); // [0, 0, 1, 1, 2, 2]
```

- **Time:** O(n) — bitta o'tish.
- **Space:** O(1) — in-place.

**Izoh:** `low` 0'lar chegarasini, `high` 2'lar chegarasini ushlab turadi. `2` bilan almashtirganda `mid` ni **oshirmaymiz**, chunki `high` dan kelgan element hali tekshirilmagan.

---

## 4. Merge Intervals

Kesishuvchi oraliqlarni birlashtiramiz. Avval boshlanish bo'yicha saralab, keyin bir o'tishda merge qilamiz.

```js
function mergeIntervals(intervals) {
  if (intervals.length <= 1) return intervals;
  intervals.sort((a, b) => a[0] - b[0]); // boshlanish bo'yicha

  const result = [intervals[0]];
  for (let i = 1; i < intervals.length; i++) {
    const last = result[result.length - 1];
    const [start, end] = intervals[i];
    if (start <= last[1]) {
      last[1] = Math.max(last[1], end); // kesishadi → kengaytirish
    } else {
      result.push([start, end]); // kesishmaydi → yangi oraliq
    }
  }
  return result;
}

console.log(mergeIntervals([[1,3],[2,6],[8,10],[15,18]])); // [[1,6],[8,10],[15,18]]
```

- **Time:** O(n log n) — saralash hisobiga.
- **Space:** O(n) — natija massivi.

**Izoh:** Saralashdan keyin kesishuvchi oraliqlar yonma-yon turadi. `start <= last[1]` bo'lsa ular qoplanadi va `last[1]` ni maksimum bilan kengaytiramiz.

---

## 5. Binary Search

Saralangan massivda target indeksini O(log n) da qaytaramiz, topilmasa -1.

```js
function binarySearch(arr, target) {
  let lo = 0, hi = arr.length - 1;
  while (lo <= hi) {                    // <= muhim
    const mid = lo + ((hi - lo) >> 1);  // overflow'siz mid
    if (arr[mid] === target) return mid;
    if (arr[mid] < target) lo = mid + 1;
    else hi = mid - 1;
  }
  return -1;
}

console.log(binarySearch([1, 3, 5, 7, 9], 7)); // 3
console.log(binarySearch([1, 3, 5, 7, 9], 4)); // -1
console.log(binarySearch([], 1));              // -1
console.log(binarySearch([5], 5));             // 0
```

- **Time:** O(log n).
- **Space:** O(1).

**Izoh:** `lo <= hi` sharti bitta elementli oraliqni ham tekshiradi. `lo + ((hi - lo) >> 1)` overflow xavfsiz mid (boshqa tillarda muhim). Edge case'lar: bo'sh massiv, bitta element, target chetda — hammasi to'g'ri ishlaydi.

---

## 6. Find First and Last Position

Saralangan, dublikatli massivda target'ning birinchi va oxirgi indeksini O(log n) da topamiz. Lower/upper bound binary search.

```js
function searchRange(nums, target) {
  const lowerBound = (t) => {
    let lo = 0, hi = nums.length;
    while (lo < hi) {
      const mid = lo + ((hi - lo) >> 1);
      if (nums[mid] < t) lo = mid + 1;
      else hi = mid;
    }
    return lo;
  };

  const first = lowerBound(target);
  if (first === nums.length || nums[first] !== target) return [-1, -1];
  const last = lowerBound(target + 1) - 1; // target+1 ning lower bound - 1
  return [first, last];
}

console.log(searchRange([5, 7, 7, 8, 8, 10], 8)); // [3, 4]
console.log(searchRange([5, 7, 7, 8, 8, 10], 6)); // [-1, -1]
```

- **Time:** O(log n) — ikkita binary search.
- **Space:** O(1).

**Izoh:** `lowerBound(target)` — target'dan kichik bo'lmagan birinchi indeks (birinchi uchrash). `lowerBound(target + 1) - 1` — target'dan katta birinchi indeksdan oldingisi (oxirgi uchrash). Bu hiyla upper bound'ni alohida yozishdan qochiradi.

---

## 7. Search in Rotated Sorted Array

Aylantirilgan saralangan massivda target indeksini O(log n) da topamiz. Har qadamda qaysi yarim saralanganini aniqlaymiz.

```js
function search(nums, target) {
  let lo = 0, hi = nums.length - 1;
  while (lo <= hi) {
    const mid = lo + ((hi - lo) >> 1);
    if (nums[mid] === target) return mid;

    if (nums[lo] <= nums[mid]) {           // chap yarim saralangan
      if (nums[lo] <= target && target < nums[mid]) hi = mid - 1;
      else lo = mid + 1;
    } else {                               // o'ng yarim saralangan
      if (nums[mid] < target && target <= nums[hi]) lo = mid + 1;
      else hi = mid - 1;
    }
  }
  return -1;
}

console.log(search([4, 5, 6, 7, 0, 1, 2], 0)); // 4
console.log(search([4, 5, 6, 7, 0, 1, 2], 3)); // -1
```

- **Time:** O(log n).
- **Space:** O(1).

**Izoh:** `nums[lo] <= nums[mid]` da `=` muhim — `mid === lo` (ikki elementli oraliq) bo'lganda chap yarim "saralangan" deb hisoblanishi kerak. Saralangan yarimda target diapazonga tushsa o'sha yarmda davom etamiz.

---

## 8. Koko Eating Bananas

`piles` va `h` soat berilgan. Hamma bananni h soatda yeb bo'lish uchun minimal tezlikni "binary search on answer" bilan topamiz.

```js
function minEatingSpeed(piles, h) {
  let lo = 1, hi = Math.max(...piles);

  const hoursNeeded = (k) =>
    piles.reduce((sum, p) => sum + Math.ceil(p / k), 0);

  while (lo < hi) {
    const k = lo + ((hi - lo) >> 1);
    if (hoursNeeded(k) <= h) hi = k;  // k yetarli → kichikroqni sina
    else lo = k + 1;                  // sekin → tezroq kerak
  }
  return lo;
}

console.log(minEatingSpeed([3, 6, 7, 11], 8)); // 4
console.log(minEatingSpeed([30, 11, 23, 4, 20], 5)); // 30
```

- **Time:** O(n · log(max)) — n = uyumlar, max = eng katta uyum.
- **Space:** O(1).

**Izoh:** `feasible(k)` = "k tezlikda h soatga ulgurish" — monoton: k kattalashsa, vaqt kamayadi. Shuning uchun javoblar oralig'ida [1, max] binary search qilamiz va "ishlaydigan" eng kichik k ni topamiz.

---

## 9. Counting Sort

Diapazoni chegarali butun sonlar massivini O(n+k) da saralaymiz. Stable bo'lishi uchun prefix-sum + teskari o'tish.

```js
function countingSort(arr) {
  if (arr.length === 0) return [];
  const max = Math.max(...arr), min = Math.min(...arr);
  const range = max - min + 1;
  const count = new Array(range).fill(0);

  for (const x of arr) count[x - min]++;          // sanash
  for (let i = 1; i < range; i++) count[i] += count[i - 1]; // prefix sum

  const output = new Array(arr.length);
  for (let i = arr.length - 1; i >= 0; i--) {     // teskari → stable
    const idx = arr[i] - min;
    output[--count[idx]] = arr[i];
  }
  return output;
}

console.log(countingSort([4, 2, 2, 8, 3, 3, 1])); // [1, 2, 2, 3, 3, 4, 8]
```

- **Time:** O(n + k) — k = qiymatlar diapazoni.
- **Space:** O(n + k).

**Izoh:** Prefix-sum count har qiymatning oxirgi pozitsiyasini beradi. **Teskari** o'tish (oxiridan boshlash) + `--count` teng elementlarda dastlabki tartibni saqlaydi — bu **stability**ni ta'minlaydi. Diapazon juda katta bo'lsa (masalan 10⁹) bu yondashuv xotira jihatdan yaroqsiz.

---

## 10. Comparator tuzog'i

Obyektlar massivini avval `age` (o'sish), tenglikda `name` (alfavit) bo'yicha saralaymiz. `a - b` tuzog'idan qochamiz.

```js
function sortPeople(people) {
  return [...people].sort((a, b) =>
    a.age - b.age || a.name.localeCompare(b.name)
  );
}

const people = [
  { name: 'Bobur', age: 30 },
  { name: 'Aziz', age: 25 },
  { name: 'Aliya', age: 30 },
  { name: 'Davron', age: 25 },
];
console.log(sortPeople(people));
// [Aziz(25), Davron(25), Aliya(30), Bobur(30)]
```

- **Time:** O(n log n).
- **Space:** O(n) — `[...people]` nusxasi.

**Izoh:** `a.age - b.age || a.name.localeCompare(b.name)` — agar yoshlar teng bo'lsa (`0`, falsy), `||` orqali ism solishtiruviga o'tadi. Stringlar uchun `a - b` ishlamaydi (`NaN`), shuning uchun `localeCompare`. `[...people]` aslini buzmaslik uchun nusxa oladi.

---

## 11. Search a 2D Matrix

Har qatori saralangan va qatorlar zanjirsimon tartibli matritsada target'ni O(log(m·n)) da qidiramiz. Matritsani bitta saralangan massiv deb ko'ramiz.

```js
function searchMatrix(matrix, target) {
  const m = matrix.length, n = matrix[0].length;
  let lo = 0, hi = m * n - 1;
  while (lo <= hi) {
    const mid = lo + ((hi - lo) >> 1);
    const val = matrix[Math.floor(mid / n)][mid % n]; // 1D → 2D
    if (val === target) return true;
    if (val < target) lo = mid + 1;
    else hi = mid - 1;
  }
  return false;
}

console.log(searchMatrix([[1,3,5,7],[10,11,16,20],[23,30,34,60]], 3));  // true
console.log(searchMatrix([[1,3,5,7],[10,11,16,20],[23,30,34,60]], 13)); // false
```

- **Time:** O(log(m·n)).
- **Space:** O(1).

**Izoh:** Matritsa to'liq saralangan ketma-ketlik bo'lgani uchun bitta `m*n` uzunlikdagi massiv deb tasavvur qilamiz. `mid` indeksni qatorga `Math.floor(mid/n)` va ustunga `mid % n` formulasi bilan aylantiramiz.

---

## 12. Median of Two Sorted Arrays

Ikkita saralangan massivning umumiy medianasini O(log(min(m, n))) da topamiz. Kichikroq massiv ustida binary search partition.

```js
function findMedianSortedArrays(nums1, nums2) {
  if (nums1.length > nums2.length) [nums1, nums2] = [nums2, nums1]; // kichikroq A
  const m = nums1.length, n = nums2.length;
  const half = (m + n + 1) >> 1;

  let lo = 0, hi = m;
  while (lo <= hi) {
    const i = lo + ((hi - lo) >> 1); // A dagi partition
    const j = half - i;              // B dagi partition

    const Aleft  = i > 0 ? nums1[i - 1] : -Infinity;
    const Aright = i < m ? nums1[i]     :  Infinity;
    const Bleft  = j > 0 ? nums2[j - 1] : -Infinity;
    const Bright = j < n ? nums2[j]     :  Infinity;

    if (Aleft <= Bright && Bleft <= Aright) { // to'g'ri partition
      if ((m + n) % 2 === 1) return Math.max(Aleft, Bleft);
      return (Math.max(Aleft, Bleft) + Math.min(Aright, Bright)) / 2;
    } else if (Aleft > Bright) {
      hi = i - 1; // A da chapga
    } else {
      lo = i + 1; // A da o'ngga
    }
  }
  throw new Error('Massivlar saralangan emas');
}

console.log(findMedianSortedArrays([1, 3], [2]));    // 2
console.log(findMedianSortedArrays([1, 2], [3, 4])); // 2.5
```

- **Time:** O(log(min(m, n))).
- **Space:** O(1).

**Izoh:** Ikki massivni shunday "kesamiz"ki, chap qismda jami `(m+n+1)/2` ta element bo'lsin va `Aleft <= Bright`, `Bleft <= Aright` shartlari bajarilsin. Bu topilganda median chegara elementlardan hisoblanadi. Kichikroq massiv ustida qidirish O(log(min(m,n)))ni kafolatlaydi.

---

← [DSA bo'limiga qaytish](../../dsa/README.md)
