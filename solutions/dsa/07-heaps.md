# Heaps va Priority Queue — Yechimlar

Bu faylda [07-heaps.md](../../dsa/07-heaps.md) dagi masalalar yechimlari to'liq kod, complexity tahlili va izohlar bilan keltirilgan. Barcha yechimlar quyidagi `MinHeap` class'ga tayanadi.

```ts
class MinHeap<T> {
  heap: T[] = [];
  constructor(private compare: (a: T, b: T) => number = (a, b) => (a as any) - (b as any)) {}
  size() { return this.heap.length; }
  isEmpty() { return this.heap.length === 0; }
  peek(): T | undefined { return this.heap[0]; }
  push(val: T) {
    this.heap.push(val);
    let i = this.heap.length - 1;
    while (i > 0) {
      const p = (i - 1) >> 1;
      if (this.compare(this.heap[p], this.heap[i]) <= 0) break;
      [this.heap[p], this.heap[i]] = [this.heap[i], this.heap[p]];
      i = p;
    }
  }
  pop(): T | undefined {
    if (this.heap.length === 0) return undefined;
    const top = this.heap[0];
    const last = this.heap.pop()!;
    if (this.heap.length) {
      this.heap[0] = last;
      let i = 0, n = this.heap.length;
      while (true) {
        let s = i; const l = 2*i+1, r = 2*i+2;
        if (l < n && this.compare(this.heap[l], this.heap[s]) < 0) s = l;
        if (r < n && this.compare(this.heap[r], this.heap[s]) < 0) s = r;
        if (s === i) break;
        [this.heap[i], this.heap[s]] = [this.heap[s], this.heap[i]];
        i = s;
      }
    }
    return top;
  }
}
```

---

## 1. Kth Largest Element in an Array

**💡 G'oya:** K o'lchovli min-heap saqlaymiz. Heap K dan oshsa ildizni chiqaramiz. Oxirida ildiz — K-chi eng katta.

```js
function findKthLargest(nums, k) {
  const h = new MinHeap((a, b) => a - b);
  for (const n of nums) {
    h.push(n);
    if (h.size() > k) h.pop();
  }
  return h.peek();
}
```

**Complexity:** Time O(n log k), Space O(k).
**Izoh:** Quickselect bilan o'rtacha O(n) ga ham yetish mumkin, lekin heap-yondashuv kod jihatidan soddaroq va eng yomon holatda barqaror O(n log k).

---

## 2. Kth Smallest Element in a Sorted Matrix

**💡 G'oya:** Har bir qatorning birinchi elementini heap'ga qo'shamiz. K marta eng kichigini chiqarib, shu qatordagi keyingi elementni qo'shamiz.

```js
function kthSmallest(matrix, k) {
  const n = matrix.length;
  // [value, row, col]
  const h = new MinHeap((a, b) => a[0] - b[0]);
  for (let r = 0; r < Math.min(n, k); r++) h.push([matrix[r][0], r, 0]);

  let result;
  for (let i = 0; i < k; i++) {
    const [val, r, c] = h.pop();
    result = val;
    if (c + 1 < matrix[r].length) h.push([matrix[r][c + 1], r, c + 1]);
  }
  return result;
}
```

**Complexity:** Time O(k log n), Space O(n).
**Izoh:** Binary search on value bilan O(n log(max-min)) yechim ham bor, lekin heap-yondashuv intuitiv.

---

## 3. Top K Frequent Elements

**💡 G'oya:** Chastotani Map'da hisoblaymiz, so'ng chastota bo'yicha K o'lchovli min-heap.

```js
function topKFrequent(nums, k) {
  const freq = new Map();
  for (const n of nums) freq.set(n, (freq.get(n) || 0) + 1);

  const h = new MinHeap((a, b) => a[1] - b[1]); // [value, count]
  for (const [val, count] of freq) {
    h.push([val, count]);
    if (h.size() > k) h.pop();
  }
  return h.heap.map(([val]) => val);
}
```

**Complexity:** Time O(n log k), Space O(n).
**Izoh:** Bucket sort bilan O(n) ham mumkin (chastotani indeks sifatida ishlatib).

---

## 4. Top K Frequent Words

**💡 G'oya:** Chastota o'sish bo'yicha (kichigi yuqorida), teng chastotada esa alifbo **teskari** tartibda heap. Oxirida natijani teskari qaytaramiz.

