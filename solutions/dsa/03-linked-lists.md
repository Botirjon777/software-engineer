# Linked Lists — Yechimlar

Bu fayl [`dsa/03-linked-lists.md`](../../dsa/03-linked-lists.md) dagi "Masalalar" bo'limining to'liq yechimlarini complexity va izohlari bilan beradi. Avval mustaqil urinib ko'ring, keyin solishtiring.

Hamma yechimlarda quyidagi node ishlatiladi:

```js
class ListNode {
  constructor(val = 0, next = null) {
    this.val = val;
    this.next = next;
  }
}
```

---

## 1. Reverse Linked List

```js
// Iterative — O(n) vaqt, O(1) xotira
function reverseList(head) {
  let prev = null;
  let curr = head;
  while (curr) {
    const next = curr.next;
    curr.next = prev;
    prev = curr;
    curr = next;
  }
  return prev;
}

// Recursive — O(n) vaqt, O(n) xotira (call stack)
function reverseListRec(head) {
  if (head === null || head.next === null) return head;
  const newHead = reverseListRec(head.next);
  head.next.next = head;
  head.next = null;
  return newHead;
}
```

**Izoh:** Iterative versiyada uch pointer (`prev`, `curr`, `next`) bilan har bog'lanishni teskari yo'naltiramiz. `next`ni avval saqlamasak, qolgan ro'yxatni yo'qotamiz. Recursive versiya oxirgi node'ga yetib, qaytishda `head.next.next = head` orqali pointer'ni teskari quradi — lekin call stack tufayli O(n) xotira. Intervyuda iterative afzal (O(1) xotira).

---

## 2. Linked List Cycle

```js
// O(n) vaqt, O(1) xotira
function hasCycle(head) {
  let slow = head;
  let fast = head;
  while (fast && fast.next) {
    slow = slow.next;
    fast = fast.next.next;
    if (slow === fast) return true;
  }
  return false;
}
```

**Izoh:** Floyd's Tortoise and Hare. Slow 1, fast 2 qadam yuradi. Siklda bo'lsa fast slow'ni quvib yetadi (`slow === fast`). Sikl bo'lmasa fast `null`ga uriladi. `Set` bilan yechim ham bor (ko'rilgan node'larni saqlash), lekin u O(n) xotira ishlatadi — Floyd O(1) xotira bilan yaxshiroq.

---

## 3. Linked List Cycle II

```js
// O(n) vaqt, O(1) xotira
function detectCycle(head) {
  let slow = head;
  let fast = head;
  while (fast && fast.next) {
    slow = slow.next;
    fast = fast.next.next;
    if (slow === fast) {
      // uchrashuvdan keyin: bittasini head'ga qaytarib, ikkalasini 1 qadamdan suramiz
      let ptr = head;
      while (ptr !== slow) {
        ptr = ptr.next;
        slow = slow.next;
      }
      return ptr; // sikl boshi
    }
  }
  return null;
}
```

