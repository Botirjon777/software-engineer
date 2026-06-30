# Linked Lists

Linked list — elementlar (node'lar) xotirada ketma-ket emas, balki **pointer'lar** orqali bir-biriga bog'langan chiziqli ma'lumotlar strukturasi. Array'dan farqli o'laroq, indeks bo'yicha tasodifiy kirish (random access) yo'q, lekin boshiga/oxiriga element qo'shish/o'chirish ko'p hollarda O(1). Bu fayl singly va doubly linked list, asosiy operatsiyalar va eng ko'p so'raladigan intervyu pattern'larini (reverse, fast & slow pointers, dummy head, merge, remove Nth) qamrab oladi.

## Mundarija

- [Node tuzilishi](#node-tuzilishi)
- [Singly vs Doubly Linked List](#singly-vs-doubly-linked-list)
- [Array vs Linked List (taqqoslash)](#array-vs-linked-list)
- [Asosiy operatsiyalar va complexity](#asosiy-operatsiyalar-va-complexity)
- [Pattern: Reverse a Linked List](#pattern-reverse-a-linked-list)
- [Pattern: Fast & Slow Pointers](#pattern-fast--slow-pointers)
- [Pattern: Dummy Head Node](#pattern-dummy-head-node)
- [Pattern: Merge Two Sorted Lists](#pattern-merge-two-sorted-lists)
- [Pattern: Remove Nth Node From End](#pattern-remove-nth-node-from-end)
- [Savol-javoblar](#savol-javoblar)
- [Masalalar](#masalalar)

---

## Node tuzilishi

Linked list'ning eng kichik birligi — **node**. Har bir node ikki qismdan iborat: qiymat (`val`) va keyingi node'ga ko'rsatkich (`next`).

```js
class ListNode {
  constructor(val = 0, next = null) {
    this.val = val;
    this.next = next;
  }
}

// 1 -> 2 -> 3 -> null
const head = new ListNode(1, new ListNode(2, new ListNode(3)));
```

**💡 Tushuncha:** `head` — ro'yxatning boshiga ko'rsatkich. Agar `head`ni yo'qotsangiz, butun ro'yxatga kirish imkonini yo'qotasiz (garbage collector uni tozalaydi). Shuning uchun ko'pchilik algoritmda `head`ni alohida o'zgaruvchida saqlab, harakatni boshqa pointer (`curr`) bilan qilamiz.

---

## Singly vs Doubly Linked List

**Singly linked list** — har node faqat `next`ga ega. Faqat oldinga harakatlanish mumkin.

```js
// Singly: 1 -> 2 -> 3 -> null
class SinglyNode {
  constructor(val) {
    this.val = val;
    this.next = null;
  }
}
```

**Doubly linked list** — har node `next` va `prev`ga ega. Ikki tomonga ham harakatlanish mumkin.

```js
// Doubly: null <- 1 <-> 2 <-> 3 -> null
class DoublyNode {
  constructor(val) {
    this.val = val;
    this.next = null;
    this.prev = null;
  }
}
```

**💡 Tushuncha:** Doubly list `prev` saqlagani uchun node'ni o'chirish ham, oldingi node'ga qaytish ham oson — lekin har node qo'shimcha pointer uchun ko'proq xotira ishlatadi. **LRU cache**, brauzer tarixi (orqaga/oldinga), undo/redo kabi joylarda doubly list ishlatiladi.

| Xususiyat | Singly | Doubly |
|-----------|--------|--------|
| Pointer/node | 1 (`next`) | 2 (`next`, `prev`) |
| Yo'nalish | Faqat oldinga | Ikki tomonga |
| Xotira | Kam | Ko'p |
| Node o'chirish (pointer bor) | Oldingisini topish kerak | O(1) |

---

## Array vs Linked List

| Operatsiya | Array | Linked List |
|-----------|-------|-------------|
| Indeks bo'yicha kirish (`arr[i]`) | **O(1)** | O(n) |
| Boshiga insert/delete | O(n) (siljitish) | **O(1)** |
| Oxiriga insert | O(1)* amortized | O(n) (tail'siz) / O(1) (tail bor) |
| O'rtaga insert/delete (pointer bor) | O(n) | **O(1)** |
| Qidirish (qiymat bo'yicha) | O(n) | O(n) |
| Xotira | Ketma-ket, zich | Tarqoq + pointer overhead |
| Cache locality | Yaxshi | Yomon |

**⚠️ Ehtiyot bo'l:** "Linked list har doim array'dan tez" — bu noto'g'ri. Linked list random access'da sekin va cache locality yomon. Array boshiga emas, **oxiriga** qo'shish ko'p hollarda O(1) amortized. Linked list'ning kuchi — **boshiga/o'rtaga** (pointer mavjud bo'lganda) O(1) insert/delete.

---

## Asosiy operatsiyalar va complexity

```js
class LinkedList {
  constructor() {
    this.head = null;
    this.size = 0;
  }

  // Boshiga qo'shish — O(1)
  prepend(val) {
    this.head = new ListNode(val, this.head);
    this.size++;
  }

  // Oxiriga qo'shish — O(n) (tail pointer bo'lmasa)
  append(val) {
    const node = new ListNode(val);
    if (!this.head) {
      this.head = node;
    } else {
      let curr = this.head;
      while (curr.next) curr = curr.next;
      curr.next = node;
    }
    this.size++;
  }

  // Qiymat bo'yicha qidirish — O(n)
  find(val) {
    let curr = this.head;
    while (curr) {
      if (curr.val === val) return curr;
      curr = curr.next;
    }
    return null;
  }

  // Qiymat bo'yicha o'chirish — O(n)
  delete(val) {
    if (!this.head) return;
    if (this.head.val === val) {
      this.head = this.head.next;
      this.size--;
      return;
    }
    let curr = this.head;
    while (curr.next && curr.next.val !== val) curr = curr.next;
    if (curr.next) {
      curr.next = curr.next.next; // o'rtadagi node'ni "ko'prik" qilib o'tkazib yuborish
      this.size--;
    }
  }
}
```

**💡 Tushuncha:** Insert/delete o'zi O(1), lekin avval kerakli pozitsiyaga yetib borish O(n). Shuning uchun "node'ga pointer bor" deyilganda o'chirish O(1), aks holda umumiy operatsiya O(n).

---

## Pattern: Reverse a Linked List

Eng klassik intervyu savoli. Uch pointer (`prev`, `curr`, `next`) bilan har bir `next`ni teskari yo'naltiramiz.

**Iterative — O(n) vaqt, O(1) xotira:**

```js
function reverseList(head) {
  let prev = null;
  let curr = head;
  while (curr) {
    const next = curr.next; // keyingisini saqlab qol
    curr.next = prev;        // yo'nalishni teskari qil
    prev = curr;             // prev'ni surish
    curr = next;             // curr'ni surish
  }
  return prev; // yangi head
}
```

**Recursive — O(n) vaqt, O(n) xotira (call stack):**

```js
function reverseListRec(head) {
  if (head === null || head.next === null) return head;
  const newHead = reverseListRec(head.next);
  head.next.next = head; // keyingi node bizga qarab ko'rsatsin
  head.next = null;       // o'zimiznikini uzamiz
  return newHead;
}
```

**⚠️ Ehtiyot bo'l:** `next`ni saqlab qolmasdan `curr.next = prev` qilsangiz, ro'yxatning qolgan qismiga kirish imkonini yo'qotasiz. Tartibni unutmang: saqla → teskari qil → suril.

---

## Pattern: Fast & Slow Pointers

Ikki pointer turli tezlikda harakatlanadi (slow 1 qadam, fast 2 qadam). **Floyd's Cycle Detection** va o'rtani topishda ishlatiladi.

**Cycle detection (Floyd's Tortoise and Hare) — O(n) vaqt, O(1) xotira:**

```js
function hasCycle(head) {
  let slow = head;
  let fast = head;
  while (fast && fast.next) {
    slow = slow.next;       // 1 qadam
    fast = fast.next.next;  // 2 qadam
    if (slow === fast) return true; // uchrashdi => sikl bor
  }
  return false;
}
```

**O'rtani topish — O(n) vaqt, O(1) xotira:**

```js
function findMiddle(head) {
  let slow = head;
  let fast = head;
  while (fast && fast.next) {
    slow = slow.next;
    fast = fast.next.next;
  }
  return slow; // fast oxiriga yetganda slow o'rtada bo'ladi
}
```

**💡 Tushuncha:** Agar siklda bo'lsangiz, fast pointer slow'ni "aylanib" quvib yetadi — xuddi stadion yugurishida tez yuguruvchi sekinini lap qilgani kabi. O'rta uchun: fast 2x tez yurgani sababli oxirga yetganda slow aynan yarmiga yetadi.

---

## Pattern: Dummy Head Node

Boshidagi maxsus holatlarni (head o'zgarishi, bo'sh ro'yxat) bartaraf qilish uchun soxta (dummy/sentinel) node ishlatiladi.

```js
function removeElements(head, val) {
  const dummy = new ListNode(0, head); // dummy -> head -> ...
  let curr = dummy;
  while (curr.next) {
    if (curr.next.val === val) {
      curr.next = curr.next.next; // o'tkazib yubor
    } else {
      curr = curr.next;
    }
  }
  return dummy.next; // haqiqiy head
}
```

**💡 Tushuncha:** Dummy node bo'lmasa, head o'chishi mumkin bo'lgan har bir holatni alohida `if` bilan ishlash kerak. Dummy bilan **barcha node'lar bir xil qoidaga** bo'ysunadi — kod toza va xatosiz. `dummy.next`ni qaytaring, `dummy`ni emas.

---

## Pattern: Merge Two Sorted Lists

Ikki saralangan ro'yxatni bitta saralangan ro'yxatga birlashtirish. Dummy head bu yerda juda qulay.

```js
function mergeTwoLists(l1, l2) {
  const dummy = new ListNode(0);
  let tail = dummy;
  while (l1 && l2) {
    if (l1.val <= l2.val) {
      tail.next = l1;
      l1 = l1.next;
    } else {
      tail.next = l2;
      l2 = l2.next;
    }
    tail = tail.next;
  }
  tail.next = l1 || l2; // qolgan qismni ulab qo'y
  return dummy.next;
}
```

Complexity: **O(n + m)** vaqt, **O(1)** xotira (yangi node yaratilmaydi).

---

## Pattern: Remove Nth Node From End

Oxiridan N-chi node'ni o'chirish. Bir martalik (one-pass) yechim — ikki pointer orasida N qadam masofa ochiladi.

```js
function removeNthFromEnd(head, n) {
  const dummy = new ListNode(0, head);
  let fast = dummy;
  let slow = dummy;
  // fast'ni n+1 qadam oldinga surish
  for (let i = 0; i <= n; i++) fast = fast.next;
  // ikkalasini birga surish — fast oxiriga yetganda slow o'chiriladigan node'dan oldin turadi
  while (fast) {
    fast = fast.next;
    slow = slow.next;
  }
  slow.next = slow.next.next; // o'chirish
  return dummy.next;
}
```

Complexity: **O(n)** vaqt (bir o'tish), **O(1)** xotira.

**💡 Tushuncha:** `fast` va `slow` orasidagi masofani aynan `n` qilib saqlasak, `fast` oxiriga yetganda `slow` o'chiriladigan node'ning **oldida** turadi — bu o'chirish uchun aynan kerakli pozitsiya.

---

## Savol-javoblar

### ❓ Linked list nima va array'dan asosiy farqi nimada?

**✅ Javob:** Linked list — node'lardan iborat chiziqli struktura, har node qiymat va keyingisiga pointer saqlaydi. Array elementlari xotirada ketma-ket joylashadi va indeks bo'yicha O(1) kirish beradi; linked list elementlari tarqoq joylashadi va indeks bo'yicha kirish O(n). Buning evaziga linked list boshiga/o'rtaga (pointer mavjud bo'lganda) O(1) insert/delete beradi.

### ❓ Singly va doubly linked list orasidagi farq nima?

**✅ Javob:** Singly node faqat `next`ga ega — faqat oldinga harakat. Doubly `next` va `prev`ga ega — ikki tomonga harakat va pointer mavjud node'ni O(1)da o'chirish. Doubly ko'proq xotira ishlatadi, lekin LRU cache, undo/redo, brauzer tarixi kabi holatlarda qulay.

### ❓ Linked list'da indeks bo'yicha kirish nega O(n)?

**✅ Javob:** Array'da `arr[i]` manzilni to'g'ridan-to'g'ri hisoblaydi (bazaviy manzil + i * element_size). Linked list'da node'lar tarqoq joylashgani uchun `i`-elementga yetish uchun `head`dan boshlab `i` marta `next` bo'ylab yurish kerak — bu O(n).

### ❓ Reverse a linked list'ni iterative qanday qilasiz?

**✅ Javob:** Uch pointer: `prev=null`, `curr=head`. Har iteratsiyada `next`ni saqlaymiz, `curr.next = prev` qilib yo'nalishni teskari qilamiz, so'ng `prev` va `curr`ni oldinga suramiz. Oxirida `prev` yangi head bo'ladi. O(n) vaqt, O(1) xotira.

### ❓ Iterative va recursive reverse'dan qaysi biri yaxshiroq?

**✅ Javob:** Iterative — O(1) xotira, shuning uchun katta ro'yxatlar uchun afzal. Recursive — O(n) xotira (call stack), uzun ro'yxatda stack overflow xavfi bor. Intervyuda ikkalasini ham bilish va trade-off'ni tushuntirish kutiladi.

### ❓ Floyd's cycle detection qanday ishlaydi?

**✅ Javob:** Slow pointer 1 qadam, fast pointer 2 qadam yuradi. Agar siklda bo'lsa, fast slow'ni aylanib quvib yetadi va ular bir xil node'da uchrashadi. Sikl bo'lmasa, fast `null`ga yetadi. O(n) vaqt, O(1) xotira — `Set` ishlatishdan (O(n) xotira) afzal.

### ❓ Linked list'ning o'rtasini qanday topasiz?

**✅ Javob:** Fast & slow pointers. Slow 1 qadam, fast 2 qadam. Fast oxiriga (`null` yoki oxirgi node) yetganda slow aynan o'rtada bo'ladi. Bir o'tishda (one-pass), uzunlikni oldindan hisoblamasdan. Juft uzunlikda `while (fast && fast.next)` ikkinchi o'rtani beradi.

### ❓ Dummy head node nima uchun kerak?

**✅ Javob:** Head o'zgarishi mumkin bo'lgan operatsiyalarda (o'chirish, qo'shish) maxsus holatlarni (edge case) bartaraf qilish uchun. Dummy node head'dan oldin turadi, shuning uchun barcha node'lar bir xil mantiq bilan ishlanadi — head uchun alohida `if` shart yozish shart emas. Oxirida `dummy.next` qaytariladi.

### ❓ Tail pointer linked list'ga nima beradi?

**✅ Javob:** Tail pointer oxirgi node'ga ko'rsatadi, shuning uchun oxiriga qo'shish (append) O(n) o'rniga O(1) bo'ladi. Queue'ni linked list bilan implement qilishda muhim — enqueue (tail'ga qo'shish) va dequeue (head'dan olish) ikkalasi ham O(1).

### ❓ Ikki saralangan ro'yxatni qanday merge qilasiz?

**✅ Javob:** Dummy head va `tail` pointer ishlatib, ikkala ro'yxatning hozirgi node'larini taqqoslab, kichigini `tail`ga ulaymiz va o'sha ro'yxatda oldinga suramiz. Biri tugaganda qolganini `tail.next`ga ulaymiz. O(n+m) vaqt, O(1) xotira.

### ❓ Oxiridan N-chi node'ni bir o'tishda qanday o'chirasiz?

**✅ Javob:** Ikki pointer orasiga `n` qadam masofa ochamiz: `fast`ni `n+1` qadam oldinga suramiz (dummy'dan), keyin ikkalasini birga suramiz. `fast` oxiriga yetganda `slow` o'chiriladigan node'ning oldida turadi — `slow.next = slow.next.next`. O(n) vaqt, O(1) xotira.

### ❓ Linked list'da sikl bormi-yo'qmi, Set ishlatmasdan qanday aniqlaysiz?

**✅ Javob:** Floyd's algoritmi (fast & slow). `Set` yechimi O(n) xotira talab qiladi; Floyd O(1) xotira bilan ishlaydi va shuning uchun afzal. Slow va fast uchrashsa — sikl bor.

### ❓ Linked list'ni qachon array o'rniga tanlaysiz?

**✅ Javob:** Tez-tez boshiga/o'rtaga insert/delete kerak bo'lganda va random access kam bo'lganda. Misol: undo stack, queue, LRU cache, polynomial yoki big-integer'ni node'lar zanjiri sifatida saqlash. Random access ko'p bo'lsa yoki cache locality muhim bo'lsa — array afzal.

### ❓ Doubly linked list'da node'ni o'chirish nega O(1)?

**✅ Javob:** Node'ning o'zida `prev` mavjud, shuning uchun oldingi node'ni qidirib topish shart emas: `node.prev.next = node.next` va `node.next.prev = node.prev`. Singly list'da oldingi node'ni topish uchun head'dan qidirish kerak (O(n)).

### ❓ Linked list'da palindrom tekshirishni qanday optimallashtirasiz?

**✅ Javob:** O'rtani fast & slow bilan topib, ikkinchi yarmini reverse qilib, ikkala yarmni node'ma-node taqqoslaymiz. O(n) vaqt, O(1) xotira. Sodda yechim qiymatlarni array'ga ko'chirib taqqoslash bo'lardi, lekin u O(n) xotira ishlatadi.

---

## Masalalar

> Yechimlar: [`solutions/dsa/03-linked-lists.md`](../solutions/dsa/03-linked-lists.md)

1. **Reverse Linked List** — berilgan singly linked list'ni teskari qiling. Iterative va recursive ikkala yechimni ham yozing. (LeetCode 206)

2. **Linked List Cycle** — linked list'da sikl bor-yo'qligini aniqlang. O(1) xotira ishlating. (LeetCode 141)

3. **Linked List Cycle II** — siklning boshlanish node'ini qaytaring (yoki sikl bo'lmasa `null`). (LeetCode 142)

4. **Middle of the Linked List** — linked list'ning o'rta node'ini qaytaring. Juft uzunlikda ikkinchi o'rtani qaytaring. (LeetCode 876)

5. **Merge Two Sorted Lists** — ikki saralangan linked list'ni bitta saralangan list'ga birlashtiring. (LeetCode 21)

6. **Remove Nth Node From End of List** — oxiridan `n`-chi node'ni o'chiring. Bir o'tishda bajaring. (LeetCode 19)

7. **Remove Linked List Elements** — qiymati `val`ga teng barcha node'larni o'chiring. Dummy head ishlating. (LeetCode 203)

8. **Palindrome Linked List** — linked list palindromligini O(n) vaqt va O(1) xotira bilan tekshiring. (LeetCode 234)

9. **Intersection of Two Linked Lists** — ikki linked list kesishadigan node'ni toping (yoki `null`). O(1) xotira. (LeetCode 160)

10. **Add Two Numbers** — ikki son teskari tartibda linked list sifatida berilgan; ularni qo'shib, natijani xuddi shu formatda qaytaring. (LeetCode 2)

11. **Odd Even Linked List** — toq indeksdagi node'larni oldinga, juft indeksdagilarni keyinga guruhlang (qiymatni emas, node'ni). (LeetCode 328)

12. **Reorder List** — `L0 -> L1 -> ... -> Ln` ro'yxatini `L0 -> Ln -> L1 -> Ln-1 -> ...` ko'rinishiga keltiring. (LeetCode 143)

---

← [DSA bo'limiga qaytish](./README.md)