```js
function topKFrequentWords(words, k) {
  const freq = new Map();
  for (const w of words) freq.set(w, (freq.get(w) || 0) + 1);

  // min-heap: kam chastotali yoki (teng chastotada) "katta" so'z yuqorida bo'lsin
  const h = new MinHeap((a, b) => {
    if (a[1] !== b[1]) return a[1] - b[1];       // chastota o'sish
    return b[0].localeCompare(a[0]);             // alifbo teskari
  });
  for (const [w, c] of freq) {
    h.push([w, c]);
    if (h.size() > k) h.pop();
  }
  const res = [];
  while (!h.isEmpty()) res.push(h.pop()[0]);
  return res.reverse(); // eng yuqori chastota oldinga
}
```

**Complexity:** Time O(n log k), Space O(n).
**Izoh:** Tie-break tartibi nozik — alifbo bo'yicha teskari solishtirish min-heap ichida to'g'ri so'zni "yo'qotish"ga olib keladi.

---

## 5. Merge K Sorted Lists

**💡 G'oya:** Har bir list'ning bosh node'ini heap'ga. Eng kichigini olib, uning `next`ini qo'shamiz.

```js
function mergeKLists(lists) {
  const h = new MinHeap((a, b) => a.val - b.val);
  for (const node of lists) if (node) h.push(node);

  const dummy = { val: 0, next: null };
  let tail = dummy;
  while (!h.isEmpty()) {
    const node = h.pop();
    tail.next = node;
    tail = node;
    if (node.next) h.push(node.next);
  }
  return dummy.next;
}
```

**Complexity:** Time O(N log k) (N — jami node), Space O(k).
**Izoh:** Heap o'lchami doim ≤ k bo'ladi, shuning uchun har push/pop O(log k).

---

## 6. Find Median from Data Stream

**💡 G'oya:** Ikki heap — chap yarmi uchun max-heap, o'ng yarmi uchun min-heap. O'lchamlarni balansda saqlaymiz.

```js
class MedianFinder {
  constructor() {
    this.lo = new MinHeap((a, b) => b - a); // max-heap (kichik yarmi)
    this.hi = new MinHeap((a, b) => a - b); // min-heap (katta yarmi)
  }
  addNum(num) {
    this.lo.push(num);
    this.hi.push(this.lo.pop());           // lo eng kattasini hi ga
    if (this.hi.size() > this.lo.size()) {
      this.lo.push(this.hi.pop());         // balans
    }
  }
  findMedian() {
    if (this.lo.size() > this.hi.size()) return this.lo.peek();
    return (this.lo.peek() + this.hi.peek()) / 2;
  }
}
```

**Complexity:** addNum O(log n), findMedian O(1), Space O(n).
**Izoh:** `lo` doim teng yoki bitta ko'p element saqlaydi — shu sabab toq holatda median `lo.peek()`.

---

## 7. K Closest Points to Origin

**💡 G'oya:** Masofa kvadrati (sqrt shart emas) bo'yicha K o'lchovli max-heap. K dan oshsa eng uzog'ini chiqaramiz.

```js
function kClosest(points, k) {
  const dist = ([x, y]) => x * x + y * y;
  const h = new MinHeap((a, b) => dist(b) - dist(a)); // max-heap
  for (const p of points) {
    h.push(p);
    if (h.size() > k) h.pop();
  }
  return h.heap;
}
```

**Complexity:** Time O(n log k), Space O(k).
**Izoh:** `Math.sqrt` keraksiz — kvadratlar solishtirish uchun yetarli, bu aniqlik va tezlikni saqlaydi.

---

## 8. Last Stone Weight

**💡 G'oya:** Max-heap. Ikki eng kattasini olib, farqini qaytaramiz.

```js
function lastStoneWeight(stones) {
  const h = new MinHeap((a, b) => b - a); // max-heap
  for (const s of stones) h.push(s);
  while (h.size() > 1) {
    const a = h.pop(), b = h.pop();
    if (a !== b) h.push(a - b);
  }
  return h.isEmpty() ? 0 : h.peek();
}
```

**Complexity:** Time O(n log n), Space O(n).

---

## 9. Task Scheduler

**💡 G'oya:** Eng ko'p uchraydigan vazifani aniqlash kifoya. Formula: `max(jami, (maxFreq - 1) * (n + 1) + maxFreqElumlari)`.