**Izoh:** Floyd algoritmining matematik xususiyati: uchrashuv nuqtasidan va head'dan bir vaqtda 1 qadamdan yursak, ular aynan **sikl boshida** uchrashadi. Sababi: head'dan sikl boshigacha masofa = uchrashuv nuqtasidan sikl boshigacha (sikl bo'ylab) masofaga teng. Buni isbotlash intervyuda qo'shimcha ball beradi.

---

## 4. Middle of the Linked List

```js
// O(n) vaqt, O(1) xotira
function middleNode(head) {
  let slow = head;
  let fast = head;
  while (fast && fast.next) {
    slow = slow.next;
    fast = fast.next.next;
  }
  return slow;
}
```

**Izoh:** Fast 2x tez yurgani uchun oxiriga yetganda slow yarmiga yetadi. `while (fast && fast.next)` sharti juft uzunlikda **ikkinchi** o'rtani beradi (masala talabi). Agar birinchi o'rta kerak bo'lsa, shartni `while (fast.next && fast.next.next)` qilib o'zgartiriladi.

---

## 5. Merge Two Sorted Lists

```js
// O(n + m) vaqt, O(1) xotira
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
  tail.next = l1 || l2; // qolgan qism allaqachon saralangan
  return dummy.next;
}
```

**Izoh:** Dummy head boshlang'ich edge case'larni bartaraf qiladi. `tail` pointer natija ro'yxatining oxirini kuzatadi. Kichikroq qiymatli node'ni ulaymiz. Biri tugaganda qolganini bevosita ulaymiz — yangi node yaratmaymiz, shuning uchun O(1) xotira.

---

## 6. Remove Nth Node From End of List

```js
// O(n) vaqt (bir o'tish), O(1) xotira
function removeNthFromEnd(head, n) {
  const dummy = new ListNode(0, head);
  let fast = dummy;
  let slow = dummy;
  for (let i = 0; i <= n; i++) fast = fast.next; // fast'ni n+1 oldinga
  while (fast) {
    fast = fast.next;
    slow = slow.next;
  }
  slow.next = slow.next.next;
  return dummy.next;
}
```

**Izoh:** `fast`ni `n+1` qadam oldinga surib, `slow` bilan orasida `n` masofa hosil qilamiz. `fast` `null`ga yetganda `slow` o'chiriladigan node'ning **oldida** turadi. Dummy node head'ning o'zi o'chirilishi mumkin bo'lgan holatni (`n` = uzunlik) muammosiz hal qiladi.

---

## 7. Remove Linked List Elements

```js
// O(n) vaqt, O(1) xotira
function removeElements(head, val) {
  const dummy = new ListNode(0, head);
  let curr = dummy;
  while (curr.next) {
    if (curr.next.val === val) {
      curr.next = curr.next.next; // o'tkazib yubor
    } else {
      curr = curr.next;
    }
  }
  return dummy.next;
}
```

**Izoh:** Dummy head bo'lmasa, head'ning o'zi `val`ga teng bo'lgan holat (ketma-ket bir nechta bo'lishi mumkin) alohida ishlanishi kerak edi. Dummy bilan barcha node bir xil mantiqqa bo'ysunadi. O'chirishdan keyin `curr`ni surmaymiz — chunki yangi `curr.next` ham `val` bo'lishi mumkin.

---

## 8. Palindrome Linked List

```js
// O(n) vaqt, O(1) xotira
function isPalindrome(head) {
  // 1. O'rtani top
  let slow = head, fast = head;
  while (fast && fast.next) {
    slow = slow.next;
    fast = fast.next.next;
  }
  // 2. Ikkinchi yarmni reverse qil
  let prev = null;
  while (slow) {
    const next = slow.next;
    slow.next = prev;
    prev = slow;
    slow = next;
  }
  // 3. Ikki yarmni taqqosla
  let left = head, right = prev;
  while (right) {
    if (left.val !== right.val) return false;
    left = left.next;
    right = right.next;
  }
  return true;
}
```

**Izoh:** Fast & slow bilan o'rtani topib, ikkinchi yarmni teskari qilamiz, so'ng ikki yarmni node'ma-node taqqoslaymiz. O(1) xotira — qiymatlarni array'ga ko'chirish (O(n) xotira) yechimidan afzal. Talab bo'lsa, ro'yxatni asl holiga qaytarish uchun ikkinchi yarmni qayta reverse qilish mumkin.

---

## 9. Intersection of Two Linked Lists

```js
// O(n + m) vaqt, O(1) xotira
function getIntersectionNode(headA, headB) {
  let a = headA;
  let b = headB;
  while (a !== b) {
    a = a === null ? headB : a.next;
    b = b === null ? headA : b.next;
  }
  return a; // kesishma node yoki null
}
```

**Izoh:** Ikki pointer o'z ro'yxati tugagach ikkinchi ro'yxat boshiga "sakraydi". Shu tariqa ikkalasi ham `lenA + lenB` qadam bosadi va aynan kesishma node'da uchrashadi (yoki ikkalasi `null` bo'lib tugaydi). Uzunlik farqini qo'lda hisoblamasdan ham ishlaydi — nafis yechim.

---

## 10. Add Two Numbers

```js
// O(max(n, m)) vaqt, O(max(n, m)) xotira (natija ro'yxati)
function addTwoNumbers(l1, l2) {
  const dummy = new ListNode(0);
  let tail = dummy;
  let carry = 0;
  while (l1 || l2 || carry) {
    const sum = (l1?.val || 0) + (l2?.val || 0) + carry;
    carry = Math.floor(sum / 10);
    tail.next = new ListNode(sum % 10);
    tail = tail.next;
    l1 = l1?.next || null;
    l2 = l2?.next || null;
  }
  return dummy.next;
}
```

**Izoh:** Raqamlar teskari tartibda saqlangani uchun birlik xonasidan boshlab qo'shamiz — bu maktab "ustun bilan qo'shish"ga o'xshaydi. `carry` (ko'chirish) keyingi xonaga uzatiladi. Sikl sharti `carry`ni ham tekshiradi — masalan `[5] + [5] = [0,1]` da oxirgi ko'chirish uchun.

---

## 11. Odd Even Linked List

```js
// O(n) vaqt, O(1) xotira
function oddEvenList(head) {
  if (!head) return head;
  let odd = head;
  let even = head.next;
  const evenHead = even;
  while (even && even.next) {
    odd.next = even.next;
    odd = odd.next;
    even.next = odd.next;
    even = even.next;
  }
  odd.next = evenHead; // toq zanjir oxiriga juft zanjirni ulash
  return head;
}
```

**Izoh:** Ikki alohida zanjir quramiz — toq pozitsiyalar va juft pozitsiyalar (qiymat emas, **indeks** bo'yicha). `evenHead`ni saqlab qolamiz, oxirida toq zanjir oxiriga ulaymiz. Yangi node yaratmasdan, faqat pointer'larni qayta bog'lab O(1) xotira.

---

## 12. Reorder List

```js
// O(n) vaqt, O(1) xotira
function reorderList(head) {
  if (!head || !head.next) return;
  // 1. O'rtani top
  let slow = head, fast = head;
  while (fast.next && fast.next.next) {
    slow = slow.next;
    fast = fast.next.next;
  }
  // 2. Ikkinchi yarmni reverse qil
  let second = slow.next;
  slow.next = null;
  let prev = null;
  while (second) {
    const next = second.next;
    second.next = prev;
    prev = second;
    second = next;
  }
  // 3. Ikki yarmni galma-gal qo'shib chiq
  let first = head;
  second = prev;
  while (second) {
    const t1 = first.next;
    const t2 = second.next;
    first.next = second;
    second.next = t1;
    first = t1;
    second = t2;
  }
}
```

**Izoh:** Uch bosqich: (1) o'rtani top, (2) ikkinchi yarmni reverse qil, (3) ikki yarmni galma-gal (zip) ulab chiq. Bu uch klassik pattern (middle, reverse, merge)ning kombinatsiyasi — shuning uchun intervyularda sevimli. O(1) xotira, chunki faqat pointer'lar qayta bog'lanadi.

---

← [DSA bo'limiga qaytish](../../dsa/README.md)
