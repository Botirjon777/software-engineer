# Stacks va Queues

Stack va queue — eng sodda, lekin eng ko'p ishlatiladigan abstrakt ma'lumotlar strukturalari. Stack **LIFO** (Last In, First Out), queue esa **FIFO** (First In, First Out) tartibda ishlaydi. Ular function call stack, undo/redo, BFS, ifoda parsing kabi joylarda har kuni qo'llaniladi. Bu fayl ularning JS implementatsiyasi, complexity nozikliklari va asosiy intervyu pattern'larini (valid parentheses, monotonic stack, min stack, expression evaluation, queue↔stack konversiyalari) qamrab oladi.

## Mundarija

- [Stack (LIFO)](#stack-lifo)
- [Queue (FIFO)](#queue-fifo)
- [Nega array.shift() sekin](#nega-arrayshift-sekin)
- [Deque (ikki tomonlama navbat)](#deque)
- [Priority Queue eslatma](#priority-queue-eslatma)
- [Use-case'lar](#use-caselar)
- [Pattern: Valid Parentheses](#pattern-valid-parentheses)
- [Pattern: Monotonic Stack](#pattern-monotonic-stack)
- [Pattern: Min Stack](#pattern-min-stack)
- [Pattern: Evaluate Expression](#pattern-evaluate-expression)
- [Pattern: Queue ↔ Stack](#pattern-queue--stack)
- [Savol-javoblar](#savol-javoblar)
- [Masalalar](#masalalar)

---

## Stack (LIFO)

Stack — oxirgi qo'shilgan element birinchi chiqadigan struktura. Faqat **bir uchidan** (top) ishlanadi: `push` (qo'shish), `pop` (olib chiqish), `peek` (ko'rish). JS'da array bevosita stack vazifasini bajaradi — `push`/`pop` oxiridan ishlaydi va O(1).

```js
class Stack {
  constructor() {
    this.items = [];
  }
  push(val) { this.items.push(val); }      // O(1)
  pop() { return this.items.pop(); }        // O(1) — oxirgi element
  peek() { return this.items[this.items.length - 1]; } // O(1)
  isEmpty() { return this.items.length === 0; }
  get size() { return this.items.length; }
}

const s = new Stack();
s.push(1); s.push(2); s.push(3);
s.pop();   // => 3 (oxirgi kirgan birinchi chiqadi)
s.peek();  // => 2
```

**💡 Tushuncha:** Stack'ni idishga qalangan tarelkalar deb tasavvur qiling — oxirgi qo'ygan tarelkani birinchi olasiz. `push`/`pop`/`peek` barchasi O(1), chunki faqat bitta uch (top) bilan ishlaymiz.

---

## Queue (FIFO)

Queue — birinchi qo'shilgan element birinchi chiqadigan struktura. Navbatdagi odamlar kabi: kim oldin kelsa, o'sha oldin xizmat oladi. `enqueue` (oxiriga qo'shish), `dequeue` (boshidan olish), `peek` (birinchini ko'rish).

Naif JS yechimi `push` + `shift` bo'ladi, lekin `shift` O(n). Samarali yechim — **index pointer** bilan:

```js
class Queue {
  constructor() {
    this.items = {};
    this.head = 0; // dequeue indeksi
    this.tail = 0; // enqueue indeksi
  }
  enqueue(val) { this.items[this.tail++] = val; } // O(1)
  dequeue() {                                      // O(1)
    if (this.isEmpty()) return undefined;
    const val = this.items[this.head];
    delete this.items[this.head++];
    return val;
  }
  peek() { return this.items[this.head]; } // O(1)
  isEmpty() { return this.head === this.tail; }
  get size() { return this.tail - this.head; }
}
```

**💡 Tushuncha:** Index pointer texnikasi `head` va `tail` indekslarini siljitadi — elementlarni jismonan siljitmaydi. Shu sababli `enqueue` va `dequeue` ikkalasi ham O(1).

---

## Nega array.shift() sekin

```js
const q = [];
q.push(1); q.push(2); q.push(3); // [1, 2, 3]
q.shift(); // => 1, lekin qolgan barcha element bittaga chapga siljiydi: O(n)
```

**⚠️ Ehtiyot bo'l:** `arr.shift()` array'ning birinchi elementini olib tashlaydi, **qolgan barcha elementni bitta chapga siljitadi** — bu O(n). Kichik massivlarda sezilmaydi, lekin millionlab `dequeue`da bu O(n²) ga aylanadi. To'g'ri yechim: index pointer (yuqoridagi `Queue`) yoki ikki stack bilan queue.

**Ikki stack bilan queue:**

```js
class QueueViaStacks {
  constructor() {
    this.inStack = [];
    this.outStack = [];
  }
  enqueue(val) { this.inStack.push(val); } // O(1)
  dequeue() {                              // amortized O(1)
    if (this.outStack.length === 0) {
      while (this.inStack.length) this.outStack.push(this.inStack.pop());
    }
    return this.outStack.pop();
  }
}
```

**💡 Tushuncha:** `inStack`ga qo'shamiz, `outStack`dan olamiz. `outStack` bo'shaganda `inStack`ni unga "ag'daramiz" — tartib teskarilanib, FIFO hosil bo'ladi. Har element ko'pi bilan bir marta ko'chiriladi, shuning uchun amortized O(1).

---

## Deque

Deque (double-ended queue) — ikkala uchidan ham qo'shish/olish mumkin bo'lgan struktura. Stack va queue'ning umumlashmasi.

```js
// JS array deque sifatida (kichik hajmda yetarli):
const dq = [];
dq.push(1);     // oxiriga qo'shish
dq.unshift(0);  // boshiga qo'shish — O(n)!
dq.pop();       // oxiridan olish
dq.shift();     // boshidan olish — O(n)!
```

**⚠️ Ehtiyot bo'l:** Array bilan deque'da `unshift`/`shift` O(n). Haqiqiy O(1) deque kerak bo'lsa, doubly linked list yoki ikki tomonlama o'sadigan struktura ishlatiladi. Sliding window maximum kabi masalalarda deque (index'larni saqlash) juda foydali.

---

## Priority Queue eslatma

Priority queue — elementlar FIFO emas, **prioritet** bo'yicha chiqadi (eng kichik yoki eng katta avval). Samarali implementatsiya **heap** (binary heap) bilan bo'ladi: insert va extract O(log n).

**💡 Tushuncha:** Array bilan har safar minimumni qidirish O(n) bo'ladi. Heap bu operatsiyalarni O(log n)ga tushiradi. Batafsil: [Heaps va Priority Queue](./07-heaps.md). Dijkstra, top-K, task scheduler kabi masalalarda heap asosidagi priority queue ishlatiladi.

---

## Use-case'lar

| Struktura | Tipik use-case |
|-----------|----------------|
| Stack | Function call stack, undo/redo, ifoda parsing, DFS, brauzer "back" tugmasi, balanced parentheses |
| Queue | BFS, task/job scheduling, printer navbati, message queue, rate limiting |
| Deque | Sliding window maximum, palindrom tekshirish, LRU yordamida |
| Priority Queue | Dijkstra, A*, top-K elements, event simulation |

**💡 Tushuncha:** Tilning o'zi stack ishlatadi — har funksiya chaqirig'i **call stack**ga qo'yiladi, return bo'lganda olinadi. Rekursiya juda chuqur bo'lsa, "stack overflow" aynan shu stack to'lib ketgani.

---

## Pattern: Valid Parentheses

Qavslar to'g'ri yopilganini tekshirish — klassik stack masalasi.

```js
function isValid(s) {
  const stack = [];
  const pairs = { ')': '(', ']': '[', '}': '{' };
  for (const ch of s) {
    if (ch === '(' || ch === '[' || ch === '{') {
      stack.push(ch); // ochuvchi — stack'ga
    } else {
      if (stack.pop() !== pairs[ch]) return false; // mos yopuvchi emas
    }
  }
  return stack.length === 0; // hamma ochilgan qavs yopilganmi?
}
```

Complexity: **O(n)** vaqt, **O(n)** xotira.

**💡 Tushuncha:** Stack eng oxirgi ochilgan qavsni tepada saqlaydi. Yopuvchi qavs kelsa, u eng oxirgi ochilganga mos kelishi shart — bu LIFO tabiati bilan ideal mos tushadi.

---

## Pattern: Monotonic Stack

Monotonic stack — elementlari doim o'sib (yoki kamayib) boradigan tartibda saqlanadigan stack. "Next greater element", "daily temperatures" kabi masalalarda ishlatiladi.

```js
// Next Greater Element: har element uchun o'ngdagi birinchi kattaroq elementni top
function nextGreaterElements(nums) {
  const res = new Array(nums.length).fill(-1);
  const stack = []; // indekslarni saqlaydi (qiymatlari kamayuvchi tartibda)
  for (let i = 0; i < nums.length; i++) {
    while (stack.length && nums[i] > nums[stack[stack.length - 1]]) {
      const idx = stack.pop();
      res[idx] = nums[i]; // joriy element — stack tepasi uchun "next greater"
    }
    stack.push(i);
  }
  return res;
}
```

Complexity: **O(n)** vaqt (har element bir marta push/pop), **O(n)** xotira.

**💡 Tushuncha:** Naif yechim har element uchun o'ngga qarab qidiradi — O(n²). Monotonic stack "kutayotgan" elementlarni saqlaydi va kattaroq element kelganda ularni bir marta hal qiladi — shuning uchun O(n).

---

## Pattern: Min Stack

`getMin`ni O(1)da qaytaradigan stack. Har push'da o'sha paytdagi minimumni ham saqlaymiz.

```js
class MinStack {
  constructor() {
    this.stack = [];
    this.minStack = []; // har bosqichdagi minimum
  }
  push(val) {
    this.stack.push(val);
    const min = this.minStack.length
      ? Math.min(val, this.minStack[this.minStack.length - 1])
      : val;
    this.minStack.push(min);
  }
  pop() {
    this.minStack.pop();
    return this.stack.pop();
  }
  top() { return this.stack[this.stack.length - 1]; }
  getMin() { return this.minStack[this.minStack.length - 1]; } // O(1)
}
```

Barcha operatsiyalar **O(1)**, qo'shimcha **O(n)** xotira.

**💡 Tushuncha:** `minStack` har bir holatdagi minimumni "tarix" sifatida saqlaydi. `pop` qilganda ikkala stack ham sinxron qisqaradi, shuning uchun joriy minimum doim to'g'ri.

---

## Pattern: Evaluate Expression

**Reverse Polish Notation (RPN)** ifodasini hisoblash — stack uchun klassik.

```js
function evalRPN(tokens) {
  const stack = [];
  const ops = {
    '+': (a, b) => a + b,
    '-': (a, b) => a - b,
    '*': (a, b) => a * b,
    '/': (a, b) => Math.trunc(a / b),
  };
  for (const t of tokens) {
    if (t in ops) {
      const b = stack.pop();
      const a = stack.pop();
      stack.push(ops[t](a, b)); // tartib muhim: a OP b
    } else {
      stack.push(Number(t));
    }
  }
  return stack.pop();
}
// evalRPN(["2","1","+","3","*"]) => 9   ((2+1)*3)
```

Complexity: **O(n)** vaqt, **O(n)** xotira.

**⚠️ Ehtiyot bo'l:** Operand tartibi muhim — `b` avval pop bo'ladi (chunki u keyin push qilingan), shuning uchun `a OP b`. `-` va `/` da tartibni almashtirsangiz natija noto'g'ri bo'ladi.

---

## Pattern: Queue ↔ Stack

**Stack'dan Queue (yuqorida ko'rdik):** ikki stack, `in` va `out`. Amortized O(1).

**Queue'dan Stack:** bitta queue bilan. `push`da yangi elementni qo'shib, undan oldingilarni aylantirib orqasiga olib o'tamiz.

```js
class StackViaQueue {
  constructor() {
    this.queue = []; // index-pointer queue bo'lsa yanada yaxshi
  }
  push(val) {
    this.queue.push(val);
    // yangi elementni "old"ga olib chiqish uchun qolganini aylantirish
    for (let i = 0; i < this.queue.length - 1; i++) {
      this.queue.push(this.queue.shift());
    }
  }
  pop() { return this.queue.shift(); } // O(1) (push O(n))
  top() { return this.queue[0]; }
}
```

**💡 Tushuncha:** Trade-off bor: bu yondashuvda `push` O(n), `pop` O(1). Aksincha qilib `pop`ni O(n), `push`ni O(1) ham qilish mumkin. Qaysi operatsiya tez-tez chaqirilishiga qarab tanlang.

---

## Savol-javoblar

### ❓ Stack va queue orasidagi farq nima?

**✅ Javob:** Stack — LIFO (Last In First Out), faqat bir uchidan (top) ishlanadi. Queue — FIFO (First In First Out), bir uchidan qo'shilib (rear), boshqasidan olinadi (front). Stack DFS va undo uchun, queue BFS va scheduling uchun mos.

### ❓ JS'da stack'ni qanday implement qilasiz?

**✅ Javob:** Oddiy array yetarli: `push` (oxiriga qo'shish) va `pop` (oxiridan olish) ikkalasi ham O(1). `peek` uchun `arr[arr.length-1]`. Array oxiridan ishlash siljitishni talab qilmaydi, shuning uchun samarali.

### ❓ Nega array.shift() bilan queue yomon g'oya?

**✅ Javob:** `shift()` birinchi elementni olib, qolgan barcha elementni bitta chapga siljitadi — O(n). Ko'p dequeue qilinsa, umumiy O(n²) bo'ladi. Yaxshiroq: index pointer (head/tail indekslarini siljitish) yoki ikki stack — har biri O(1) yoki amortized O(1).

### ❓ Ikki stack bilan queue qanday quriladi?

**✅ Javob:** `inStack`ka enqueue qilamiz. Dequeue'da `outStack` bo'sh bo'lsa, `inStack`ni `outStack`ka ag'daramiz (tartib teskarilanadi → FIFO), so'ng `outStack`dan pop qilamiz. Har element ko'pi bilan bir marta ko'chadi → amortized O(1).

### ❓ Deque nima va qachon kerak?

**✅ Javob:** Deque — ikkala uchidan ham qo'shish/olish mumkin bo'lgan struktura. Sliding window maximum (oynadagi maksimumni O(n)da topish), palindrom tekshirish kabi masalalarda ishlatiladi. JS array'da `unshift`/`shift` O(n) bo'lgani uchun haqiqiy O(1) deque doubly linked list bilan quriladi.

### ❓ Priority queue oddiy queue'dan nimasi bilan farq qiladi?

**✅ Javob:** Oddiy queue FIFO — kelish tartibida chiqaradi. Priority queue prioritet bo'yicha chiqaradi (eng kichik/katta avval). Samarali implementatsiyasi heap bilan — insert va extract O(log n). Dijkstra, top-K, scheduling'da ishlatiladi.

### ❓ Valid parentheses'ni nega stack bilan yechamiz?

**✅ Javob:** Eng oxirgi ochilgan qavs eng birinchi yopilishi kerak — bu aniq LIFO. Ochuvchi qavslarni stack'ga push qilamiz, yopuvchi kelganda top bilan mos kelishini tekshiramiz. Oxirida stack bo'sh bo'lsa — to'g'ri. O(n) vaqt.

### ❓ Monotonic stack nima va qachon ishlatiladi?

**✅ Javob:** Elementlari monotonik (o'suvchi yoki kamayuvchi) tartibda saqlanadigan stack. "Next greater/smaller element", "daily temperatures", "largest rectangle in histogram" kabi masalalarni O(n)da yechadi — har element bir marta push va bir marta pop bo'ladi.

### ❓ Min stack'da getMin'ni qanday O(1) qilasiz?

**✅ Javob:** Asosiy stack bilan birga `minStack` yuritamiz — har push'da o'sha paytdagi minimumni saqlaymiz (`min(yangi, oldingi min)`). `pop`da ikkala stack sinxron qisqaradi. `getMin` faqat `minStack` tepasini qaytaradi — O(1).

### ❓ RPN (postfix) ifodani qanday hisoblaysiz?

**✅ Javob:** Token'larni o'qiymiz: son bo'lsa stack'ga push; operator bo'lsa ikki operandni pop qilib (tartib: avval pop bo'lgani `b`), `a OP b` ni hisoblab natijani push qilamiz. Oxirida stack'da bitta natija qoladi. O(n).

### ❓ Function call stack nima?

**✅ Javob:** Dastur har funksiya chaqirig'ini call stack'ga (frame) qo'yadi: lokal o'zgaruvchilar, return manzili shu yerda saqlanadi. Funksiya return qilganda frame pop bo'ladi. Rekursiya juda chuqur bo'lsa, stack to'lib "stack overflow" yuz beradi — bu nega iterative yechim ba'zan afzalligini tushuntiradi.

### ❓ BFS'da nega queue, DFS'da nega stack ishlatiladi?

**✅ Javob:** BFS qatlam-qatlam (eng yaqindan) yuradi — birinchi ko'rilgan node birinchi qayta ishlanishi kerak → FIFO → queue. DFS chuqurroq ketib, keyin orqaga qaytadi — eng oxirgi node birinchi qayta ishlanadi → LIFO → stack (yoki rekursiya orqali call stack).

### ❓ Stack va queue'ni linked list bilan implement qilsa bo'ladimi?

**✅ Javob:** Ha. Stack uchun head'dan push/pop — O(1). Queue uchun head'dan dequeue, tail pointer bilan tail'ga enqueue — ikkalasi O(1). Linked list dinamik o'lchamga ega va array'dagi qayta o'lcham (resize) muammosi bo'lmaydi.

### ❓ Array'da push O(1) amortized degani nima?

**✅ Javob:** Array to'lganda ikki barobar kattaroq joyga ko'chiriladi (O(n)), lekin bu kamdan-kam yuz beradi. Ko'p push'lar bo'yicha o'rtacha (amortized) xarajat O(1) bo'lib qoladi. Shuning uchun stack'da `push` "amortized O(1)" deyiladi.

### ❓ Sliding window maximum'ni deque bilan qanday yechasiz?

**✅ Javob:** Monotonic deque'da **indekslarni** saqlaymiz, qiymatlari kamayuvchi tartibda. Yangi element kelganda undan kichik bo'lgan orqadagi indekslarni olib tashlaymiz, oynadan chiqib ketgan indeksni oldidan olib tashlaymiz. Deque oldida doim joriy oyna maksimumi turadi. O(n).

---

## Masalalar

> Yechimlar: [`solutions/dsa/04-stacks-queues.md`](../solutions/dsa/04-stacks-queues.md)

1. **Valid Parentheses** — `()[]{}` qavslari to'g'ri ochilib-yopilganini stack bilan tekshiring. (LeetCode 20)

2. **Min Stack** — `push`, `pop`, `top` va `getMin` operatsiyalarining barchasi O(1) bo'lgan stack quring. (LeetCode 155)

3. **Implement Queue using Stacks** — faqat ikki stack ishlatib queue (`push`, `pop`, `peek`, `empty`) quring. Amortized O(1). (LeetCode 232)

4. **Implement Stack using Queues** — queue(lar) ishlatib stack quring. Push yoki pop'dan birini O(n) qilishingiz mumkin. (LeetCode 225)

5. **Evaluate Reverse Polish Notation** — RPN (postfix) ifodani stack bilan hisoblang. (LeetCode 150)

6. **Next Greater Element I** — har element uchun o'ngdagi birinchi kattaroq elementni monotonic stack bilan toping. (LeetCode 496)

7. **Daily Temperatures** — har kun uchun necha kundan keyin harorat ko'tarilishini monotonic stack bilan hisoblang. (LeetCode 739)

8. **Largest Rectangle in Histogram** — histogrammada eng katta to'rtburchak yuzasini monotonic stack bilan toping. (LeetCode 84)

9. **Sliding Window Maximum** — `k` o'lchamli oynani siljitib har oynadagi maksimumni deque bilan O(n)da toping. (LeetCode 239)

10. **Basic Calculator II** — `+ - * /` va bo'sh joylardan iborat ifodani stack bilan hisoblang (qavssiz). (LeetCode 227)

11. **Decode String** — `3[a2[c]]` kabi kodlangan satrni stack bilan dekodlang. (LeetCode 394)

12. **Remove Duplicate Letters** — har harf bir marta qoladigan, leksikografik eng kichik natijani monotonic stack bilan toping. (LeetCode 316)

---

← [DSA bo'limiga qaytish](./README.md)