```js
function leastInterval(tasks, n) {
  const freq = new Map();
  for (const t of tasks) freq.set(t, (freq.get(t) || 0) + 1);
  const counts = [...freq.values()];
  const maxFreq = Math.max(...counts);
  const maxCount = counts.filter(c => c === maxFreq).length;

  const slots = (maxFreq - 1) * (n + 1) + maxCount;
  return Math.max(tasks.length, slots);
}
```

**Complexity:** Time O(n), Space O(1) (eng ko'pi 26 ta harf).
**Izoh:** Heap bilan simulyatsiya ham mumkin (har round eng ko'p chastotalilarni tanlash), lekin formula O(n) va soddaroq.

---

## 10. Reorganize String

**💡 G'oya:** Max-heap chastota bo'yicha. Har qadamda eng ko'p uchraydigan ikki **har xil** harfni navbatma-navbat qo'yamiz.

```js
function reorganizeString(s) {
  const freq = new Map();
  for (const c of s) freq.set(c, (freq.get(c) || 0) + 1);

  const h = new MinHeap((a, b) => b[1] - a[1]); // [char, count] max-heap
  for (const [c, cnt] of freq) h.push([c, cnt]);

  let prev = null, res = '';
  while (!h.isEmpty()) {
    const [c, cnt] = h.pop();
    res += c;
    if (prev && prev[1] > 0) h.push(prev); // oldingisini qaytaramiz
    prev = [c, cnt - 1];
  }
  return res.length === s.length ? res : '';
}
```

**Complexity:** Time O(n log k) (k ≤ 26), Space O(k).
**Izoh:** `prev`ni bir qadam ushlab turish qo'shni bir xil harflar bo'lishini oldini oladi.

---

## 11. Sort Characters By Frequency

**💡 G'oya:** Chastotani sanab, max-heap orqali kamayish tartibida string yasaymiz.

```js
function frequencySort(s) {
  const freq = new Map();
  for (const c of s) freq.set(c, (freq.get(c) || 0) + 1);

  const h = new MinHeap((a, b) => b[1] - a[1]); // [char, count]
  for (const [c, cnt] of freq) h.push([c, cnt]);

  let res = '';
  while (!h.isEmpty()) {
    const [c, cnt] = h.pop();
    res += c.repeat(cnt);
  }
  return res;
}
```

**Complexity:** Time O(n + k log k), Space O(n).

---

## 12. Sliding Window Median

**💡 G'oya:** Ikki heap (max-heap `lo`, min-heap `hi`) + "lazy deletion" (kechiktirilgan o'chirish), chunki heap'da ixtiyoriy elementni o'chirish qiyin.

```js
function medianSlidingWindow(nums, k) {
  const result = [];
  const lo = new MinHeap((a, b) => b - a); // max-heap
  const hi = new MinHeap((a, b) => a - b); // min-heap
  const removed = new Map(); // lazy deletion hisoblagichi
  let balance = 0; // lo va hi balansi

  const prune = (heap) => {
    while (!heap.isEmpty() && removed.get(heap.peek()) > 0) {
      removed.set(heap.peek(), removed.get(heap.peek()) - 1);
      heap.pop();
    }
  };

  for (let i = 0; i < nums.length; i++) {
    // qo'shish
    if (lo.isEmpty() || nums[i] <= lo.peek()) { lo.push(nums[i]); balance++; }
    else { hi.push(nums[i]); balance--; }

    // chiqib ketadigan element (lazy)
    if (i >= k) {
      const out = nums[i - k];
      removed.set(out, (removed.get(out) || 0) + 1);
      if (out <= lo.peek()) balance--; else balance++;
    }

    // balanslash
    if (balance > 1) { hi.push(lo.pop()); balance -= 2; }
    else if (balance < 0) { lo.push(hi.pop()); balance += 2; }

    prune(lo); prune(hi);

    if (i >= k - 1) {
      const median = (k % 2)
        ? lo.peek()
        : (lo.peek() + hi.peek()) / 2;
      result.push(median);
    }
  }
  return result;
}
```

**Complexity:** Time O(n log k), Space O(k).
**Izoh:** Lazy deletion — heap'dan ixtiyoriy elementni darrov o'chirib bo'lmagani uchun, uni "o'chirilgan" deb belgilab, ildizga chiqqanida tashlaymiz. `balance` haqiqiy (o'chirilmagan) elementlar bo'yicha hisoblanadi.

---

← [DSA bo'limiga qaytish](../../dsa/README.md)
