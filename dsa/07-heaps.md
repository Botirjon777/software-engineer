# Heaps va Priority Queue

Heap — bu **complete binary tree** ko'rinishidagi maxsus ma'lumotlar strukturasi bo'lib, undan eng kichik yoki eng katta elementni doimo O(1) vaqtda olish imkonini beradi. Priority Queue (ustuvor navbat) odatda aynan heap orqali amalga oshiriladi. Bu hujjatda heap qanday ishlashi, array ko'rinishi, asosiy operatsiyalar va eng ko'p uchraydigan pattern'larni ko'rib chiqamiz.

## Mundarija

- [Heap nima](#heap-nima)
- [Complete binary tree va array ko'rinishi](#complete-binary-tree-va-array-korinishi)
- [Asosiy operatsiyalar](#asosiy-operatsiyalar)
- [Build heap O(n)](#build-heap-on)
- [Priority Queue](#priority-queue)
- [JS'da MinHeap implementatsiyasi](#jsda-minheap-implementatsiyasi)
- [Use-case'lar](#use-caselar)
- [Pattern'lar](#patternlar)
- [Savol-javoblar](#savol-javoblar)
- [Masalalar](#masalalar)

---

## Heap nima

**💡 Tushuncha:** Heap — bu **heap property** (heap xususiyati) ga bo'ysunadigan complete binary tree. Ikki turi bor:

- **Min-heap:** har bir node'ning qiymati o'z farzandlaridan (children) kichik yoki teng. Ildiz (root) — eng kichik element.
- **Max-heap:** har bir node'ning qiymati o'z farzandlaridan katta yoki teng. Ildiz — eng katta element.

Diqqat: heap **to'liq tartiblangan emas**. Faqat parent-child munosabati kafolatlanadi, aka-uka (sibling) node'lar o'rtasida hech qanday tartib yo'q. Shuning uchun heap'dan ketma-ket extract qilsangiz, elementlar tartiblangan ketma-ketlikda chiqadi, lekin ichki array tartiblangan bo'lmaydi.

```
Min-heap misol:           Max-heap misol:
        1                        9
       / \                      / \
      3   2                    7   8
     / \  /                   / \  /
    7  4 5                   3  4 5
```

---

## Complete binary tree va array ko'rinishi

**💡 Tushuncha:** Complete binary tree — bu daraxt bo'lib, oxirgi darajadan (level) tashqari barcha darajalar to'liq to'ldirilgan, oxirgi daraja esa chapdan o'ngga qarab to'ldiriladi. Aynan shu xususiyat tufayli heap'ni **array** orqali ifodalash mumkin — pointer'larsiz.

`0`-indeksdan boshlanadigan array uchun index formulalari:

```js
// node index = i bo'lsa:
parent(i) = Math.floor((i - 1) / 2)
leftChild(i)  = 2 * i + 1
rightChild(i) = 2 * i + 2
```

Misol: `[1, 3, 2, 7, 4, 5]` arrayi yuqoridagi min-heap'ni ifodalaydi.

```
Index:  0  1  2  3  4  5
Value:  1  3  2  7  4  5

index 0 (qiymat 1) → left = index 1 (3), right = index 2 (2)
index 1 (qiymat 3) → left = index 3 (7), right = index 4 (4)
index 2 (qiymat 2) → left = index 5 (5)
```

**⚠️ Ehtiyot bo'l:** Agar array `1`-indeksdan boshlansa, formulalar boshqacha: `parent = i/2`, `left = 2*i`, `right = 2*i+1`. Ko'pchilik JS implementatsiyalari `0`-indeksdan boshlanadi, shuning uchun formulalaringizni doim tekshiring.

---

## Asosiy operatsiyalar

**💡 Tushuncha:** Heap'da ikki asosiy "tiklash" (restore) operatsiyasi bor: **heapify-up** (sift-up) va **heapify-down** (sift-down). Ular heap property buzilganda uni tiklaydi.

### insert (heapify-up) — O(log n)

Yangi elementni array oxiriga qo'shamiz, so'ng uni parent'i bilan solishtirib, kerak bo'lsa yuqoriga "ko'taramiz" (swap), to heap property tiklanguncha.

```js
insert(val) {
  this.heap.push(val);
  this._siftUp(this.heap.length - 1);
}

_siftUp(i) {
  while (i > 0) {
    const parent = Math.floor((i - 1) / 2);
    if (this.heap[parent] <= this.heap[i]) break; // min-heap shart
    [this.heap[parent], this.heap[i]] = [this.heap[i], this.heap[parent]];
    i = parent;
  }
}
// Complexity: O(log n) — daraxt balandligi bo'yicha
```

### extractMin (heapify-down) — O(log n)

Ildizni (eng kichik element) qaytaramiz. Oxirgi elementni ildizga ko'chiramiz, so'ng uni kichikroq farzandi bilan almashtirib pastga "tushiramiz".

```js
extractMin() {
  const min = this.heap[0];
  const last = this.heap.pop();
  if (this.heap.length > 0) {
    this.heap[0] = last;
    this._siftDown(0);
  }
  return min;
}

_siftDown(i) {
  const n = this.heap.length;
  while (true) {
    let smallest = i;
    const l = 2 * i + 1, r = 2 * i + 2;
    if (l < n && this.heap[l] < this.heap[smallest]) smallest = l;
    if (r < n && this.heap[r] < this.heap[smallest]) smallest = r;
    if (smallest === i) break;
    [this.heap[i], this.heap[smallest]] = [this.heap[smallest], this.heap[i]];
    i = smallest;
  }
}
// Complexity: O(log n)
```

### peek — O(1)

```js
peek() {
  return this.heap[0]; // faqat ildizga qaraymiz, o'zgartirmaymiz
}
// Complexity: O(1)
```

---

## Build heap O(n)

**💡 Tushuncha:** Tayyor arraydan heap qurish uchun har bir elementni `insert` qilsangiz O(n log n) bo'ladi. Lekin **bottom-up** usulda (oxirgi parent'dan boshlab `siftDown` qilib) buni O(n) da bajarish mumkin.

```js
buildHeap(arr) {
  this.heap = arr;
  // Oxirgi non-leaf node'dan boshlab teskari yo'nalishda
  for (let i = Math.floor(arr.length / 2) - 1; i >= 0; i--) {
    this._siftDown(i);
  }
}
// Complexity: O(n) — matematik isbot bilan, intuitiv O(n log n) bo'lib ko'rinsa-da
```

**✅ Sabab:** Pastdagi node'lar (ko'pchilik) kam ish bajaradi, yuqoridagilar (oz son) ko'p ish bajaradi. Yig'indi O(n) ga teng bo'lib chiqadi.

---

## Priority Queue

**💡 Tushuncha:** Priority Queue (ustuvor navbat) — bu navbat bo'lib, undan element chiqarganda navbat tartibi emas, balki **ustuvorlik (priority)** bo'yicha eng yuqori (yoki eng past) element chiqadi. Heap aynan shu uchun ideal: `insert` O(log n), `extract` O(log n), `peek` O(1).

Ko'pincha element bilan birga priority'ni juftlik sifatida saqlaymiz: `[priority, value]`. Solishtirishni priority bo'yicha qilamiz.

---

## JS'da MinHeap implementatsiyasi

**⚠️ Ehtiyot bo'l:** JavaScript va TypeScript'da built-in heap yoki priority queue **yo'q** (Java'da `PriorityQueue`, C++'da `priority_queue` bor). Intervyu va real loyihalarda heap'ni qo'lda yozish kerak bo'ladi. Quyida to'liq generic MinHeap class:

```ts
class MinHeap<T> {
  private heap: T[] = [];
  // compare(a, b) < 0 => a "kichikroq" (yuqoriroq priority)
  constructor(private compare: (a: T, b: T) => number = (a, b) =>
    (a as any) - (b as any)) {}

  size(): number { return this.heap.length; }
  isEmpty(): boolean { return this.heap.length === 0; }
  peek(): T | undefined { return this.heap[0]; }

  push(val: T): void {
    this.heap.push(val);
    this._siftUp(this.heap.length - 1);
  }

  pop(): T | undefined {
    if (this.heap.length === 0) return undefined;
    const top = this.heap[0];
    const last = this.heap.pop()!;
    if (this.heap.length > 0) {
      this.heap[0] = last;
      this._siftDown(0);
    }
    return top;
  }

  private _siftUp(i: number): void {
    while (i > 0) {
      const parent = (i - 1) >> 1;
      if (this.compare(this.heap[parent], this.heap[i]) <= 0) break;
      [this.heap[parent], this.heap[i]] = [this.heap[i], this.heap[parent]];
      i = parent;
    }
  }

  private _siftDown(i: number): void {
    const n = this.heap.length;
    while (true) {
      let smallest = i;
      const l = 2 * i + 1, r = 2 * i + 2;
      if (l < n && this.compare(this.heap[l], this.heap[smallest]) < 0) smallest = l;
      if (r < n && this.compare(this.heap[r], this.heap[smallest]) < 0) smallest = r;
      if (smallest === i) break;
      [this.heap[i], this.heap[smallest]] = [this.heap[smallest], this.heap[i]];
      i = smallest;
    }
  }
}

// Max-heap kerak bo'lsa, compare'ni teskari qiling:
const maxHeap = new MinHeap<number>((a, b) => b - a);
```

---

## Use-case'lar

**💡 Tushuncha:** Heap quyidagi muammolarda eng yaxshi yechim hisoblanadi:

- **Top-K:** eng katta/kichik K element (masalan, Kth largest).
- **Merge K sorted:** bir nechta tartiblangan ro'yxatni birlashtirish.
- **Median stream:** oqimda kelayotgan sonlar medianasini real vaqtda topish (ikki heap orqali).
- **Dijkstra algoritmi:** eng qisqa yo'l topishda min-heap priority queue sifatida ishlatiladi.
- **Task scheduling:** eng yuqori prioritetli vazifani tanlash.

---

## Pattern'lar

### Kth largest element

**💡 Tushuncha:** K ta o'lchamli **min-heap** saqlaymiz. Har bir elementni qo'shamiz, agar heap o'lchami K dan oshsa, eng kichigini (ildiz) chiqaramiz. Oxirida ildiz — K-chi eng katta element.

```js
function findKthLargest(nums, k) {
  const minHeap = new MinHeap((a, b) => a - b);
  for (const n of nums) {
    minHeap.push(n);
    if (minHeap.size() > k) minHeap.pop();
  }
  return minHeap.peek();
}
// Time: O(n log k), Space: O(k)
```

### Top K frequent

**💡 Tushuncha:** Avval har bir elementning chastotasini (frequency) hisoblaymiz, so'ng chastota bo'yicha K o'lchovli min-heap ishlatamiz.

```js
function topKFrequent(nums, k) {
  const freq = new Map();
  for (const n of nums) freq.set(n, (freq.get(n) || 0) + 1);

  const minHeap = new MinHeap((a, b) => a[1] - b[1]); // [value, count]
  for (const [val, count] of freq) {
    minHeap.push([val, count]);
    if (minHeap.size() > k) minHeap.pop();
  }
  return minHeap.heap.map(([val]) => val);
}
// Time: O(n log k), Space: O(n)
```

### Merge K sorted lists

**💡 Tushuncha:** Har bir ro'yxatning birinchi elementini heap'ga qo'shamiz. Eng kichigini chiqaramiz, uning keyingi elementini qo'shamiz.

```js
function mergeKLists(lists) {
  const minHeap = new MinHeap((a, b) => a.val - b.val);
  for (const node of lists) if (node) minHeap.push(node);

  const dummy = { next: null };
  let tail = dummy;
  while (!minHeap.isEmpty()) {
    const node = minHeap.pop();
    tail.next = node;
    tail = node;
    if (node.next) minHeap.push(node.next);
  }
  return dummy.next;
}
// Time: O(N log k), N = jami node'lar soni, Space: O(k)
```

---

## Savol-javoblar

### ❓ Heap bilan tartiblangan array (sorted array) o'rtasida nima farq bor?

**✅ Javob:** Tartiblangan arrayda barcha elementlar to'liq tartibda. Heap'da esa faqat parent-child munosabati kafolatlanadi; sibling'lar tartiblanmagan. Heap insert/extract'ni O(log n) da bajaradi, sorted array esa insert uchun O(n) (siljitish kerak). Agar sizga faqat min/max kerak bo'lsa — heap; agar barcha elementlarga tartibli access kerak bo'lsa — sorted array.

### ❓ Nega heap complete binary tree bo'lishi shart?

**✅ Javob:** Complete bo'lgani uchun daraxt balanslangan bo'lib qoladi (balandlik doim O(log n)), bu esa operatsiyalar O(log n) da bajarilishini kafolatlaydi. Bundan tashqari, complete struktura array'da bo'shliqsiz (gap'siz) saqlash imkonini beradi — pointer kerak emas.

### ❓ Build heap qanday qilib O(n)? Axir har bir siftDown O(log n)-ku?

**✅ Javob:** To'g'ri, eng yomon holatda bitta siftDown O(log n). Lekin ko'pchilik node'lar (yaproqlarga yaqin) juda qisqa masofa tushadi. Matematik yig'indi `Σ (node soni darajada × shu darajadagi balandlik)` O(n) ga yaqinlashadi. Shuning uchun bottom-up build O(n).

### ❓ Min-heap'dan max-heap qanday yasaladi?

**✅ Javob:** Compare funksiyasini teskari qilish kifoya (`(a, b) => b - a`), yoki barcha qiymatlarni manfiy belgi bilan saqlash (`-x`). Eng toza yo'l — generic comparator ishlatish.

### ❓ Kth largest uchun nega min-heap, max-heap emas?

**✅ Javob:** K o'lchovli min-heap saqlaymiz — undagi ildiz har doim shu K element ichidagi eng kichigi. Hammasini qayta ishlagandan keyin, heap'da eng katta K ta element qoladi va ildiz — aynan K-chi eng katta. Max-heap ishlatib hammasini qo'shsangiz O(n log n) bo'ladi; min-heap (o'lcham K) bilan O(n log k) — samaraliroq.

### ❓ heapify-up va heapify-down qachon ishlatiladi?

**✅ Javob:** heapify-up (siftUp) — `insert`da: yangi element pastdan qo'shiladi va yuqoriga ko'tariladi. heapify-down (siftDown) — `extract`da: oxirgi element ildizga qo'yiladi va pastga tushiriladi. Build heap'da esa faqat siftDown ishlatiladi.

### ❓ Heap'da ixtiyoriy elementni o'chirish (delete) qanday?

**✅ Javob:** Avval elementning indeksini topish kerak (O(n) yoki hash-map orqali O(1)). So'ng uni oxirgi element bilan almashtirib, oxiridan o'chiramiz, keyin shu pozitsiyada siftUp yoki siftDown qilamiz (qaysi tomon kerakligini compare bilan aniqlaymiz). Umumiy O(log n), lekin indeks topish O(n) bo'lishi mumkin.

### ❓ Median stream muammosini heap bilan qanday yechamiz?

**✅ Javob:** Ikki heap ishlatamiz: chap yarmi uchun **max-heap** (kichik sonlar), o'ng yarmi uchun **min-heap** (katta sonlar). Ularning o'lchamlarini balansda saqlaymiz (farq ≤ 1). Median — yo max-heap ildizi (toq son holatda), yo ikki ildiz o'rtachasi (juft holatda). Insert O(log n), getMedian O(1).

### ❓ Priority Queue'da bir xil priority'li elementlar tartibi kafolatlanadimi?

**✅ Javob:** Yo'q. Oddiy heap **stable emas** — bir xil priority'li elementlarning kelish tartibi saqlanmaydi. Agar FIFO tartib kerak bo'lsa, comparator'ga ikkinchi kalit sifatida insert tartib raqamini (counter) qo'shing.

### ❓ Heap qanday array'siz, pointer bilan ham qurilishi mumkinmi?

**✅ Javob:** Ha, har bir node'da `left`, `right`, `parent` pointer'lar bilan ham qurish mumkin, lekin bu xotira sarfini oshiradi va kodni murakkablashtiradi. Complete tree xususiyati tufayli array eng samarali — shuning uchun amalda deyarli har doim array ishlatiladi.

### ❓ Dijkstra'da heap nima uchun kerak?

**✅ Javob:** Dijkstra har qadamda "eng kichik masofali" vertex'ni tanlashi kerak. Buni har safar butun array bo'ylab izlash O(V) bo'ladi (umumiy O(V²)). Min-heap bilan eng kichikni O(log V) da olamiz, natijada umumiy O((V + E) log V) — siyrak (sparse) graflarda ancha tezroq.

### ❓ JS'da heap yo'qligi sababli intervyuga qanday tayyorlanish kerak?

**✅ Javob:** MinHeap class'ni yoddan yoza olishingiz kerak: `push` (siftUp), `pop` (siftDown), `peek`. Comparator parametri bilan generic qilib yozsangiz, max-heap va custom-priority holatlarini ham bitta class bilan qoplaysiz. Bu intervyu uchun "shablon" kod.

### ❓ Heap sort qanday ishlaydi va complexity'si qanday?

**✅ Javob:** Avval arraydan max-heap quramiz (O(n)), so'ng ildizni (eng katta) oxiriga almashtirib, heap o'lchamini kamaytirib siftDown qilamiz. Buni n marta takrorlaymiz. Umumiy O(n log n), in-place (O(1) qo'shimcha xotira), lekin stable emas.

### ❓ Top K frequent'ni heap'siz O(n) da yechish mumkinmi?

**✅ Javob:** Ha — **bucket sort** orqali. Chastota indeks bo'lgan bucket'lar yasaymiz (`bucket[freq]`), so'ng eng katta chastotadan boshlab K ta element to'playmiz. Bu O(n), heap-yondashuv esa O(n log k). Lekin heap-yondashuv kodi soddaroq va streaming holatlarda yaxshiroq.

---

## Masalalar

> Yechimlar: [07-heaps yechimlari](../solutions/dsa/07-heaps.md)

1. **Kth Largest Element in an Array** — Tartiblanmagan arrayda K-chi eng katta elementni toping (sort qilmasdan).
2. **Kth Smallest Element in a Sorted Matrix** — Har bir qatori va ustuni tartiblangan matritsada K-chi eng kichik elementni toping.
3. **Top K Frequent Elements** — Arrayda eng ko'p uchraydigan K ta elementni qaytaring.
4. **Top K Frequent Words** — String arrayda eng ko'p uchraydigan K ta so'zni chastota va alifbo tartibida qaytaring.
5. **Merge K Sorted Lists** — K ta tartiblangan linked list'ni bitta tartiblangan list'ga birlashtiring.
6. **Find Median from Data Stream** — Oqimda kelayotgan sonlar uchun istalgan vaqtda medianani qaytaradigan struktura yozing.
7. **K Closest Points to Origin** — Nuqtalar ichidan origin'ga (0,0) eng yaqin K tasini toping.
8. **Last Stone Weight** — Har qadamda eng og'ir ikki toshni urishtirib, qolgan vaznni toping (max-heap).
9. **Task Scheduler** — Bir xil vazifalar orasida `n` interval bo'lishi shart bo'lganda, minimal jami vaqtni hisoblang.
10. **Reorganize String** — String harflarini shunday qayta joylashtiringki, qo'shni harflar bir xil bo'lmasin.
11. **Sort Characters By Frequency** — String'dagi belgilarni chastotasi kamayish tartibida qayta joylashtiring.
12. **Sliding Window Median** — `k` o'lchovli oynani siljitib, har bir pozitsiya uchun medianani hisoblang.

---

← [DSA bo'limiga qaytish](./README.md)
